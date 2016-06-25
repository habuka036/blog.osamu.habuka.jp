---
title: "etcd-fs を試す"
date: 2014-08-14 13:13:20 +0900
tags: 
---

皆さん、素敵な etcd ライフを送ってますかー？

<!--more-->

僕はかなりの周回遅れで etcd を触り始めました。とは言え、本格的に利用しているわけではなく、まだツンツンしたぐらいです。

普段、私生活では何ら分散 KVS を必要する生活を送っていませんが、「これほど皆に持て囃されているなら、きっと僕にとっても何かメリットあるんじゃないの？」と思い、触り始めた次第です。

で、色々思うところがあって、いい感じに自分の環境で活用できないかと思って調べているうちに、[[etcd-fs|https://github.com/xetorthio/etcd-fs]] という逸品に出会ったので、今日はこれを使ってみた記録を。

ちなみに「etcd-fs って何ぞや？」って思ったそこのあなた、名前の通りです。「Use etcd as a filesystem」です。(説明からして既に手抜きでごめんなさい)

あ、あと、ビルドの仕方とかは割愛します。だって公式ドキュメントに書いてあるように make build を実行するだけですもの。ただし、注意点が一つあって、go のバージョンが 1.3 以上じゃないとビルドが失敗します。なので僕の環境は以下のようになってます。[^1]

    $ echo $GOROOT
    /home/osamu/local/go/
    
    $ which go
    /home/osamu/local/go//bin/go
    
    $ go version
    go version go1.3 linux/amd64


まず、etcd-fs を使うためには etcd を起動しておかないといけません。

    $ etcd -addr=127.0.0.1:4001 -peer-addr=127.0.0.1:7001


それと、etcd をマウントするポイントも用意しておかなければいけません。

    $ mkdir -p ~/mnt/etcd


そしたら、etcd-fs を叩いて etcd をマウントします。僕の環境では go なプロダクツを ~/local/src/ に配下に置くという変な構成にしちゃったので、etcd-fs も ~/local/src/etcd-fs/ に置いてあります。

    $ ~/local/src/etcd-fs/etcdfs ~/mnt/etcd/ http://127.0.0.1:4001
    /bin/fusermount: failed to open /etc/fuse.conf: Permission denied


/etc/fuse.conf を読み込む権限がないと↑のように Permission denied と言われますが、まぁ、etcd-fs 自体はそれでも動くので、スルーします。ちゃんと fuse.conf を読んで欲しい場合は適切に設定しましょう。

で、マウントできたか確認します。


    $ LANG=C ls -al ~/mnt/etcd/
    total 4
    drw-rw-rw- 0 osamu osamu    0 Jan  1  1970 .
    drwxrwxr-x 3 osamu osamu 4096 Aug 13 23:24 ..


はい、etcd に何も登録していない場合は上記のように空っぽな状態になります。

なので、早速 etcd に値を登録してみます。以下は rsyncd.conf というファイルを etcd に登録してみた的な感じです。

    $ curl -v -X PUT http://127.0.0.1:4001/v2/keys/rsyncd.conf -d value="
    > pid file = /var/run/rsyncd.pid
    > use chroot = yes
    > read only = yes
    >
    > [share]
    >     path = /var/share
    >     comment = share directory
    > "

ファイルとして認識されているか見てみます。

    $ LANG=C ls -al ~/mnt/etcd/
    total 4
    drw-rw-rw- 0 osamu osamu    0 Jan  1  1970 .
    drwxrwxr-x 3 osamu osamu 4096 Aug 13 23:24 ..
    -rw-rw-rw- 1 osamu osamu  126 Jan  1  1970 rsyncd.conf


タイムスタンプが変なのは仕方ないとして、ファイルもファイルサイズもちゃんとありそうですので、中身を見てみます。

    $ cat ~/mnt/etcd/rsyncd.conf
    
    pid file = /var/run/rsyncd.pid
    use chroot = yes
    read only = yes
    
    [share]
        path = /var/share
        comment = share directory

おぉ！、ちゃんと登録されていますね。じゃぁ、このファイル/KVSの末尾に値を追加します。

    $ echo "    exclude = /.swp" >> ~/mnt/etcd/rsyncd.conf

で、ファイルとして更新されているかを確認します。

    osamu@osamu-ThinkPad-X230 ~ $ LANG=C ls -al ~/mnt/etcd/
    total 4
    drw-rw-rw- 0 osamu osamu    0 Jan  1  1970 .
    drwxrwxr-x 3 osamu osamu 4096 Aug 13 23:24 ..
    -rw-rw-rw- 1 osamu osamu  146 Jan  1  1970 rsyncd.conf

さっきまで 126 bytes だったので、20 bytes 増えていますね。(改行コード含めて丁度 20 bytes 追加したので計算は合ってますね)

では、etcd 側から値を参照してみましょう。

    $ curl -X GET http://127.0.0.1:4001/v2/keys/rsyncd.conf
    {"action":"get","node":{"key":"/rsyncd.conf","value":"\npid file = /var/run/rsyncd.pid\nuse chroot = yes\nread only = yes\n\n[share]\n    path = /var/share\n    comment = share directory\n    exclude = /.swp\n","modifiedIndex":16,"createdIndex":16}}

おぉ！ echo で追加した値がちゃんと反映されています。

ちなみに、etcd-fs でマウントしていると、以下のように見えます。

    $ mount | grep /mnt/etcd
    pathfs.pathInode on /home/osamu/mnt/etcd type fuse.pathfs.pathInode (rw,nosuid,nodev,user=osamu)


で、これをアンマウントするには以下のように実行します。

    $ fusermount -u ~/mnt/etcd

アンマウントすると本来のディレクトリの情報に戻ります。

    $ LANG=C ls -al ~/mnt/
    total 12
    drwxrwxr-x  3 osamu osamu 4096 Aug 13 23:24 .
    drwxr-xr-x 66 osamu osamu 4096 Aug 14 00:26 ..
    drwxrwxr-x  2 osamu osamu 4096 Aug 13 23:24 etcd

以上、手抜きでしたが、etcd-fs を使ってみたメモです。機会があれば、これをイジって ini ファイルを Key/Value として扱えるモノを探してみます。(無ければ作るか…)

----

[^1]: 何を言いたいかというと、本家から Go 言語のバイナリをもってきて、それを使っていますよ…と


