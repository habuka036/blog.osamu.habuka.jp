---
title: "Eucalyptus4 でインスタンスタイプのパラメータを変更する方法"
date: 2014-12-11 02:47:38 +0900
tags: ["Eucalyptus","AdventCalendar"]
---
これは [Eucalyptus Advent Calendar 2014](http://www.adventar.org/calendars/547) の11日目のエントリです。昨日は [@giraffeforestg](https://twitter.com/giraffeforestg) さんの「[Eucalyptus 4 の VMware Supportについて調べてみる](http://giraffeforestg.blog.fc2.com/blog-entry-196.html) 」でした。

体当たりで調べてくださってありがとうございます。そうなんです、Eucalyptus にはサブスクリプションを購入しないと使えない機能がいくつかあります。

<!--more-->

----

### インスタンスタイプの値を変更してみる

今日はまた小ネタですが Eucalyptus 4 でインスタンスタイプの値を変更する方法について紹介します。

Eucalyptus はバージョン2系頃までは管理者用の WebUI がありました(評判は良くなかったですが)。でも Eucalyptus 3 から管理者用の画面が綺麗になり、そして Eucalyptus 4 では管理者用の WebUI が消えたように見えます。(少なくとも見当りません)

じゃー、インスタンスタイプはどこで変更するんだ？という疑問がわきまして、本家のドキュメントを見ると [euca-modify-instance-type](https://www.eucalyptus.com/docs/euca2ools/3.0/euca2ools-guide/euca-modify-instance-type.html) というコマンドが増えていました。ちなみにインスタンスタイプの一覧は [euca-describe-instance-types](https://www.eucalyptus.com/docs/euca2ools/3.0/euca2ools-guide/euca-describe-instance-types.html) コマンドで出ます。

    # euca-describe-instance-types 
    INSTANCETYPE	Name         CPUs  Memory (MiB)  Disk (GiB)
    INSTANCETYPE	m1.small        1           256           5
    INSTANCETYPE	t1.micro        1           256           5
    INSTANCETYPE	m1.medium       1           512          10
    INSTANCETYPE	c1.medium       2           512          10
    INSTANCETYPE	m1.large        2           512          10
    INSTANCETYPE	m1.xlarge       2          1024          10
    INSTANCETYPE	c1.xlarge       2          2048          10
    INSTANCETYPE	m2.xlarge       2          2048          10
    INSTANCETYPE	m3.xlarge       4          2048          15
    INSTANCETYPE	m2.2xlarge      2          4096          30
    INSTANCETYPE	m3.2xlarge      4          4096          30
    INSTANCETYPE	cc1.4xlarge     8          3072          60
    INSTANCETYPE	m2.4xlarge      8          4096          60
    INSTANCETYPE	hi1.4xlarge     8          6144         120
    INSTANCETYPE	cc2.8xlarge    16          6144         120
    INSTANCETYPE	cg1.4xlarge    16         12288         200
    INSTANCETYPE	cr1.8xlarge    16         16384         240
    INSTANCETYPE	hs1.8xlarge    48        119808       24000

ということで、早速 m1.small をメモリ 32MB、ディスク 1GB にしてみます。

    # euca-modify-instance-type -d 1 -m 32 m1.small
    INSTANCETYPE	m1.small	1	32	1

    # euca-describe-instance-types m1.small
    INSTANCETYPE	Name      CPUs  Memory (MiB)  Disk (GiB)
    INSTANCETYPE	m1.small     1            32           1

あら簡単。

----
ということで、Eucalyptus Advent Calendar は絶賛募集中です。
