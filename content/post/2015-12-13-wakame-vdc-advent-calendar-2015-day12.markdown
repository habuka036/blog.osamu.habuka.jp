---
title: "コンテナのこえ"
date: 2015-12-13 04:31:48 +0900
tags: ["Wakame-vdc","AdventCalendar"]
---

Wakame-vdc ユーザの皆さん、磯野家の皆さん、こんにちは。これはオレオレ WakameVdc / OpenVNet Advent Calendar 2015 の12日目のエントリです。

11日目のエントリにて lxc のコンテナが起動できるようにすることができたので、本当はもう少しちゃんとしたパッチを書きたいところですが、その前に11日目のエントリで充分か否かを先に検証したいので、本日は起動したコンテナに接続できるかどうかを確認したいと思います。

<!--more-->

起動したコンテナに早速 ssh で接続…11日目のエントリでは「ssh で接続するもいつまで待ってもログインプロンプトが返ってこない」という現象や「エンターキーを押す毎にプロンプトの表示がホストマシンのものとコンテナのものとがサイクリックに切り替わる」という不思議満点な状況でした。(動画にでも撮っておけばよかった…)

で今日の環境は手作業修正によるものではなく、パッチを当てた LiveDVD から起動してみましたが、やはり「ssh で接続するもいつまで待ってもログインプロンプトが返ってこない」という状況でした。

なのでまずは現状の確認を行ないます。まず、コンテナの lxc 設定ですが、以下のようになっています。

    [root@host-133-130-109-122 wakame]# cat /var/lib/wakame-vdc/instances/i-kg0uuloj/lxc.conf 
    lxc.utsname = i-kg0uuloj
    
    lxc.network.type = veth
    lxc.network.link = br0
    
    lxc.network.veth.pair = vif-3kd9fd4z
    lxc.network.hwaddr = 52:54:00:c3:2f:4f
    lxc.network.flags = up
    lxc.tty = 4
    lxc.pts = 1024
    lxc.rootfs = /var/lib/wakame-vdc/instances/i-kg0uuloj/rootfs
    lxc.rootfs.mount = /var/lib/wakame-vdc/instances/i-kg0uuloj/rootfs
    lxc.mount.entry = proc        /proc                   proc    defaults        0 0
    lxc.mount.entry = sysfs       /sys                    sysfs   defaults        0 0
    #lxc.mount = /var/lib/wakame-vdc/instances/i-kg0uuloj/fstab
    #lxc.cap.drop = mac_admin sys_boot
    lxc.cgroup.devices.deny = a
    # /dev/null and zero
    lxc.cgroup.devices.allow = c 1:3 rwm
    lxc.cgroup.devices.allow = c 1:5 rwm
    # consoles
    lxc.cgroup.devices.allow = c 5:0 rwm
    lxc.cgroup.devices.allow = c 5:1 rwm
    lxc.cgroup.devices.allow = c 4:0 rwm
    lxc.cgroup.devices.allow = c 4:1 rwm
    # /dev/{,u}random"
    lxc.cgroup.devices.allow = c 1:9 rwm
    lxc.cgroup.devices.allow = c 1:8 rwm
    lxc.cgroup.devices.allow = c 136:* rwm
    lxc.cgroup.devices.allow = c 5:2 rwm
    # /dev/rtc
    lxc.cgroup.devices.allow = c 254:0 rwm
    # /dev/kvm
    #lxc.cgroup.devices.allow = c 10:232 rwm
    #lxc.cgroup.devices.allow = c 10:200 rwm

う〜ん、特に変な設定は見当たらないですね…。次にコンテナの内部を見てみます。

    [root@host-133-130-109-122 wakame]# lxc-ls 
    i-kg0uuloj
    
    [root@host-133-130-109-122 wakame]# lxc-attach --name=i-kg0uuloj
    
    [root@kg0uuloj /]# ip a
    4: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
        link/ether 52:54:00:c3:2f:4f brd ff:ff:ff:ff:ff:ff
        inet 192.0.2.20/24 brd 192.0.2.255 scope global eth0
        inet6 fe80::5054:ff:fec3:2f4f/64 scope link 
           valid_lft forever preferred_lft forever
    6: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
        inet6 ::1/128 scope host 
           valid_lft forever preferred_lft forever
    [root@kg0uuloj /]# ps auxwww|grep ssh
    root       407  0.0  0.0  66688  1220 ?        Ss   01:59   0:00 /usr/sbin/sshd
    root       540  0.0  0.0  94252  4296 ?        Ss   02:02   0:00 sshd: root@pts/0 
    root       561  0.0  0.0  94252  4296 ?        Ss   02:10   0:00 sshd: root@pts/1 
    root       716  0.0  0.0 107456   932 pts/8    S+   12:47   0:00 grep ssh
    
    
    
    [root@kg0uuloj /]# /etc/init.d/sshd status
    openssh-daemon (pid  407) を実行中...

