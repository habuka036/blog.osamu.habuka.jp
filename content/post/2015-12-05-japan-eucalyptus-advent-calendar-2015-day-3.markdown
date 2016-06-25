---
title: "Eucalyptus LiveDVD を作ってみる 3日目"
date: 2015-12-05 05:05:37 +0900
tags: ["Eucalyptus","AdventCalendar"]
---

これは [(1枚目) Japan Eucalyptus Advent Calendar 2015](http://www.adventar.org/calendars/752) の3日目のエントリです。

え？更新が途切れた？そんなバカな…。2日目の続きをやりまぁす。

<!--more-->

2日目のログをもう少し遡ると以下のように変なところで umount が走ってコケてます。

      Installing: comps-extras                 ##################### [456/456]warning: %posttrans(nfs-uti
    -1:1.2.3-64.el6.x86_64) scriptlet failed, exit status 6
    error reading information on service NetworkManager: No such file or directory
    error reading information on service sshd: No such file or directory
    + echo RUN_FIRSTBOOT=NO
    + sed -i -e 's/^Defaults    requiretty/#Defaults    requiretty/' /etc/sudoers
    + /usr/sbin/useradd -c 'LiveMedia default user' eucalyquitous
    + /usr/bin/passwd -d eucalyquitous
    + echo 'eucalyquitous     ALL=(ALL)     NOPASSWD: ALL'
    + /bin/sed -i -e 's|^\(exec /sbin/mingetty \)\(.*\)|\1 --autologin eucalyquitous \2|' /etc/init/tty.c
    f
    umount: /mnt/livecd/tmp/imgcreate-6F78B4/install_root/proc: device is busy.
            (In some cases useful info about processes that use
             the device is found by lsof(8) or fuser(1))
    umount: /mnt/livecd/tmp/imgcreate-6F78B4/install_root: device is busy.
            (In some cases useful info about processes that use
             the device is found by lsof(8) or fuser(1))
    loop: can't delete device /dev/loop0: Device or resource busy
    e2fsck 1.41.12 (17-May-2010)
    _Eucalyquitous: recovering journal
    Pass 1: Checking inodes, blocks, and sizes

