---
title: "Eucalyptus 環境を再起動後にインスタンスが起動しなくなったら"
date: 2014-12-08 01:06:53 +0900
tags: ["Eucalyptus","AdventCalendar"]
---
これは [Eucalyptus Advent Calendar 2014](http://www.adventar.org/calendars/547) の8日目のエントリです。昨日は [@giraffeforestg](https://twitter.com/giraffeforestg) さんの「[Eucalyptus 4 の AUTO SCALING機能を使ってみる](http://giraffeforestg.blog.fc2.com/blog-entry-195.html) 」でした。

いつも [@giraffeforestg](https://twitter.com/giraffeforestg) さんのアドベントカレンダー投稿数に驚いてますが、今年の量は更にヤバイですね。でも、おかげで助けてもらったり励みになったりしてます、ありがとうございます。

さて、今日のエントリは「Eucalyptus 環境を再起動後にインスタンスが起動しなくなったら」です。

<!--more-->

実は、今回の Advent Calendar で使用している faststart で作った私の Eucalyptus 環境は、KVM のゲスト OS 上で動いているのですが、VM を再起動した後にインスタンスを起動しようとすると以下の画面のように「Not enough resources (0 in < 1): vm instances.」というツレないメッセージが出力されます。で、画面を見て頂くとわかりますが、インスタンスリストには「No items were found」って出ているように、リソースなんて全然枯渇してないはずなんです。

本当にリソースないの？って思って CLI 叩いて確認すると、

    # euca-describe-availability-zones verbose
    AVAILABILITYZONE	default	192.168.122.115 arn:euca:eucalyptus:default:cluster:default-cc-1/	
    AVAILABILITYZONE	|- vm types	free / max   cpu   ram  disk	
    AVAILABILITYZONE	|- m1.small	0000 / 0000   1    256     5	
    AVAILABILITYZONE	|- t1.micro	0000 / 0000   1    256     5	
    AVAILABILITYZONE	|- m1.medium	0000 / 0000   1    512    10	
    AVAILABILITYZONE	|- c1.medium	0000 / 0000   2    512    10	
    AVAILABILITYZONE	|- m1.large	0000 / 0000   2    512    10	
    AVAILABILITYZONE	|- m1.xlarge	0000 / 0000   2   1024    10	
    AVAILABILITYZONE	|- c1.xlarge	0000 / 0000   2   2048    10	
    AVAILABILITYZONE	|- m2.xlarge	0000 / 0000   2   2048    10	
    AVAILABILITYZONE	|- m3.xlarge	0000 / 0000   4   2048    15	
    AVAILABILITYZONE	|- m2.2xlarge	0000 / 0000   2   4096    30	
    AVAILABILITYZONE	|- m3.2xlarge	0000 / 0000   4   4096    30	
    AVAILABILITYZONE	|- cc1.4xlarge	0000 / 0000   8   3072    60	
    AVAILABILITYZONE	|- m2.4xlarge	0000 / 0000   8   4096    60	
    AVAILABILITYZONE	|- hi1.4xlarge	0000 / 0000   8   6144   120	
    AVAILABILITYZONE	|- cc2.8xlarge	0000 / 0000  16   6144   120	
    AVAILABILITYZONE	|- cg1.4xlarge	0000 / 0000  16   12288   200	
    AVAILABILITYZONE	|- cr1.8xlarge	0000 / 0000  16   16384   240	
    AVAILABILITYZONE	|- hs1.8xlarge	0000 / 0000  48   119808  24000	

あらやだ、本当にリソースゼロですよwww

NC ちゃん大丈夫？息してる？と思って確認すると

    # euca-describe-nodes 
    NODE	default		192.168.122.115	DISABLED	

って出力されます。そもそも CC から見て NC が無効になっているという罠。

NC のログを見ると、とかく「failed だかんね」の一点張り。もうちょっと理由を述べてくれてもいいのに…

    # tail /var/log/eucalyptus/nc.log 
    2014-12-06 17:32:22 ERROR | failed error=1
    2014-12-06 17:32:23 ERROR | failed error=1
    2014-12-06 17:32:29 ERROR | failed error=1
    2014-12-06 17:32:35 ERROR | failed error=1
    2014-12-06 17:32:42 ERROR | failed error=1
    2014-12-06 17:32:42 ERROR | failed error=1
    2014-12-06 17:32:48 ERROR | failed error=1
    2014-12-06 17:32:51 ERROR | failed error=1
    2014-12-06 17:32:54 ERROR | failed error=1
    2014-12-06 17:33:00 ERROR | failed error=1

CC のログを見ると、

    # tail /var/log/eucalyptus/cc.log 
    2014-12-06 17:33:00 ERROR | error message from ncDescribeResource: NULL parameter was passed when a non NULL parameter was expected
    2014-12-06 17:33:02  INFO | modifying node 192.168.122.115 with state=enabled
    2014-12-06 17:33:02 ERROR | returned an error
    2014-12-06 17:33:02ERROR: doEnableService() returned FAIL
    2014-12-06 17:33:06 ERROR | operation on 192.168.122.115 could not be invoked (check NC host, port, and credentials): NULL parameter was passed when a non NULL parameter was expected
    2014-12-06 17:33:06 ERROR | bad return from ncDescribeResource(192.168.122.115) (1)
    2014-12-06 17:33:06 ERROR | error message from ncDescribeResource: NULL parameter was passed when a non NULL parameter was expected
    2014-12-06 17:33:12 ERROR | operation on 192.168.122.115 could not be invoked (check NC host, port, and credentials): NULL parameter was passed when a non NULL parameter was expected
    2014-12-06 17:33:12 ERROR | bad return from ncDescribeResource(192.168.122.115) (1)
    2014-12-06 17:33:12 ERROR | error message from ncDescribeResource: NULL parameter was passed when a non NULL parameter was expected

CC は CC で「NC がちゃんと応答しないぜコンチキショウ」と言ってやがります。「modifying node 192.168.122.115 with state=enabled」って実行してるっぽいのに、NC は無効なまま。

仕方ないので、とりあえず NC を再起動してみます。

    # /etc/init.d/eucalyptus-nc restart
    Restarting Eucalyptus services: done.

で、もう一度 NC の状況を確認すると…

    # euca-describe-nodes 
    NODE	default		192.168.122.115	ENABLED	

おぉ、今度はちゃんと有効になっていますね。そーすると、リソースも…？

    # euca-describe-availability-zones verbose
    AVAILABILITYZONE	default	192.168.122.115 arn:euca:eucalyptus:default:cluster:default-cc-1/	
    AVAILABILITYZONE	|- vm types	free / max   cpu   ram  disk	
    AVAILABILITYZONE	|- m1.small	0019 / 0019   1    256     5	
    AVAILABILITYZONE	|- t1.micro	0019 / 0019   1    256     5	
    AVAILABILITYZONE	|- m1.medium	0009 / 0009   1    512    10	
    AVAILABILITYZONE	|- c1.medium	0009 / 0009   2    512    10	
    AVAILABILITYZONE	|- m1.large	0009 / 0009   2    512    10	
    AVAILABILITYZONE	|- m1.xlarge	0007 / 0007   2   1024    10	
    AVAILABILITYZONE	|- c1.xlarge	0003 / 0003   2   2048    10	
    AVAILABILITYZONE	|- m2.xlarge	0003 / 0003   2   2048    10	
    AVAILABILITYZONE	|- m3.xlarge	0003 / 0003   4   2048    15	
    AVAILABILITYZONE	|- m2.2xlarge	0001 / 0001   2   4096    30	
    AVAILABILITYZONE	|- m3.2xlarge	0001 / 0001   4   4096    30	
    AVAILABILITYZONE	|- cc1.4xlarge	0001 / 0001   8   3072    60	
    AVAILABILITYZONE	|- m2.4xlarge	0001 / 0001   8   4096    60	
    AVAILABILITYZONE	|- hi1.4xlarge	0000 / 0000   8   6144   120	
    AVAILABILITYZONE	|- cc2.8xlarge	0000 / 0000  16   6144   120	
    AVAILABILITYZONE	|- cg1.4xlarge	0000 / 0000  16   12288   200	
    AVAILABILITYZONE	|- cr1.8xlarge	0000 / 0000  16   16384   240	
    AVAILABILITYZONE	|- hs1.8xlarge	0000 / 0000  48   119808  24000	

やったー、ちゃんとリソースも復活！

----
ということで、8日目のエントリでした。9日目のエントリもお楽しみに。

