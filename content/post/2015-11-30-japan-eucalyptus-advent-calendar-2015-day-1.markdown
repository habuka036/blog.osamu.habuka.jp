---
title: "Eucalyptus LiveDVD を作ってみる 1日目"
date: 2015-11-30 23:22:39 +0900
tags: ["Eucalyptus","AdventCalendar"]
---

これは [(1枚目) Japan Eucalyptus Advent Calendar 2015](http://www.adventar.org/calendars/752) の1日目のエントリです。

えーと、何をトチ狂ったのか、一人で25日間ですよ。2013年に同じことやって帯状疱疹でまともにエントリを埋められなかった失態を忘れてしまったんですかね？

僕らが敬愛する利根川先生の声が聞こえてきますよ。


__バカがっ…！__

__とんでもない誤解だ…！__

__世間というものはとどのつまり__

__Eucalyptus Advent Calendar には何ひとつエントリしない…！__


うん、まぁ、気のせいですね。

で、本当はこのエントリで Eucalyptus LiveDVD を公開し、後ろの日付の人達にバトンを渡す予定でしたが、知ってました？リレーって独りじゃ出来無いっぽいっすよ？マジで。

ってことで、検索 bot 以外に誰も見てないと思いますので、超マイペースで LiveDVD の作成日記を25日ぐらいかけてゆっくりやろうと思います。なるべく25日間失敗し続けますように。

<!--more-->

まず今日は背伸びをせずに「LiveDVD 作成用環境の準備」と「Eucalyptus のパッケージが入っているだけの LiveDVD を作る」の2つをやって終わりとします。

# LiveDVD 作成用環境の準備

もうね、説明とか丁寧にするの面倒なので、[Wakame-vdc LiveDVD の作成環境の作り方](https://github.com/axsh/iso-no-wakame/wiki/HowtoCreateEnvironmentForLiveDVD)でも参考にして作ってください。

ほんとチョー簡単。

# Eucalyptus のパッケージが入っているだけの LiveDVD を作る

kickstart ファイル「eucalyquitous.ks」を以下の内容で保存します。

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
    

で、このファイルのある場所で以下を実行します。

    # LANG=C livecd-creator --config=eucalyquitous.ks --fslabel=Eucalyquitous

で、しばらくすると ISO ファイルが作成されます。ちなみに↑のksファイルで作成すると約CD1枚分ほどのサイズの ISO ファイルが出来ました。

    # ls -hal Eucalyquitous.iso 
    -rw-r--r-- 1 root root 634M Dec  1 01:33 Eucalyquitous.iso

明日はこの ISO ファイルから起動してみます。

