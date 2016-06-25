---
title: "Eucalyptus LiveDVD を作ってみる 2日目"
date: 2015-12-01 23:03:40 +0900
tags: ["Eucalyptus","AdventCalendar"]
---

これは [(1枚目) Japan Eucalyptus Advent Calendar 2015](http://www.adventar.org/calendars/752) の2日目のエントリです。

どうせ誰も見てないんだからくだらない前置きとかもう止めます。チラ裏はチラ裏なりに駄文でも書いておけばいんじゃないかと思いますが、あまりにも駄文すぎて資源の無駄遣いなので、もう本題に入ります。

<!--more-->

さて昨日作った ISO ファイルから起動してみるんですが、root ユーザのパスワードを設定してなかったのでログインできずに意味のない LiveDVD になってしまってました…orz

ということで、昨日の ks ファイルを修正し、以下のようにしてパスワードなしでログインできるようにしました。

    lang en_US.UTF-8
    keyboard us
    timezone Asia/Tokyo
    auth --useshadow --enablemd5
    selinux --disabled
    firewall --disabled
    
    repo --name=base        --baseurl=http://ftp.iij.ad.jp/pub/linux/centos/6.7/os/$basearch
    repo --name=updates     --baseurl=http://ftp.iij.ad.jp/pub/linux/centos/6.7/updates/$basearch
    repo --name=extras      --baseurl=http://ftp.iij.ad.jp/pub/linux/centos/6.7/extras/$basearch
    repo --name=epel        --baseurl=http://download.fedoraproject.org/pub/epel/6/$basearch
    repo --name=eucalyptus  --baseurl=http://downloads.eucalyptus.com/software/eucalyptus/4.2/centos/6/$basearch
    repo --name=euca2ools   --baseurl=http://downloads.eucalyptus.com/software/euca2ools/3.3/centos/6/$basearch
    
    xconfig --startxonboot
    
    #services --disabled=NetworkManager,network,sshd
    
    %pre
    
    %packages
    bash
    kernel
    syslinux
    passwd
    policycoreutils
    chkconfig
    authconfig
    rootfiles
    comps-extras
    xkeyboard-config
    dhclient
    livecd-tools
    # for eucalyptus
    eucalyptus-nc
    eucalyptus-cloud
    eucalyptus-service-image
    eucalyptus-cloud
    eucaconsole
    eucalyptus-cc
    eucalyptus-sc
    eucalyptus-walrus
    
    
    %post
    
    echo "Eucalyptus LiveDVD release 15.12 (Final)" > /etc/redhat-release
    LIVE_USER="eucalyquitous"
    
    cat > /root/post-install << EOF_post
    #!/bin/bash
    
    # turn off firstboot for livecd boots
    echo "RUN_FIRSTBOOT=NO" > /etc/sysconfig/firstboot
    
    sed -i -e "s/^Defaults    requiretty/#Defaults    requiretty/" /etc/sudoers
    
    ## create the LiveCD default user
    # add default user with no password
    /usr/sbin/useradd -c "LiveMedia default user" $LIVE_USER
    /usr/bin/passwd -d $LIVE_USER > /dev/null
    # give default user sudo privileges
    echo "$LIVE_USER     ALL=(ALL)     NOPASSWD: ALL" >> /etc/sudoers
    /bin/sed -i -e "s|^\(exec /sbin/mingetty \)\(.*\)|\1 --autologin $LIVE_USER \2|" /etc/init/tty.conf
    
    EOF_post
    
    /bin/bash -x /root/post-install 2>&1 | tee /root/post-install.log
    
    #echo "timeout 40;" > /etc/dhclient.conf
    cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime

が、何故かオートログインできないどころか、sudo でスーパーユーザにすら昇格できない…。そしてあーだこーだやっているうちに、とうとう以下のようなエラーまで出てくる始末…。

    Free inodes count wrong (61919, counted=61935).
    Fix? yes
    
    
    _Eucalyquitous: ***** FILE SYSTEM WAS MODIFIED *****
    _Eucalyquitous: 36369/98304 files (0.1% non-contiguous), 371591/379653 blocks
    /usr/lib/python2.6/site-packages/imgcreate/errors.py:45: DeprecationWarning: BaseException.message has been deprecated as of Python 2.6
      return unicode(self.message)
    Error creating Live CD : fsck after resize returned an error!  image to debug at /tmp/resize-image-Uwn9U7
     
    
    lazy umount succeeded on /mnt/livecd/tmp/imgcreate-08HiGO/install_root//proc
    Unmounting directory /mnt/livecd/tmp/imgcreate-08HiGO/install_root failed, using lazy umount
    lazy umount succeeded on /mnt/livecd/tmp/imgcreate-08HiGO/install_root

なんか不穏なエラー…。

といったところでタイムアップです…中途半端な状況ですがデバッグは明日。