んじゃー、ビルドが成功している Wakame-vdc の LiveDVD ではここはどうなっているのかと言うと、

      Installing: gnome-backgrounds            ##################### [907/907]error reading information
     on service NetworkManager: No such file or directory
    cp: cannot stat `/etc/resolv.conf': No such file or directory
    + echo RUN_FIRSTBOOT=NO
    + sed -i -e 's/^Defaults    requiretty/#Defaults    requiretty/' /etc/sudoers
    + /usr/sbin/useradd -c 'LiveMedia default user' wakame
    + /usr/bin/passwd -d wakame
    + echo 'wakame     ALL=(ALL)     NOPASSWD: ALL'
    + /bin/sed -i -e 's|^\(exec /sbin/mingetty \)\(.*\)|\1 --autologin wakame \2|' /etc/init/tty.conf
    + sed -i -e 's/\[daemon\]/[daemon]\nTimedLoginEnable=true\nTimedLogin=wakame\nTimedLoginDelay=10/' /etc/gdm/custom.conf

やはり変な umount は発生していないようです。

違う点と言えば Wakame-vdc LiveDVD のほうでは他にも処理を入れていることと、%post --nochroot のセクションもあるという点です。なのでバッドノウハウな気もしますが、以下の処理を追加で kickstart に入れて再度ビルドしてみました。

    %post --nochroot
    
    cat > /root/postnochroot-install << EOF_postnochroot
    #!/bin/bash
    
    rm -rf ${INSTALL_ROOT}/usr/share/{doc,man,info}
    
    EOF_postnochroot
    
    /bin/bash -x /root/postnochroot-install 2>&1 | tee /root/postnochroot-install.log

うーん、やっぱり以下のように失敗します。

    + rm -rf /mnt/livecd/tmp/imgcreate-O60XR_/install_root/usr/share/doc /mnt/livecd/tmp/imgcreate-O60XR_
    /install_root/usr/share/man /mnt/livecd/tmp/imgcreate-O60XR_/install_root/usr/share/info
    umount: /mnt/livecd/tmp/imgcreate-O60XR_/install_root/proc: device is busy.
            (In some cases useful info about processes that use
             the device is found by lsof(8) or fuser(1))
    umount: /mnt/livecd/tmp/imgcreate-O60XR_/install_root: device is busy.
            (In some cases useful info about processes that use
             the device is found by lsof(8) or fuser(1))
    loop: can't delete device /dev/loop1: Device or resource busy

fuser で見てみます。

    + rm -rf /mnt/livecd/tmp/imgcreate-lWxyO0/install_root/usr/share/doc /mnt/livecd/tmp/imgcreate-lWxyO0
    /install_root/usr/share/man /mnt/livecd/tmp/imgcreate-lWxyO0/install_root/usr/share/info
    + /sbin/fuser -vm /mnt/livecd/tmp/imgcreate-lWxyO0/install_root/proc/
                         USER        PID ACCESS COMMAND
    /mnt/livecd/tmp/imgcreate-lWxyO0/install_root/proc/:
                         root       1355 f.... rsyslogd
                         root       1445 f.... acpid
                         haldaemon   1457 f.... hald
                         root       3440 F.... libvirtd
                         root       7273 F.... libvirtd
                         root      13917 F.... libvirtd
                         root      14660 F.... libvirtd
                         root      19529 F.... libvirtd
                         root      28104 F.... libvirtd
    umount: /mnt/livecd/tmp/imgcreate-lWxyO0/install_root/proc: device is busy.
            (In some cases useful info about processes that use
             the device is found by lsof(8) or fuser(1))
    umount: /mnt/livecd/tmp/imgcreate-lWxyO0/install_root: device is busy.
            (In some cases useful info about processes that use
             the device is found by lsof(8) or fuser(1))
    loop: can't delete device /dev/loop5: Device or resource busy

libvirtd....？あぁ！もしかして！と思い Eucalyptus に関するパッケージのインストール指定を外してビルドしてみると

    + rm -rf /mnt/livecd/tmp/imgcreate-tERkpC/install_root/usr/share/doc /mnt/livecd/tmp/imgcreate-tERkpC
    /install_root/usr/share/man /mnt/livecd/tmp/imgcreate-tERkpC/install_root/usr/share/info
    + /sbin/fuser -vm /mnt/livecd/tmp/imgcreate-tERkpC/install_root/proc/
                         USER        PID ACCESS COMMAND
    /mnt/livecd/tmp/imgcreate-tERkpC/install_root/proc/:
                         root       1355 f.... rsyslogd
                         root       1445 f.... acpid
                         haldaemon   1457 f.... hald
                         root       3440 F.... libvirtd
                         root       7273 F.... libvirtd
                         root      13917 F.... libvirtd
                         root      14660 F.... libvirtd
                         root      19529 F.... libvirtd
                         root      28104 F.... libvirtd
    e2fsck 1.41.12 (17-May-2010)
    Pass 1: Checking inodes, blocks, and sizes

あれ？半分勘違い。てっきり libvirtd はインストールした Eucalyptus のパッケージに動かされたのかと思いましたが、どうやら表示されているのはビルド環境のホスト側の情報だったみたいです。が、一方で umount のエラーが消えたのはやはり Eucalyptus のパッケージが関係しているようです。

ということで、Eucalyptus のパッケージをインストールすると何が起きるのかを見てみたいと思います。

まずインストールされるパッケージは以下。

    # grep "Installing: euca" 20151203-4.eucaly.log
      Installing: eucalyptus                   ##################### [251/456] 
      Installing: eucalyptus-blockdev-utils    ##################### [259/456]/var/tmp/rpm-tmp.EoxuZE: line 2: /sbin/service: No such file or directory
      Installing: eucalyptus-axis2c-common     ##################### [377/456] 
      Installing: euca2ools                    ##################### [379/456] 
      Installing: eucalyptus-admin-tools       ##################### [380/456] 
      Installing: eucalyptus-imaging-toolkit   ##################### [389/456] 
      Installing: eucanetd                     ##################### [418/456] 
      Installing: eucalyptus-common-java-libs  ##################### [423/456] 
      Installing: eucalyptus-common-java       ##################### [424/456] 
      Installing: eucalyptus-nc                ##################### [442/456]Stopping libvirtd daemon: [FAILED]
      Installing: eucalyptus-sc                ##################### [444/456]Starting SCSI target daemon: [  OK  ]
      Installing: eucalyptus-cloud             ##################### [445/456] 
      Installing: eucalyptus-walrus            ##################### [446/456] 
      Installing: eucalyptus-cc                ##################### [447/456] 
      Installing: eucaconsole                  ##################### [449/456] 
      Installing: eucalyptus-service-image     ##################### [450/456] 

これらのパッケージをダウンロードして何が実行されるかを見ます。で、全部をここに貼るのもアレなので原因になるものだけ。

    # rpm -qp --scripts eucalyptus-nc-4.2.0-0.0.24054.2.el6.x86_64.rpm 
    警告: eucalyptus-nc-4.2.0-0.0.24054.2.el6.x86_64.rpm: ヘッダ V4 RSA/SHA1 Signature, key ID c1240596: NOKEY
    postinstall scriptlet (using /bin/sh):
    if [ -e /etc/rc.d/init.d/libvirtd ]; then
        chkconfig --add libvirtd
        /sbin/service libvirtd restart
    fi
    chkconfig --add eucalyptus-nc
    usermod -a -G kvm eucalyptus
    exit 0
    preuninstall scriptlet (using /bin/sh):
    if [ "$1" = "0" ]; then
        if [ -f /etc/eucalyptus/eucalyptus.conf ]; then
            /sbin/service eucalyptus-nc stop
        fi
        chkconfig --del eucalyptus-nc
    fi
    exit 0

あぁ、やっぱり libvirtd を起動してますね。あとは

    # rpm -qp --scripts eucalyptus-sc-4.2.0-0.0.24054.2.el6.x86_64.rpm 
    警告: eucalyptus-sc-4.2.0-0.0.24054.2.el6.x86_64.rpm: ヘッダ V4 RSA/SHA1 Signature, key ID c1240596: NOKEY
    postinstall scriptlet (using /bin/sh):
    if [ -e /etc/rc.d/init.d/tgtd ]; then
        chkconfig --add tgtd
        /sbin/service tgtd start
    fi
    exit 0

あぁ、tgtd も起動してますね。他は無さそうですので、この2つを停止するように kickstart ファイルに書いてビルドします。

    + /bin/sed -i -e 's|^\(exec /sbin/mingetty \)\(.*\)|\1 --autologin eucalyquitous \2|' /etc/init/tty.c
    onf
    + /sbin/service libvirtd stop
    Stopping libvirtd daemon:                                  [  OK  ]
    + /sbin/service tgtd stop
    Stopping SCSI target daemon:                               [  OK  ]
    + rm -rf /mnt/livecd/tmp/imgcreate-ST404K/install_root/usr/share/doc /mnt/livecd/tmp/imgcreate-ST404K
    /install_root/usr/share/man /mnt/livecd/tmp/imgcreate-ST404K/install_root/usr/share/info
    e2fsck 1.41.12 (17-May-2010)
    Pass 1: Checking inodes, blocks, and sizes

やったー。成功です。ってことで、明日はこの作成した ISO ファイルで起動して Eucalyptus のセットアップができるかを試してみます。

