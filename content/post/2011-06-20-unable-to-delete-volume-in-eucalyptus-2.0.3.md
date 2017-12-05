+++
date = "2011-06-20T01:13:20+09:00"
title = "Eucalyptus 2.0.3 で DeleteVolume できなくなる件"
tags = ["Eucalyptus"]
+++

ということで、突如 DeleteVolume ができなくなった我が家の Eucalyptus ですが、以下のような感じです。

DescribeVolume の結果は特に問題なし

```
 [root@eucalyptus ~]# euca-describe-volumes
 VOLUME  vol-58C70625     1              cluster0        available       2011-06-09T04:25:15.757Z
 VOLUME  vol-592B061A     2              cluster0        available       2011-06-09T04:25:28.287Z
```
 
DeleteVolumeを実施

```
 [root@eucalyptus ~]# euca-delete-volume vol-58C70625
 VOLUME  vol-58C70625
```
 
でも消えてない…

```
 [root@eucalyptus ~]# euca-describe-volumes
 VOLUME  vol-58C70625     1              cluster0        available       2011-06-09T04:25:15.757Z
 VOLUME  vol-592B061A     2              cluster0        available       2011-06-09T04:25:28.287Z
```
 
/var/log/messages には LVM の lv がないというログが…

```
 Jun 20 02:01:20 eucalyptus tgtd: backed_file_open(92) Could not open /dev/vg-1iOYfw../lv-CdT28w.., No such file or directory
```
 
ちなみに DB には問題なさげ…

```
 [root@eucalyptus ~]# grep vg-1iOYfw.. -r /var/lib/eucalyptus/db/
 /var/lib/eucalyptus/db/eucalyptus_storage.script:INSERT INTO ISCSIVOLUMEINFO VALUES('ff808081307291f4013072a5d7960014','/dev/loop1',NULL,'lv-CdT28w..','/dev/loop1','cluster0',2,NULL,'available','vg-1iOYfw..','vol-592B061A',NULL,1,'iqn.2009-06.com.eucalyptus.cluster0:store2','eucalyptus',2)
```
 
ループバックデバイスにもちゃんと…

```
 [root@eucalyptus ~]# losetup -a
 /dev/loop0: [0813]:118587831 (//var/lib/eucalyptus/volumes/vol-58C70625)
 /dev/loop1: [0813]:118587832 (//var/lib/eucalyptus/volumes/vol-592B061A)
```
 
実体もある…

```
 [root@eucalyptus ~]# ls -al /var/lib/eucalyptus/volumes/vol-592B061A
 -rw-r--r-- 1 eucalyptus eucalyptus 2151677952 Jun  9 13:25 /var/lib/eucalyptus/volumes/vol-592B061A
``` 

削除しようとしたボリュームに対する tgtadm コマンドが止ってる…？

```
 [root@eucalyptus ~]# ps auxww | grep [i]scsi
 root      4221  0.0  0.0   3796   440 ?        S    02:01   0:00 tgtadm --lld iscsi --op new --mode logicalunit --tid 2 --lun 1 -b /dev/vg-1iOYfw../lv-CdT28w..
 root     20681  0.0  0.0   3796   436 ?        S    03:04   0:00 tgtadm --lld iscsi --op unbind --mode target --tid 1 -I ALL
```
 
確かにない…

```
 [root@eucalyptus ~]# ls -al /dev/vg-1iOYfw../lv-CdT28w..
 ls: /dev/vg-1iOYfw../lv-CdT28w..: No such file or directory
```
 
っていうか、すごく…何もないです…

```
 [root@eucalyptus ~]# ls /dev/ | grep vg- | wc -l
 0
```

で、原因追求しようとしたけど、あまり深く追う余裕がないので、さっくりと諦めて復旧…っていうか、「再インストールするよりはマシだよ」レベルのあまり良くない直し方で。

CLC/Walrus/SC を停止 (同居環境なため、Walrus は完全に巻き添え)

```
 [root@eucalyptus ~]# /etc/init.d/eucalyptus-cloud stop
 Stopping Eucalyptus services: walrus sc cloud done.
```
 
