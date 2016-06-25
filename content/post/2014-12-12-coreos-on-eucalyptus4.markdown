---
title: "CoreOS on Eucalyptus4"
date: 2014-12-12 02:16:33 +0900
tags: ["Eucalyptus","AdventCalendar"]
---
これは [Eucalyptus Advent Calendar 2014](http://www.adventar.org/calendars/547) の12日目のエントリです。昨日は私のエントリ「[Eucalyptus4 でインスタンスタイプのパラメータを変更する方法](http://blog.osamu.habuka.jp/blog/2014/12/11/eucalyptus-advent-calendar-2014-11th/)」でした。

小ネタというより「マニュアルを紹介した」程度でしたが、今日もそんな感じです。

<!--more-->

今日は CoreOS を Eucalyptus4 で動かしてみるという内容ですが、CoreOS の公式ドキュメントの [Running CoreOS on Eucalyptus 3.4](https://coreos.com/docs/running-coreos/platforms/eucalyptus/) を参考にして進めます。

まぁ、公式ドキュメントに書いてあるように、簡単なことしかしません。

    (OpenStack 用のイメージファイルを取得して解凍)
    # wget http://stable.release.core-os.net/amd64-usr/current/coreos_production_openstack_image.img.bz2
    # bunzip2 coreos_production_openstack_image.img.bz2
    (ファイル形式は qcow2 形式なので、raw 形式に変換する)
    # file coreos_production_openstack_image.img
    coreos_production_openstack_image.img: Qemu Image, Format: Qcow , Version: 2
    # qemu-img convert -O raw coreos_production_openstack_image.img coreos_production_openstack_image.raw
    (公式ドキュメントでは Eucalyptus 3 系だったので 3 つのコマンドを利用しているが、Eucalyptus 4 系からは1コマンドで処理可能)
    # euca-install-image -b coreos -r x86_64 -i coreos_production_openstack_image.raw -n coreos --virtualization-type hvm
    /var/tmp/bundle-BRRcnG/coreos_production_openstack_image.raw.part.00 100% |=========================================================|  10.00 MB   8.00 MB/s Time: 0:00:01
    (中略)
    /var/tmp/bundle-BRRcnG/coreos_production_openstack_image.raw.manifest.xml 100% |====================================================|   6.39 kB   2.81 kB/s Time: 0:00:02
    IMAGE   emi-c3cd53c4

あとは登録されたマシンイメージを使ってインスタンスを起動するだけです。

    $ ssh -i ~/euca4_20141203.pem core@192.168.122.246
    The authenticity of host '192.168.122.246 (192.168.122.246)' can't be established.
    ED25519 key fingerprint is 43:6c:bd:63:0e:35:2b:b8:98:95:4f:8b:f6:22:80:78.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '192.168.122.246' (ED25519) to the list of known hosts.
    CoreOS (stable)
    core@192 ~ $

やったー  :)

メモリの消費量が 114MB 程度だと言うので、起動後のメモリを見てみます。

    core@192 ~ $ free -m
                 total       used       free     shared    buffers     cached
    Mem:           494        113        380          0          3         70
    -/+ buffers/cache:         39        454
    Swap:            0          0          0

おぉ、確かに。では、次にキャッシュらを解放してみます。

    core@192 ~ $ sudo sysctl -w vm.drop_caches=3
    vm.drop_caches = 3
    core@192 ~ $ free -m
                 total       used       free     shared    buffers     cached
    Mem:           494         62        431          0          0         25
    -/+ buffers/cache:         36        457
    Swap:            0          0          0

64MB 以下ですね。小さい。

----
ということで、Eucalyptus Advent Calendar は絶賛募集中です。


