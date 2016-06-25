---
title: ブロックストレージサービスで車輪の再開発(2)
date: 2014-08-17 01:05:42 +0900
comments: true
tags: ["binder"]
---

外部コマンド叩く話のつづき
--------------------------

前回は truncate コマンドを叩くよりシステムコールの truncate を呼んだほうが実行時のコストが低いんじゃないかという話をしましたが、今回は Ciner の NFS ドライバーにある _create_regular_file() が dd コマンドを叩いていることについて考えたいと思います。コード的には以下の部分です。

<!--more-->

https://github.com/openstack/cinder/blob/master/cinder/volume/drivers/nfs.py#L227-L238

で、dd コマンドの代わりに何を呼べばいいかと coreutils の dd.c を読むと最終的にはシステムコール read と write を呼んでます。じゃぁ、python でも /dev/zero を read() で読み込んで write() で書き出せばいいのかと思いがちですが…

ちょっと待ってください、nfs.py では空のボリュームファイルを作るためにコマンド叩いているので /dev/zero から読み取るようになっていますが、システムコールを叩く方式を考えるなら何もわざわざ /dev/zero から読み込まなくても null (0x00) を write() で書けばいいということになると思います。

で、相変わらずの拙いコードですが、以下のようになります。
``` python
#!/usr/bin/python

import subprocess
from benchmarker import Benchmarker

def execute(*cmd):
    _PIPE = subprocess.PIPE
    p = subprocess.Popen(cmd, stdin=_PIPE, stdout=_PIPE, close_fds=True,
                         shell=False)
    (stdin, stdout, stderr) = (p.stdin, p.stdout, p.stderr)


def create_regular_file_by_command(path):

    Ki = 1024
    Mi = 1024 ** 2
    Gi = 1024 ** 3    

    block_size_mb = 1
    block_count = 1 * Gi / (block_size_mb * Mi)
    
    execute('/bin/dd', 'if=/dev/zero', 'of=%s' % path,
            'bs=%dM' % block_size_mb,
            'count=%d' % block_count,)

def create_regular_file_by_syscall_write(path):
    size = 1024 ** 3
    count = size / 4096
    null_4k = "\0" * 4096
    file = open(path, 'w')
    for i in xrange(count):
        file.write(null_4k)
    file.close()

loop = 5
with Benchmarker() as bm:
    with bm('syscall_write'):
        for i in xrange(loop):
            create_regular_file_by_syscall_write('/mnt/sda6/test/sys_write_createfile%s.img' % i)

    with bm('command'):
        for i in xrange(loop):
            create_regular_file_by_command('/mnt/sda6/test/cmd_createfile%s.img' % i)
```

そして実行結果は以下になりました。

    $ python ~/create_regular_file.py
    ## benchmarker:       release 3.0.1 (for python)
    ## python platform:   linux2 [GCC 4.8.2]
    ## python version:    2.7.6
    ## python executable: /usr/bin/python
    
    ##                                 user       sys     total      real
    syscall_write                    1.2300    7.3600    8.5900   20.8038
    command                          0.0000    0.0000    0.0000    0.0190
    
    ## Ranking                         real
    command                          0.0190 (100.0%) *************************
    syscall_write                   20.8038 (  0.1%)
    
    ## Ratio Matrix                    real      [01]      [02]
    [01] command                     0.0190    100.0% 109281.1%
    [02] syscall_write              20.8038      0.1%    100.0%
    
    1024+0 records in
    1024+0 records out
    1073741824 bytes (1.1 GB) copied, 45.8355 s, 23.4 MB/s
    1024+0 records in
    1024+0 records out
    1073741824 bytes (1.1 GB) copied, 46.7574 s, 23.0 MB/s
    1024+0 records in
    1024+0 records out
    1073741824 bytes (1.1 GB) copied, 47.6069 s, 22.6 MB/s
    1024+0 records in
    1024+0 records out
    1073741824 bytes (1.1 GB) copied, 47.6249 s, 22.5 MB/s
    1024+0 records in
    1024+0 records out
    1073741824 bytes (1.1 GB) copied, 49.6295 s, 21.6 MB/s


うーん、コマンドのほうは subprocess.Popen() なので並列に実行されていますが、write() のほうは特に何もしてないので直列に実行されててベンチマークでの比較ができない感じでした。

まー、ベンチマーク終了後の出力には 1 ファイルあたり 45〜50 秒ほどかかっている計算になりますので、仮に上記のように 5 ファイル作成するのであれば、dd コマンドより write() のほうが半分ぐらいの速度で終ると言ってもいいような気がします。

ちなみに write() だけじゃなく mmap() も試そうと思ったのですが、スキル不足のため断念しましたσ^^;

個人的には完全に直列はアレですが、せめて少し並列化させて write() を呼ぶほうが良いように思えます。

では今日はここまで。