はい、ちゃんと NIC に IP が振られ、ssh もちゃんと起動しています。

    [root@kg0uuloj /]# ls -al /home/
    合計 12
    drwxr-xr-x  3 root       root       4096  4月 19 17:29 2015 .
    dr-xr-xr-x 23 root       root       4096 12月 10 01:59 2015 ..
    drwx------  2 wakame-vdc wakame-vdc 4096  4月 19 17:29 2015 wakame-vdc
    
    # getent shadow wakame-vdc
    wakame-vdc:$6$o0NRcBn2YU/D00L$ZfpDzmFviu/YNwJDgI/QNZuLdE.JibVMBEhIs45P2C/GQnZ71bnDofck8U.hk61CvqlgcZfg0vxFLLRes5Pw2/:16544:0:99999:7:::
    
    [root@kg0uuloj /]# ls -al /home/wakame-vdc/
    合計 20
    drwx------ 2 wakame-vdc wakame-vdc 4096  4月 19 17:29 2015 .
    drwxr-xr-x 3 root       root       4096  4月 19 17:29 2015 ..
    -rw-r--r-- 1 wakame-vdc wakame-vdc   18 10月 16 22:56 2014 .bash_logout
    -rw-r--r-- 1 wakame-vdc wakame-vdc  176 10月 16 22:56 2014 .bash_profile
    -rw-r--r-- 1 wakame-vdc wakame-vdc  134  4月 19 17:29 2015 .bashrc

wakame-vdc というユーザが作成されており、何らかのパスワードが設定されてはいますが、ssh の鍵は設定されていないようです。
    
    [root@kg0uuloj /]# ls -al /root/
    合計 32
    dr-xr-x---  3 root root 4096 12月 10 01:59 2015 .
    dr-xr-xr-x 23 root root 4096 12月 10 01:59 2015 ..
    -rw-r--r--  1 root root   18  5月 20 19:45 2009 .bash_logout
    -rw-r--r--  1 root root  176  5月 20 19:45 2009 .bash_profile
    -rw-r--r--  1 root root  176  9月 23 12:59 2004 .bashrc
    -rw-r--r--  1 root root  100  9月 23 12:59 2004 .cshrc
    drwx------  2 root root 4096 12月 10 01:59 2015 .ssh
    -rw-r--r--  1 root root  129 12月  4 06:42 2004 .tcshrc
    [root@kg0uuloj /]# ls -al /root/.ssh/
    合計 12
    drwx------ 2 root root 4096 12月 10 01:59 2015 .
    dr-xr-x--- 3 root root 4096 12月 10 01:59 2015 ..
    -rw------- 1 root root  396 12月 10 01:59 2015 authorized_keys

    [root@kg0uuloj /]# cat /root/.ssh/authorized_keys 
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDZhAOcHSe4aY8GwwLCJ4Et3qUBcyVPokFoCyCrtTZJVUU++B9554ahiVcrQCbfuDlaXV2ZCfIND+5N1UEk5umMoQG1aPBw9Nz9wspMpWiTKGOAm99yR9aZeNbUi8zAfyYnjrpuRUKCH1UPmh6EDaryFNDsxInmaZZ6701PgT++cZ3Vy/r1bmb93YvpV+hfaL/FmY3Cu8n+WJSoJQZ4eCMJ+4Pw/pkxjfuLUw3mFl40RVAlwlTuf1I4bB/m1mjlmirBEU6+CWLGYUNWDKaFBpJcGB6sXoQDS4FvlV92tUAEKIBWG5ma0EXBdJQBi1XxSCU2p7XMX8DhS7Gj/TSu7011 wakame-vdc.pem


一方で root ユーザには ssh 鍵が設定されていました。中身を見ると起動時に指定した LiveDVD のデモ用の ssh 鍵でした。

    [root@kg0uuloj /]# grep -v ^# /etc/ssh/sshd_config |grep -v ^$
    Protocol 2
    SyslogFacility AUTHPRIV
    PermitRootLogin yes
    PasswordAuthentication no
    ChallengeResponseAuthentication no
    GSSAPIAuthentication no
    GSSAPICleanupCredentials yes
    UsePAM yes
    AcceptEnv LANG LC_CTYPE LC_NUMERIC LC_TIME LC_COLLATE LC_MONETARY LC_MESSAGES
    AcceptEnv LC_PAPER LC_NAME LC_ADDRESS LC_TELEPHONE LC_MEASUREMENT
    AcceptEnv LC_IDENTIFICATION LC_ALL LANGUAGE
    AcceptEnv XMODIFIERS
    X11Forwarding yes
    UseDNS no
    Subsystem	sftp	/usr/libexec/openssh/sftp-server

