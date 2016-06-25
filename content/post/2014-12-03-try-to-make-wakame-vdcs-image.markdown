---
title: "Wakame-VDC のマシンイメージを作ってみる"
date: 2014-12-03 00:07:46 +0900
tags: ["Wakame-vdc","AdventCalendar"]
---
これは [Wakame-vdc / OpenVNet Advent Calendar 2014](http://www.adventar.org/calendars/577) 3日目の投稿です。

昨日は [@giraffeforestg](https://twitter.com/giraffeforestg) さんの「[2014年に書いたWakame-vdc/OpenVNetのblogを振り返ってみる](http://giraffeforestg.blog.fc2.com/blog-entry-191.html)」でした。

今日は「ぶっつけ本番で Wakame-VDC のマシンイメージを作ってみよう」です。

今まで中身が気になっていましたが、実は Wakame-VDC のマシンイメージの中身をちゃんと見たことありませんでした。もちろん作ってみたことも。ただ、前にあくしゅの山崎さんに「AWS で動くイメージなら動くよ」って言われたような言われてないような…もしかしたら酔って幻聴を聴いただけかもしれませんが、とりあえず「API だけ互換の Eucalyptus、API 以外は互換もする Wakame-VDC」って昔に言われた気がするので、ぶっつけ本番でマシンイメージを作っても何とかなるだろう？と始めたいと思います。(長い導入だ…)

<!--more-->

で、いくら「ぶっつけ本番」とは言え、少しは事前知識を得たいので Wakame-VDC の既存のマシンイメージを見てみることにします。ってことで、調査対象はよくデモ環境とかで利用する ubuntu-lucid-kvm-md-32.raw.gz を開きます。

    # gunzip ubuntu-lucid-kvm-md-32.raw.gz

すると ubuntu-lucid-kvm-md-32.raw っていうファイルが解凍されるので、このファイルをおもむろに file コマンドで見ます。

    # file ubuntu-lucid-kvm-md-32.raw 
    ubuntu-lucid-kvm-md-32.raw: x86 boot sector

おんやぁ？ AWS 的なフラットファイルを期待したんですが、どうやらこれはパーティションを含んでいる予感がするので、fdisk コマンドで見てみます。(ところで AWS のマシンイメージって今もフラットファイルなんすかね？)

    # fdisk -l ubuntu-lucid-kvm-md-32.raw
    
    Disk ubuntu-lucid-kvm-md-32.raw: 657 MB, 657457152 bytes
    4 heads, 32 sectors/track, 10032 cylinders, total 1284096 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/ptimal): 512 bytes / 512 bytes
    Disk identifier: 0x0000e8e0
    
                         Device Boot      Start         End      Blocks   Id  System
    ubuntu-lucid-kvm-md-32.raw1              63      974609      487273+  83  Linux
    ubuntu-lucid-kvm-md-32.raw2          974848     1224703      124928   82  Linux swap / Solaris

ということで、2つのパーティションを含んでいました。1つ目は Linux パーティションで、2つ目は swap パーティションです。
となると、次にやることは1つ目のパーティションの中身を見ることです。

    # mkdir -p ./raw
    # mount -o loop,offset=$(( 63 * 512 )) ubuntu-lucid-kvm-md-32.raw ./raw/

ってやってマウントすると中身が見れます。

    # ls -al ./raw/
    total 49
    drwxr-xr-x 21 root root  1024 Jul 17  2012 .
    drwxr-xr-x  6 root root   600 Dec  3 00:11 ..
    drwxr-xr-x  2 root root  3072 Jul 17  2012 bin
    drwxr-xr-x  3 root root  1024 Jul 17  2012 boot
    drwxr-xr-x  4 root root  3072 Jul 17  2012 dev
    drwxr-xr-x 54 root root  5120 Jul 26  2012 etc
    drwxr-xr-x  3 root root  1024 Jul 17  2012 home
    lrwxrwxrwx  1 root root    37 Jul 17  2012 initrd.img -> boot/initrd.img-2.6.32-41-generic-pae
    drwxr-xr-x 13 root root  7168 Jul 17  2012 lib
    drwx------  2 root root 12288 Jul 17  2012 lost+found
    drwxr-xr-x  2 root root  1024 Jul 17  2012 media
    drwxr-xr-x  2 root root  1024 Apr 23  2010 mnt
    drwxr-xr-x  2 root root  1024 Jul 17  2012 opt
    drwxr-xr-x  2 root root  1024 Apr 23  2010 proc
    drwx------  2 root root  1024 Jul 17  2012 root
    drwxr-xr-x  2 root root  4096 Jul 17  2012 sbin
    drwxr-xr-x  2 root root  1024 Dec  6  2009 selinux
    drwxr-xr-x  2 root root  1024 Jul 17  2012 srv
    drwxr-xr-x  2 root root  1024 Mar 30  2010 sys
    drwxrwxrwt  2 root root  1024 Jul 17  2012 tmp
    drwxr-xr-x 10 root root  1024 Jul 17  2012 usr
    drwxr-xr-x 13 root root  1024 Jul 17  2012 var
    lrwxrwxrwx  1 root root    34 Jul 17  2012 vmlinuz -> boot/vmlinuz-2.6.32-41-generic-pae

で、これまたおもむろに勘で rc.local を覗いてみると

    # cat ./raw/etc/rc.local 
    /etc/wakame-init md
    exit 0

起動処理の最後で wakame-init っていうコマンドを実行しています。これ何だ？と思い…

    # file ./raw/etc/wakame-init
    ./raw/etc/wakame-init: Bourne-Again shell script, ASCII text executable

確認するとシェルスクリプトです。中身を見たところ、Wakame-VDC テイストではあるものの、IaaS のインスタンスには必要不可欠な処理が書かれていました。さしずめ Wakame-VDC 版 cloud-init という感じでしょうか。

で、github から元になるスクリプトを持ってきてもいいんですが、とりあえず手抜きでこのスクリプトを流用することにしました。ちなみに他に wakame を冠したファイルやディレクトリが無いかを見てみましたが、どうやらこの wakame-init だけのようです。(他の名前で何かあるかもしれないけど、未調査)

    # cp ./raw/etc/wakame-init /tmp/
    # find ./raw/|grep wakame
    ./raw/etc/wakame-init

さて次にマシンイメージを作りますが、いちから OS のインストールとかをやってもいいんですがそこは時間ないので割愛します。代わりに既存の Eucalyptus 用のマシンイメージを流用します。

    # wget http://eucalyptus.machine-image.com/downloads/euca-large-gentoo-2012.07.19-x86_64.tgz
    # tar -xzSf euca-large-gentoo-2012.07.19-x86_64.tgz 
    # file euca-large-gentoo-2012.07.19-x86_64/euca-large-gentoo-2012.07.19-x86_64.img 
    euca-large-gentoo-2012.07.19-x86_64/euca-large-gentoo-2012.07.19-x86_64.img: ReiserFS V3.6

上記のようにマシンイメージをダウンロードして展開します。そう、Eucalyptus 2.x 系の頃のマシンイメージなのでフラットファイルなんです。なのでこれをパーティション含みのファイルに変換します。(大人の事情で細かい説明は端折ります)

    # mkdir -p ./{base,new}_img
    # mount -o loop,ro euca-large-gentoo-2012.07.19-x86_64/euca-large-gentoo-2012.07.19-x86_64.img ./base_img/
    # df -h
    Filesystem      Size  Used Avail Use% Mounted on
    dev/loop1      4.5G  2.7G  1.9G  59% /mnt/sda6/base_img
    (使用サイズが 2.7GB なので、とりあえず 4GB の空ファイルを作成する)
    # truncate -s 4GB euca-large-gentoo-2012.07.19-x86_64.wakame-vdc.img
    (作成した空ファイルをループバックデバイスに結びつける)
    # losetup -f --show euca-large-gentoo-2012.07.19-x86_64.wakame-vdc.img 
    /dev/loop2
    (パーティショニングを実施)
    # sfdisk /dev/loop2 << EOF
    > ,400,L
    > ,,S
    > ; 
    > ;
    > EOF
    # losetup -d /dev/loop2 
    # kpartx -a euca-large-gentoo-2012.07.19-x86_64.wakame-vdc.img
    # mkfs.reiserfs /dev/mapper/loop2p1
    # mkswap /dev/mapper/loop2p2 
	    # mount /dev/mapper/loop2p1 ./new_img/
    (フラットファイルの中身をコピー)
    # rsync -PHSav ./base_img/ ./new_img/
    # umount ./base_img
    # umount ./new_img
    # ls -l euca-large-gentoo-2012.07.19-x86_64.wakame-vdc.img
    -rw-r--r-- 1 root root 4000000000 Dec  3 11:43 euca-large-gentoo-2012.07.19-x86_64.wakame-vdc.img
    # ls -al /dev/loop2
    brw-rw---- 1 root disk 7, 2 Dec  3 11:10 /dev/loop2
    # echo "0 $(( 4000000000 / 512 )) linear 7:2 0" | dmsetup create hda
    # ls -al /dev/mapper/
    total 0
    drwxr-xr-x  2 root root      80 Dec  3 13:08 .
    drwxr-xr-x 16 root root    4360 Dec  3 13:08 ..
    crw-------  1 root root 10, 236 Nov 28 17:11 control
    lrwxrwxrwx  1 root root       7 Dec  3 13:08 hda -> ../dm-0
    # kpartx -a /dev/mapper/hda 
    # ls -l /dev/mapper/hda*
    lrwxrwxrwx 1 root root      7 Dec  3 13:08 /dev/mapper/hda -> ../dm-0
    brw-rw---- 1 root disk 252, 1 Dec  3 13:09 /dev/mapper/hda1
    brw-rw---- 1 root disk 252, 3 Dec  3 13:09 /dev/mapper/hda2
    # mount /dev/mapper/hdb1 ./new_img/
    # cp -a ./new_img/boot/grub/device.map ./new_img/boot/grub/device.map.orig
    # echo '(hd0) /dev/mapper/hda' > ./new_img/boot/grub/device.map
    # grub-install --root-directory=./new_img/ /dev/mapper/hdb
    Installing for i386-pc platform.
    grub-install.real: error: cannot find a GRUB drive for /dev/mapper/hdb.  Check your device.map.

    # mount -t proc none ./new_img/proc
    # mount -o bind /dev/ ./new_img/dev/
    # chroot ./new_img/ /bin/bash  
    # sed -i -e 's/hd0,1/hd0,0/' boot/grub/grub.conf
    # grub --device-map=/boot/grub/device.map
    grub> root (hd0,0)
     Filesystem type is reiserfs, partition type 0x83
    
    grub> setup (hd0)
     Checking if "/boot/grub/stage1" exists... yes
     Checking if "/boot/grub/stage2" exists... yes
     Checking if "/boot/grub/reiserfs_stage1_5" exists... yes
     Running "embed /boot/grub/reiserfs_stage1_5 (hd0)"... failed (this is not fatal)
     Running "embed /boot/grub/reiserfs_stage1_5 (hd0,0)"... failed (this is not fatal)
     Running "install /boot/grub/stage1 (hd0) /boot/grub/stage2 p /boot/grub/menu.lst "... succeeded
    Done.

う〜ん、stage1_5 の追加が失敗している…。これ駄目な気がしますが「ぶっつけ本番」の名のもと進めます。

    (chroot 下から抜けます)
    # exit
    # umount ./new_img/proc/ ./new_img/dev 
    # cp -a /tmp/wakame-init ./new_img/etc/
    # cat >> ./new_img/etc/rc.local <<EOF
    > /etc/wakame-init md
    > exit 0
    > EOF
    # chmod 755 ./new_img/etc/rc.local 
    # sync
    # # umount ./new_img 
    # kpartx -d /dev/mapper/hda
    # dmsetup remove hda
    # losetup -d /dev/loop2
    # file euca-large-gentoo-2012.07.19-x86_64.wakame-vdc.img 
    euca-large-gentoo-2012.07.19-x86_64.wakame-vdc.img: x86 boot sector

ということで、何とか空のファイルがブータブルイメージのように見えます。で、これを Wakame-VDC に登録します。

    # /opt/axsh/wakame-vdc/dcmgr/bin/vdc-manage -e
    vdc-manage>> backupobject add \
     > --uuid bo-gentoo \
     > --display-name "Gentoo Linux" \
     > --storage-id bkst-local \
     > --object-key euca-large-gentoo-2012.07.19-x86_64.wakame-vdc.img.gz \
     > --size 841701 \
     > --allocation-size 4194304 \
     > --container-format gz \
     > --checksum 98f623f6d035b259496f18899c37bdb3
    bo-gentoo
    
    vdc-manage>> image add local bo-gentoo \
     > --account-id a-shpoolxx \
     > --uuid wmi-gentoo \
     > --root-device /dev/sda1 \
     > --display-name "Gentoo Linux 2012.07.19"
    wmi-gentoo

と、登録の各種パラメータは勘で入れてるので間違っているかもしれません。まー「ぶっつけ本番」ですので…。

で、Wakame-VDC の WebUI を開くと…おぉ、登録したイメージが見れます。(当たり前だ)

![](/images/wakame-vdc.adventcalendar.2014.1203-01.png )

このイメージを指定して起動をかけると…おぉ、起動します。

![](/images/wakame-vdc.adventcalendar.2014.1203-02.png )

で、この起動したインスタンスに…残念ながら接続できませんでした。というのも心当たりは山程あるのですが、何故起動しなかったのか？を調べる話は他の日のネタにしようと思います。すみません<(＿ ＿)>

----
ということで [Wakame-vdc / OpenVNet Advent Calendar 2014](http://www.adventar.org/calendars/577) 3日目の投稿でした。次の日は…まだエントリしていないようですが、僕のようなこんなグダグダな内容のエントリもあるので、気軽にエントリしてみてください。


