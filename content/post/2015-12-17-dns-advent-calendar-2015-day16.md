+++
tags = ["DNS","AdventCalendar"]
date = "2015-12-17T04:49:39+09:00"
title = "SkyDNSのプチ性能調査"

+++

DNS 大好き娘の皆さん今晩は。これは DNS Advent Calendar 2015 の 16 日目のエントリです。

16日目のエントリなのに 16 日に公開できなくてごめんなさい…orz

<!--more-->

このエントリでは SkyDNS の簡単な性能調査をしてみようと思います。

性能計測パターンは1台と3台の2構成で計測してみます。(本当はもっと台数を線形に増やしてみたいんですが、ちょっと時間的に辛いので、日和って3台までで…)

以下、簡単な環境情報です。

* AMI: Amazon Linux AMI 2015.09.1 (HVM), SSD Volume Type - ami-383c1956
* SkyDNS の Instance Type: t2.micro
* 負荷をかける側の Instance Type: t2.medium
* SkyDNS: 
* etcd:

1台構成時も3台構成時も1台のインスタンスに etcd と SkyDNS がそれぞれ配置されます。図にすると以下のような構成になります。

![](/images/DNS.adventcalendar.2015.1216-01.png )

賢明な諸兄は「え？この構成なら何台構成になっても同じじゃね？」と気付いたと思いますが、まぁ何事もやってみないとわからないってことで、まずは1台構成から測定してみます。測定環境はこんな感じです。

![](/images/DNS.adventcalendar.2015.1216-02.png )

蛇足かもしれませんが、Amazon Linux 上に SkyDNS 環境を構築する手順が以下になります。

    # yum install git
    # cd /usr/local/src/
    # wget https://storage.googleapis.com/golang/go1.5.2.linux-amd64.tar.gz
    # tar -xzf go1.5.2.linux-amd64.tar.gz 
    # mv go /opt/
    # export GOPATH=/usr/local/go
    # export GOROOT=/opt/go
    # export PATH=$GOROOT/bin:$GOPATH/bin:$PATH
    # mkdir -p $GOPATH
    # go get github.com/coreos/etcd
    # cd $GOPATH/src/github.com/coreos/etcd/etcdctl
    # go install
    # go get github.com/skynetservices/skydns

一方、Amazon Linux 上に dnsperf をインストールする手順は以下です。

    # yum --enablerepo=epel install dnsperf

では早速 SkyDNS を起動させます。

    # etcd &
    # skydns -addr 0.0.0.0:53 &

次に SkyDNS に今回の性能調査のためのデータを以下のように手抜きで投入します。(SkyDNS が起動しているホストで実行)

    # for i in `seq 1 10000`; do echo $i | md5sum | sed -e "s|\([0-9a-f]*\)\s.*$|etcdctl set /skydns/local/skydns/\1 `printf "'{\\"host\\":\\"192.168.%d.%d\\"}'" $(( ($i >> 8) & 0xff )) $(( $i & 0xff ))`|" | sh; done

次に投入したデータを dnsperf で索くためのクエリを以下のように作成します。(dnsperf をインストールしたホストで実行)

    # for i in `seq 1 10000`; do echo $i | md5sum | sed -e "s|\([0-9a-f]*\)\s.*$|\1.local.\tA|"; done > /tmp/queryfile

ちゃんと索けるかどうかを確認します。

    # dig @172.30.2.34 a `head -n 1 /tmp/queryfile`



