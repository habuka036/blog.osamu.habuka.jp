---
title: "1ノードで何台起動できるか？"
date: 2014-12-15 23:08:28 +0900
tags: ["Eucalyptus","AdventCalendar"]
---
これは [Eucalyptus Advent Calendar 2014](http://www.adventar.org/calendars/547) の15日目のエントリです。昨日は私のエントリ「[CirrOS を Eucalyptus4 で動かしてみる](http://blog.osamu.habuka.jp/blog/2014/12/15/eucalyptus-advent-calendar-2014-14th/)」でした。まぁ、動かなかったんですけどね。

今日は「1ノードで何台起動できるか？」です。もうタイトルからして失敗の匂いがプンプンしやがります。

<!-- more -->

で、環境ですが、この Advent Calendar でやってきたように、KVM のゲスト OS 上に Eucalyptus 環境を立ててますので、そこで大量のインスタンスを起動してみたいと思います。本当は昨日のエントリで CirrOS が無事起動した続きとしてこのエントリが引き立つはずでしたが、なんか失敗への道筋が出来上がっている気がしてなりません。

Eucalyptus 環境のメモリは約 12GB で、インスタンスに使用できるディスク容量は約 60GB で、CPU コア数はたったの2コアという貧相な環境です。

    [root@euca4 ~]# free -m
                 total       used       free     shared    buffers     cached
    Mem:         11902       3356       8546        295          1       1232
    -/+ buffers/cache:       2123       9779
    Swap:         3967          0       3967
    
    [root@euca4 ~]# df -h
    Filesystem            Size  Used Avail Use% Mounted on
    /dev/mapper/vg_euca4-lv_root
                           50G   14G   33G  30% /
    tmpfs                 6.2G  164K  6.2G   1% /dev/shm
    /dev/vda1             477M   61M  391M  14% /boot
    /dev/mapper/vg_euca4-lv_home
                           64G  2.4G   59G   4% /var/lib/eucalyptus
    
    [root@euca4 ~]# grep ^pro /proc/cpuinfo 
    processor	: 0
    processor	: 1

それでも無理矢理設定して、256 インスタンスまでは起動できるようにしてみました。

    [root@euca4 ~]# euca-describe-availability-zones verbose
    AVAILABILITYZONE	default	192.168.122.115 arn:euca:eucalyptus:default:cluster:default-cc-1/	
    AVAILABILITYZONE	|- vm types	free / max   cpu   ram  disk	
    AVAILABILITYZONE	|- m1.small	0256 / 0256   1     32     1	
    AVAILABILITYZONE	|- t1.micro	0203 / 0203   1     64     1	
    AVAILABILITYZONE	|- m1.medium	0101 / 0101   1    128     1	
    AVAILABILITYZONE	|- c1.medium	0025 / 0025   2    512    10	
    AVAILABILITYZONE	|- m1.large	0025 / 0025   2    512    10	
    AVAILABILITYZONE	|- m1.xlarge	0012 / 0012   2   1024    10	
    AVAILABILITYZONE	|- c1.xlarge	0006 / 0006   2   2048    10	
    AVAILABILITYZONE	|- m2.xlarge	0006 / 0006   2   2048    10	
    AVAILABILITYZONE	|- m3.xlarge	0006 / 0006   4   2048    15	
    AVAILABILITYZONE	|- m2.2xlarge	0003 / 0003   2   4096    30	
    AVAILABILITYZONE	|- m3.2xlarge	0003 / 0003   4   4096    30	
    AVAILABILITYZONE	|- cc1.4xlarge	0004 / 0004   8   3072    60	
    AVAILABILITYZONE	|- m2.4xlarge	0003 / 0003   8   4096    60	
    AVAILABILITYZONE	|- hi1.4xlarge	0002 / 0002   8   6144   120	
    AVAILABILITYZONE	|- cc2.8xlarge	0002 / 0002  16   6144   120	
    AVAILABILITYZONE	|- cg1.4xlarge	0001 / 0001  16   12288   200	
    AVAILABILITYZONE	|- cr1.8xlarge	0000 / 0000  16   16384   240	
    AVAILABILITYZONE	|- hs1.8xlarge	0000 / 0000  48   119808  24000	

Eucalyptus4 の WebUI ではマシンイメージから新規にインスタンスを起動する場合は、1セキュリティグループにつきインスタンスは10個までという制約があります。なんちゃらスタックやちょめちょめスタックとかでも似たような話を聴きますが、Eucalyptus でも同じような不具合があるんでしょうかね？

いや、無い、無いと信じたい。伊達に他のほにゃららスタックらより叩かれまくってきたわけじゃないということを、今日ここで証明したい。

脱線しました。で、1度に10インスタンスまでという制約があるのは新規の起動のときのみで、既存のインスタンスと同じ構成で起動する分には制約がありません。

なので、昨日のエントリで起動した CirrOS インスタンスをそのまま流用します。エコです。

では、インスタンスのプルダウンメニューから「Launch more like this」を選んで、一度に200インスタンスを起動してみようと思います。(「起動」って言っても OS の起動は失敗しているので、ズルい気もしますが気にしない)

どどーん。

![](/images/eucalyptus.adventcalendar.2014.1215-01.png )

インスタンス一覧に画面が遷移するところで Internal Server Error になりました。お約束すぎるwww

![](/images/eucalyptus.adventcalendar.2014.1215-02.png )

ところがどっこい、しばらくしてからアクセスすると何と 94 インスタンスが起動に成功しています。

![](/images/eucalyptus.adventcalendar.2014.1215-03.png )

で、他のインスタンスはどうしたのかと言うと、pending のままでした。

何が起きているのかと Eucayptus 側の OS の syslog を見てみると、

    Dec 14 23:29:56 euca4 dhcpd: No subnet declaration for vnet68 (no IPv4 addresses).
    Dec 14 23:29:56 euca4 dhcpd: ** Ignoring requests on vnet68.  If this is not what
    Dec 14 23:29:56 euca4 dhcpd:    you want, please write a subnet declaration
    Dec 14 23:29:56 euca4 dhcpd:    in your dhcpd.conf file for the network segment
    Dec 14 23:29:56 euca4 dhcpd:    to which interface vnet68 is attached. **
    Dec 14 23:29:56 euca4 dhcpd: 
    Dec 14 23:29:56 euca4 dhcpd: 
    Dec 14 23:29:56 euca4 dhcpd: No subnet declaration for vnet67 (no IPv4 addresses).
    Dec 14 23:29:56 euca4 dhcpd: ** Ignoring requests on vnet67.  If this is not what
    Dec 14 23:29:56 euca4 dhcpd:    you want, please write a subnet declaration
    Dec 14 23:29:56 euca4 dhcpd:    in your dhcpd.conf file for the network segment
    Dec 14 23:29:56 euca4 dhcpd:    to which interface vnet67 is attached. **
    Dec 14 23:29:56 euca4 dhcpd: 
    Dec 14 23:29:56 euca4 dhcpd: 
    Dec 14 23:29:56 euca4 rsyslogd-2177: imuxsock begins to drop messages from pid 23662 due to rate-limiting

ってな感じで dhcpd が「定義が足んないんですけどぉ〜」って怒ってました。実際に /var/run/eucalyptus/net/euca-dhcp.conf を見ると 94 ホスト分しか定義が書かれてませんでした。

    # cat /var/run/eucalyptus/net/euca-dhcp.conf | grep ^host | wc -l
    94

逆を言えば、このファイルへの書き出しが上手く行っていたら、200台起動も夢じゃなかったということになりますでしょうかね。

いやー、Eucalyptus 素晴しい。
