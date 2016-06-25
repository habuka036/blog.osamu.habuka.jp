---
title: "Wakame-VDC の LiveDVD を作る方法 〜CentOS 版〜"
date: 2014-12-01 01:02:42 +0900
tags: ["Wakame-vdc","AdventCalendar"]
---
これは [Wakame-vdc / OpenVNet Advent Calendar 2014](http://www.adventar.org/calendars/577) 1日目の投稿です。

1日目ということで難易度の高くない…というか低めの内容になっています。題して「Wakame-VDC の LiveDVD を作る方法 〜CentOS 版〜」です。

<!--more-->

まず最初に LiveDVD を作るための環境を準備します。[こちらのページ](https://github.com/axsh/iso-no-wakame/wiki/HowtoCreateEnvironmentForLiveDVD) に環境の作り方が書いてあるので参考にしてください。

次に、以下を実行して LiveDVD を作成するための ks ファイルをダウンロードします。

    mkdir -p ./wakame-vdc.livedvd/
    cd ./wakame-vdc.livedvd/
    wget https://raw.githubusercontent.com/axsh/iso-no-wakame/master/wakame-livedvd.ks

次に、3つの rpm とマシンイメージをダウンロードします。

    # 上記の ./wakame-vdc.livedvd/ 配下で作業
    mkdir -p ./rpms/
    cd ./rpms/
    wget http://dlc.openvnet.axsh.jp/packages/rhel/openvswitch/6.5/kmod-openvswitch-2.3.0-1.el6.x86_64.rpm
    wget http://dlc.openvnet.axsh.jp/packages/rhel/openvswitch/6.5/openvswitch-2.3.0-1.x86_64.rpm
    wget http://dlc.wakame.axsh.jp/packages/3rd/rhel/6/master/wakame-vdc-ruby-2.0.0.247.axsh0-1.x86_64.rpm
    cd ..
    wget http://dlc.wakame.axsh.jp.s3.amazonaws.com/demo/vmimage/ubuntu-lucid-kvm-md-32.raw.gz

次に、LiveDVD を作成する際にダウンロードする各種 rpm をキャッシュするディレクトリを作ります。

    mkdir -p /var/cache/live

最後に以下を実行すると LiveDVD の iso ファイルが作成されます。

    # 上記の ./wakame-vdc.livedvd/ 配下で作業
    LANG=C livecd-creator --cache=/var/cache/live --config=wakame-livedvd.ks --fslabel=1boxwakame

あとは作成された iso ファイルを使って実機で起動させるもよし、何かしらのゲスト OS として起動するもよしですが…ひとつ重大な注意点があります。

実はこの投稿を書いている時点ではこれによって作成された LiveDVD では Wakame-VDC がちゃんと動きません。何故かというと…原因がまだ究明中なのですが、[恐らく LiveDVD 上の rabbitmq が正しく動けていない様子](https://github.com/axsh/iso-no-wakame/issues/11)です。

ですので「我こそは！」という方は是非この問題を解いてみてください。

なお、この問題が解決していない LiveDVD は[ここ](https://s3-ap-northeast-1.amazonaws.com/iso-no-wakwame/1boxwakame.iso)からダウンロードできます。[iso ファイルのハッシュ](https://s3-ap-northeast-1.amazonaws.com/iso-no-wakwame/1boxwakame.md5sum)。[LiveDVD の使い方](https://s3-ap-northeast-1.amazonaws.com/iso-no-wakwame/1boxwakame.txt)。

ということで、是非素敵な Wakame ライフをすごしてみてください。



