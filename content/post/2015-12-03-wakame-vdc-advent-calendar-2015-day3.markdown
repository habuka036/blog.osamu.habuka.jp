---
title: "おれは KVM をやめるぞ！ジョジョーーッ！"
date: 2015-12-03 15:08:31 +0900
tags: ["Wakame-vdc","AdventCalendar"]
---

Wakame-vdc ユーザの皆さん、磯野家の皆さん、こんにちは。これはオレオレ WakameVdc / OpenVNet Advent Calendar 2015 の3日目のエントリです。

エントリのタイトルが「Wakame-vdc LiveDVD 16.1 を作る X日目」だと面白味がないので、もうちょっと内容とタイトルが結びついてわかり易くなるようにしました。

ということで、今日は昨日に引き続き Wakame-vdc LiveDVD をデバッグしてみようという話です。

<!--more-->

昨日はインスタンスを起動したけど接続できない、何故か？が問題でしたので、まずは現状を確認してみましょう。

インスタンスのプロセスは以下のようなパラメータで起動しています。(わかり易くするために適宜改行してます)

    /usr/libexec/qemu-kvm \
    -m 256 
    -smp 2 
    -name vdc-i-oo5ginzo 
    -pidfile /var/lib/wakame-vdc/instances/i-oo5ginzo/kvm.pid 
    -daemonize 
    -monitor telnet:127.0.0.1:28684,server,nowait 
    -no-kvm-pit-reinjection 
    -vnc 127.0.0.1:23172 
    -serial telnet:127.0.0.1:29521,server,nowait 
    -drive file=/var/lib/wakame-vdc/instances/i-oo5ginzo/vol-fnqwdqv3,id=vol-fnqwdqv3-drive,if=none,serial=vol-fnqwdqv3,cache=none,aio=native 
    -device ide-drive,id=vol-fnqwdqv3,drive=vol-fnqwdqv3-drive,bootindex=0 
    -drive file=/var/lib/wakame-vdc/instances/i-oo5ginzo/metadata.img,id=metadata-drive,if=none,serial=metadata,cache=none,aio=native 
    -device ide-drive,id=metadata,drive=metadata-drive 
    -net nic,vlan=0,macaddr=52:54:00:74:a4:6d,model=e1000,addr=10 
    -net tap,vlan=0,ifname=vif-8azuiln0,script=no,downscript=no

[去年のアドカレ](http://blog.osamu.habuka.jp/blog/2014/12/05/debug-for-wakame-vdcs-instance/) でも書きましたが、まずは stone で tcprelay して vnc に接続してみようじゃないですか。

    # /usr/local/bin/stone -d 127.0.0.1:23172 133.130.116.239:23172

![](/images/wakame-vdc.adventcalendar.2015.1203-01.png )

おぅ…接続切られます…orz

一方で telnet も駄目です。

    # telnet 127.0.0.1 29521
    Trying 127.0.0.1...
    Connected to 127.0.0.1.
    Escape character is '^]'.

じゃぁ、デバッグログでも吐かせてみようと、/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/hypervisor/linux_hypervisor/kvm.rb をイジって、qemu-kvm のコマンドラインに「-d /tmp/wakame-vm.log」って付けるようにしてみて、hva を再起動してみたんですが…

    # /opt/axsh/wakame-vdc/dcmgr/bin/vdc-manage host show
    UUID                 Node ID               Hypervisor  Architecture  Usage  Status   Scheduling
    hn-node3ab67f68a978  hva.node3ab67f68a978  kvm         x86_64        2%     offline  true

残念ながら offline から状態が変わらなくなりました。

で、今さらですが、実は私の新 ConoHa のアカウントは以下のように VT が入ってないので、そもそも ConoHa を使うときに Wakame-vdc の hva で`KVM を選択すると駄目なような気がします。

こんな感じ。

    $ egrep "(vmx|smx)" /proc/cpuinfo | wc -l
    0

ということで、Wakame-vdc LiveDVD のデフォルトの仮想化ドライバは KVM でなくLXC を使うように変更することにします。

相変らず刻んですみませんが、LXC 化させる話は明日以降に。

