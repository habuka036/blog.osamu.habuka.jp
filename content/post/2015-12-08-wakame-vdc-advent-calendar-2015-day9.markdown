---
title: "特命係長 只野 LXC"
date: 2015-12-08 21:24:09 +0900
tags: ["Wakame-vdc","AdventCalendar"]
---

Wakame-vdc ユーザの皆さん、磯野家の皆さん、こんにちは。これはオレオレ WakameVdc / OpenVNet Advent Calendar 2015 の9日目のエントリです。

7日目のエントリで「参考になる情報として、[@giraffeforestg](https://twitter.com/giraffeforestg) さんが書いた [Wakame-vdc / OpenVNet Advent Calendar 2014](http://www.adventar.org/calendars/577) の [12/21 分のエント>リ](http://giraffeforestg.blog.fc2.com/blog-category-29.html) があるので、ますそれを一読します。」とか書いておきながら、全然それに沿わずにやってたので、[@giraffeforestg](https://twitter.com/giraffeforestg) さんがやったように、lxc 単体でちゃんと動くかを今日は確認してみたいと思います。

<!--more-->

ってことで、[@giraffeforestg](https://twitter.com/giraffeforestg) さんの実施したことをトレースします。

    # yum install lxc-templates.x86_64
    # ls /usr/share/lxc/templates/
    lxc-alpine     lxc-busybox  lxc-debian    lxc-gentoo        lxc-oracle  lxc-ubuntu
    lxc-altlinux   lxc-centos   lxc-download  lxc-openmandriva  lxc-plamo   lxc-ubuntu-cloud
    lxc-archlinux  lxc-cirros   lxc-fedora    lxc-opensuse      lxc-sshd
    
    # lxc-create -t centos -n testvm01
    Host CPE ID from /etc/system-release-cpe: cpe:/o:centos:linux:6:GA
    dnsdomainname: 不明なホスト
    Checking cache download in /var/cache/lxc/centos/x86_64/6/rootfs ... 
    Downloading centos minimal ...
    読み込んだプラグイン:fastestmirror, security
    インストール処理の設定をしています
    base                                                                            | 3.7 kB     00:00     
    base/primary_db                                                                 | 4.6 MB     00:00     
    updates                                                                         | 3.4 kB     00:00     
    updates/primary_db                                                              | 2.6 MB     00:00     
    依存性の解決をしています
    --> トランザクションの確認を実行しています。
    ---> Package chkconfig.x86_64 0:1.3.49.3-5.el6 will be インストール
    --> 依存性の処理をしています: rtld(GNU_HASH) のパッケージ: chkconfig-1.3.49.3-5.el6.x86_64
    
    (中略)
    
    完了しました!
    Download complete.
    Copy /var/cache/lxc/centos/x86_64/6/rootfs to /var/lib/lxc/testvm01/rootfs ... 
    Copying rootfs to /var/lib/lxc/testvm01/rootfs ...
    Storing root password in '/var/lib/lxc/testvm01/tmp_root_pass'
    Expiring password for user root.
    passwd: 成功
    
    Container rootfs and config have been created.
    Edit the config file to check/enable networking setup.
    
    The temporary root password is stored in:
    
            '/var/lib/lxc/testvm01/tmp_root_pass'
    
    
    The root password is set up as expired and will require it to be changed
    at first login, which you should do as soon as possible.  If you lose the
    root password or wish to change it without starting the container, you
    can change it from the host by running the following command (which will
    also reset the expired flag):
    
            chroot /var/lib/lxc/testvm01/rootfs passwd


成功しました。

    # lxc-ls 
    testvm01
    # chroot /var/lib/lxc/testvm01/rootfs passwd
    # lxc-start -n testvm01
    lxc-start: conf.c: instantiate_veth: 3105 failed to attach 'veth8QPHXV' to the bridge 'virbr0': No such device
    lxc-start: conf.c: lxc_create_network: 3388 failed to create netdev
    lxc-start: start.c: lxc_spawn: 841 failed to create the network
    lxc-start: start.c: __lxc_start: 1100 failed to spawn 'testvm01'
    lxc-start: lxc_start.c: main: 341 The container failed to start.
    lxc-start: lxc_start.c: main: 345 Additional information can be obtained by setting the --logfile and --logpriority options.

あれ…？ virbr0 って…あ、そうか

    # cat /etc/lxc/default.conf 
    lxc.network.type = veth
    lxc.network.link = virbr0
    lxc.network.flags = up

既定値が virbr0 でした。
で、それを元にして生成された testvm01 の設定も virbr0 になっているので、/var/lib/lxc/testvm01/config の lxc.network.link の値を virbr0 を br0 に変更して、もう一度起動してみます。

    # lxc-start -n testvm01
    lxc-start: cgfs.c: cgfs_init: 2246 cgroupfs failed to detect cgroup metadata
    lxc-start: start.c: lxc_spawn: 869 failed initializing cgroup support
    lxc-start: start.c: __lxc_start: 1100 failed to spawn 'testvm01'
    lxc-start: lxc_start.c: main: 341 The container failed to start.
    lxc-start: lxc_start.c: main: 345 Additional information can be obtained by setting the --logfile and --logpriority options.

あれ？やっぱり駄目です。で、メッセージにはさっきから --logfile オプションのことを教えてくれているので、このオプションを利用してログを取ります。

    # lxc-start -n testvm01 --logfile /tmp/lxc.log
    lxc-start: cgfs.c: cgfs_init: 2246 cgroupfs failed to detect cgroup metadata
    lxc-start: start.c: lxc_spawn: 869 failed initializing cgroup support
    lxc-start: start.c: __lxc_start: 1100 failed to spawn 'testvm01'
    lxc-start: lxc_start.c: main: 341 The container failed to start.
    lxc-start: lxc_start.c: main: 345 Additional information can be obtained by setting the --logfile and --logpriority options.
    # cat /tmp/lxc.log 
          lxc-start 1449578879.006 ERROR    lxc_cgfs - cgfs.c:cgfs_init:2246 - cgroupfs failed to detect cgroup metadata
          lxc-start 1449578879.006 ERROR    lxc_start - start.c:lxc_spawn:869 - failed initializing cgroup support
          lxc-start 1449578879.056 ERROR    lxc_start - start.c:__lxc_start:1100 - failed to spawn 'testvm01'
          lxc-start 1449578879.056 ERROR    lxc_start_ui - lxc_start.c:main:341 - The container failed to start.
          lxc-start 1449578879.056 ERROR    lxc_start_ui - lxc_start.c:main:345 - Additional information can be obtained by setting the --logfile and --logpriority options.

そーいや昨日のエラーも「cgroup filesystem is not mounted.」で、今日のエラーも「cgroupfs failed to detect cgroup metadata」で、なんか cgroupfs で設定が足りてないのかもしれません。

ふと…

    # mount
    /dev/mapper/VolGroup-lv_root on / type ext4 (rw)
    proc on /proc type proc (rw)
    sysfs on /sys type sysfs (rw)
    devpts on /dev/pts type devpts (rw,gid=5,mode=620)
    tmpfs on /dev/shm type tmpfs (rw)
    /dev/vda1 on /boot type ext4 (rw)
    none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)

あれ？何か足りてないような気が…、確か中井さんのブログで…[LXC が利用するための /lxc/cgroup ディレクトリをマウント](http://enakai00.hatenablog.com/entry/20110529/1306658627) って書いてありました。なのでそれをやります。

    # mkdir -p /lxc/cgroup
    # mount -t cgroup lxc /lxc/cgroup
    # mount
    /dev/mapper/VolGroup-lv_root on / type ext4 (rw)
    proc on /proc type proc (rw)
    sysfs on /sys type sysfs (rw)
    devpts on /dev/pts type devpts (rw,gid=5,mode=620)
    tmpfs on /dev/shm type tmpfs (rw)
    /dev/vda1 on /boot type ext4 (rw)
    none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
    lxc on /lxc/cgroup type cgroup (rw)

これでもう一度起動してみます。

    # lxc-start -n testvm01 --logfile /tmp/lxc.log
    (中略)
    Bringing up loopback interface:                            [  OK  ]
    Bringing up interface eth0:  
    Determining IP information for eth0... failed.
                                                               [FAILED]

あぁ、はい、確かにそうです、br0 の先には DHCP サーバが居ないので当然ながら IP を取得できませんが、まぁでもこれは起動したと言っていいでしょう。

では、改めて先日の Wakame-vdc の LiveDVD でもう一度挑戦してみたいと思いますが、それはまた明日。