iSCSI まわりを停止

```
 killall -9 tgtadm
 killall -9 tgtd
```
 
lv を削除

```
 [root@eucalyptus ~]# lvdisplay
   --- Logical volume ---
   LV Name                /dev/vg-1iOYfw../lv-CdT28w..
 (snip)
   --- Logical volume ---
   LV Name                /dev/vg-3QD4nw../lv-V_kuAw..
 [root@eucalyptus ~]# lvremove /dev/vg-1iOYfw../lv-CdT28w..
   Logical volume "lv-CdT28w.." successfully removed
 [root@eucalyptus ~]# lvremove /dev/vg-3QD4nw../lv-V_kuAw..
   Logical volume "lv-V_kuAw.." successfully removed
```
 
vg を削除

```
 [root@eucalyptus ~]# vgdisplay
   --- Volume group ---
   VG Name               vg-1iOYfw..
 (snip)
   --- Volume group ---
   VG Name               vg-3QD4nw..
 [root@eucalyptus ~]# vgremove vg-1iOYfw..
   Volume group "vg-1iOYfw.." successfully removed
 [root@eucalyptus ~]# vgremove vg-3QD4nw..
   Volume group "vg-3QD4nw.." successfully removed
```
 
pv を削除

```
 [root@eucalyptus ~]# pvdisplay
   --- Physical volume ---
   PV Name               /dev/loop1
 (snip)
   --- Physical volume ---
   PV Name               /dev/loop0
 [root@eucalyptus ~]# pvremove /dev/loop1
   Labels on physical volume "/dev/loop1" successfully wiped
 [root@eucalyptus ~]# pvremove /dev/loop0
   Labels on physical volume "/dev/loop0" successfully wiped
```
 
ループバックマウントを解放

```
 [root@eucalyptus ~]# losetup -d /dev/loop0
 [root@eucalyptus ~]# losetup -d /dev/loop1
```

で、ここまでは簡単だし、上記のような手作業じゃなくサラっとワンライナーで消したりしてもいいんですが、こっから先が説明が面倒かつ間違えると環境破壊を起してしまうので、慎重に。

DB をバックアップ

```
 [root@eucalyptus ~]# cp -a /var/lib/eucalyptus/db /root/var.lib.eucalyptus.db.`date +'%Y%m%d'`
```
 
ボリュームについて記述してあるテーブルとトランザクションログのファイルを表示

```
 [root@eucalyptus ~]# cd /var/lib/eucalyptus/db/
 [root@eucalyptus db]# grep vol- -r ./ | cut -d':' -f1 | sort | uniq
 ./eucalyptus_images.log
 ./eucalyptus_images.script
 ./eucalyptus_storage.script
```

で、ここで表示されたテーブルとトランザクションログの中から前述のボリュームに該当するレコードとそれに付随する SQL 文を削除します (本当は CLC だけもう一度起動して、トランザクションログをテーブルデータにマージさせたほうが楽です)。

これらの作業が終ったら、CLC/Walrus/SC を起動してボリュームを作ってみると、環境が直ったか否かがわかります。

CreateVolume して…

```
 [root@eucalyptus db]# euca-create-volume -s 8 -z cluster0
 VOLUME  vol-592C0622    8       creating        2011-06-19T18:45:19.713Z
```
 
DescribeVolume して…

```
 [root@eucalyptus db]# euca-describe-volumes
 VOLUME  vol-592C0622     8              cluster0        available       2011-06-19T18:45:19.713Z
```
 
lv もちゃんとあるよね？
```
 [root@eucalyptus db]# ls /dev/|grep vg-|wc -l
 1
```

最初、本件は連載のネタになるかと考えたんですが、バッドノウハウすぎて、ちっとも「復旧」になってないので日記に書くことにしました。一応上記の手順では実体のボリュームは /var/lib/eucalyptus/volumes/ 配下に残っているので、ちょっと面倒な手順を要しますが、データを取り出すことは可能です。(それについては別の機会で触れ…ないかも)