sshd の設定を見ると、root ユーザのログインは許可されておりますが、PasswordAuthentication も ChallengeResponseAuthentication も no になっているため、既定で設定されている wakame-vdc にはログインできなさそうです。

    [root@host-133-130-109-122 wakame]# ssh -l wakame-vdc 192.0.2.20
    Permission denied (publickey).

ね、ほら。

では、root ユーザでログインしたときはコンテナにそもそもちゃんとアクセスできているのか…？ですが、/var/log/secure を見ると以下のようなログが残っていました。

    Dec 10 15:39:23 kg0uuloj sshd[835]: Accepted publickey for root from 192.0.2.1 port 43501 ssh2
    Dec 10 15:39:23 kg0uuloj sshd[835]: pam_unix(sshd:session): session opened for user root by (uid=0)
    Dec 10 15:39:23 kg0uuloj sshd[837]: error: ioctl(TIOCSCTTY): Operation not permitted
    Dec 10 15:39:23 kg0uuloj sshd[837]: error: open /dev/tty failed - could not set controlling tty: No such file or directory

tty が無いって言ってますね…。そりゃぁログインができないわけです…。

    [root@hongo0u5 /]# ls -al /dev/|grep tty
    crw-------  1 root tty  136, 4 12月 12 20:21 2015 console
    crw-------  1 root root 136, 0 12月 12 20:21 2015 tty1
    crw-------  1 root root 136, 1 12月 12 20:21 2015 tty2
    crw-------  1 root root 136, 2 12月 12 20:21 2015 tty3
    crw-------  1 root root 136, 3 12月 12 20:21 2015 tty4

ちなみに Wakame-vdc と関係なく lxc-create -t centos で作成したコンテナの中身を見てみると…

    [root@centos /]# ls -al /dev/|grep tty
    crw-rw-rw-  1 root root 5, 0 12月 10 05:49 2015 tty
    crw-rw-rw-  1 root root 4, 0 12月 10 05:49 2015 tty0
    lrwxrwxrwx  1 root root    8 12月 10 05:50 2015 tty1 -> lxc/tty1
    lrwxrwxrwx  1 root root    8 12月 10 05:50 2015 tty2 -> lxc/tty2
    lrwxrwxrwx  1 root root    8 12月 10 05:50 2015 tty3 -> lxc/tty3
    lrwxrwxrwx  1 root root    8 12月 10 05:50 2015 tty4 -> lxc/tty4

はい、ちゃんと /dev/tty が存在しています。あと気になるのが、tty1〜tty4 に違いがあることとか…

tty1〜tty4 の違いは別として、とりあえず tty と tty0 を作ってみます。

    [root@hongo0u5 /]# mknod -m 666 /dev/tty c 5 0
    [root@hongo0u5 /]# mknod -m 666 /dev/tty0 c 4 0
    [root@hongo0u5 /]# ls -al /dev/|grep tty
    crw-------  1 root tty  136, 4 12月 12 20:21 2015 console
    crw-rw-rw-  1 root root   5, 0 12月 12 20:38 2015 tty
    crw-rw-rw-  1 root root   4, 0 12月 12 20:38 2015 tty0
    crw-------  1 root root 136, 0 12月 12 20:21 2015 tty1
    crw-------  1 root root 136, 1 12月 12 20:21 2015 tty2
    crw-------  1 root root 136, 2 12月 12 20:21 2015 tty3
    crw-------  1 root root 136, 3 12月 12 20:21 2015 tty4

で、再度コンテナに ssh してみます。

    Dec 12 20:44:37 hongo0u5 sshd[570]: Accepted publickey for root from 192.0.2.1 port 43663 ssh2
    Dec 12 20:44:37 hongo0u5 sshd[570]: pam_unix(sshd:session): session opened for user root by (uid=0)
    Dec 12 20:44:37 hongo0u5 sshd[572]: error: ioctl(TIOCSCTTY): Operation not permitted
    Dec 12 20:44:37 hongo0u5 sshd[572]: error: open /dev/tty failed - could not set controlling tty: No such device or address

う〜ん、やっぱりログインできません…。が、エラーが「No such file or directory」から「No such device or address」に変化しています。

うむむぅ…。続きはまた明日…。(「また明日」って、既にこのエントリを書いている時点で13日に入ってしまいましたが…f^^;)

