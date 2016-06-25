---
title: "Cirros を Eucalyptus4 で動かしてみる"
date: 2014-12-15 12:28:49 +0900
tags: ["Eucalyptus","AdventCalendar"]
---

今日は Cirros を Eucalyptus4 のインスタンスとして動かしてみようと思います。

<!--more-->

Eucalyptus3 のときは[こんな風](https://www.eucalyptus.com/blog/2014/02/02/cirros-perfect-machine-image-eucalyptus-cloud-debugging)に eustore コマンドを使って簡単にインストールできてたのですが、残念ながら Eucalyptus4 では eustore コマンドは廃止されたようです。

なので、Eucalyptus4 では手動で入れてみますが、特に難しい作業ではありません。

    (イメージファイルをダウンロードします)
    # wget http://download.cirros-cloud.net/0.3.1/cirros-0.3.1-x86_64-disk.img
    (ファイル形式が qcow2 なので raw 形式に変換します)
    # file cirros-0.3.1-x86_64-disk.img 
    cirros-0.3.1-x86_64-disk.img: Qemu Image, Format: Qcow , Version: 2
    # qemu-img convert -O raw cirros-0.3.1-x86_64-disk.img cirros-0.3.1-x86_64-disk.raw
    (Eucalyptus に登録します)
    # euca-install-image -b cirros031 -r x86_64 -i cirros-0.3.1-x86_64-disk.raw -n cirros031 --virtualization-type hvm
    /var/tmp/bundle-O0s4JM/cirros-0.3.1-x86_64-disk.raw.part.00 100% |===================|  10.00 MB  25.81 MB/s Time: 0:00:00
    /var/tmp/bundle-O0s4JM/cirros-0.3.1-x86_64-disk.raw.part.01 100% |===================|   1.81 MB   7.52 MB/s Time: 0:00:00
    /var/tmp/bundle-O0s4JM/cirros-0.3.1-x86_64-disk.raw.manifest.xml 100% |==============|   3.49 kB   1.61 kB/s Time: 0:00:02
    IMAGE	emi-b27ea8b4

以上であっという間に完了です。あとはインスタンスを起動するだけです。

……が、インスタンスのコンソールを確認すると、以下のような出力が延々て出続け、起動が成功しませんでした。

    SeaBIOS (version seabios-0.6.1.2-28.el6)
    
    
    Machine UUID e0c370a4-513a-c61d-c8f3-398b3c25420d
    
    
    
    
    
    
    gPXE (http://etherboot.org) - 00:03.0 C100 PCI2.10 PnP BBS PMM01E0@10 C100
    
    
    Press Ctrl-B to configure gPXE (PCI 00:03.0)...
    
                                                                                   
    
    
    
    
    
    Booting from Hard Disk...
    
    
    GRUB Loading stage1.5.
    
    
    
    
    
    
    GRUB loading, please wait...
    
    
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    
    Starting up ...
    
    
    ogle, Inc.
    Serial Graphics Adapter 12/07/11
    SGABIOS $Id: sgabios.S 8 2010-04-22 00:03:40Z nlaredo $ (mockbuild@c6b18n1.dev.centos.org) Wed Dec  7 17:04:47 UTC 2011
    Term: 80x24
    4 0
    

ということで、見事に失敗している様子です。

