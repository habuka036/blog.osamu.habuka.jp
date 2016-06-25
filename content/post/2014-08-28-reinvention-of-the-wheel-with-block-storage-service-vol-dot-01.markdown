---
title: "ブロックストレージサービスで車輪の再開発(1)"
date: 2014-08-16 01:05:45 +0900
tags: ["binder"]
---

ストレージサービスについて
--------------------------

今日から何回か(もしくは何十回か)に分けてブロックストレージサービスネタを展開していこうと思います。

まず、ここで言うところの「ブロックストレージサービス」ですが、簡単に言うと OpenStack の Cinder ですね。

「だったら Cinder 使うなりイジるなりすればいいじゃん」っていう声が聴こえてきますが、宗教上の理由で OpenStack は扱えないので自分で再開発することにします。(じゃないとブログのネタにならない)

<!--more-->

NFS を Rest API で触りたい
--------------------------

えーと、何で NFS かと言うと、Cinder がまだ Cinder として Nova から分化する以前、「NFS も EBS みたいに切り出せたらいいのに」っていう話が OpenStack の開発とは全然関係ない、とある場所で出てまして、その要望に応えるために Rest API で NFS を払い出す仕組みを作ろうとしてました。で設計も終わって実装していたけれど色々な事情で途中で停止してしまい、気付いたら Cindar にそんな感じの機能が備わってました。

で、どう実装されているかとても気になったので、Cindar のコードを見ているうちに「あれ？これってブログのネタにならないかしら？」と思った次第です。

ちなみに今日見てた中で一番気になったのは以下の部分でした。

https://github.com/openstack/cinder/blob/master/cinder/volume/drivers/nfs.py#L222-L246

何が気になったのかと言うと、

* 恥しながら、truncate というコマンドとシステムコールを今頃知りました。 [^1]

* Eucalyptus でも言えることですが、実装コストを軽くするためとは言え、外部コマンドを叩く方法はあまり好きになれない。[^2]
* 何で run_as_root=True で叩いているんだろう？ cinder ユーザに権限を与えさえすれば別にスーパーユーザで叩くことも無いんじゃないかしら？何か理由があるんだろうか？

という点が気になりました。

truncate とスパースファイル
---------------------------

で、まず一点目の「truncate」ですが、

    $ truncate -s 1G truncate.img

なんていう風に実行するだけで、指定したサイズでスパースファイルを作ってくれます。これは便利。

というのも今まで僕はスパースファイルを作るときは

    $ dd if=/dev/zero of=dd.count0.img bs=1 count=0 seek=$(( 1 * 1024 * 1024 * 1024 ))

なんていう風に実行していました。[^3]

あ、話が脱線しますが、dd でスパースファイルを作るときに稀に以下のように実行している人が居るんですが、count=1 にする理由って何でなんでしょう？

    $ dd if=/dev/zero of=dd.count1.img bs=1 count=1 seek=1G

count=1 にすると以下に示すとおり、指定したサイズより bs * count 分だけ大きいファイルが作られてしまうんですが意図して指定しているんでしょうか…？

    $ ls -sl
    total 3076
    1024 -rw-r--r-- 1 root root 1073741824 Aug 16 12:02 dd.count0.img
    1028 -rw-r--r-- 1 root root 1073741825 Aug 16 12:02 dd.count1.img
    1024 -rw-r--r-- 1 root root 1073741824 Aug 16 11:13 truncate.img


えー、話が脱線しましたが、今度からはスパースファイルを作る際には truncate コマンドを使うようにします。ありがとう Cinder。

外部コマンド叩く話
------------------

で、二点目の「外部コマンドを叩く方法はあまり好きになれない」について。

上でもちょっぴり書きましたが、truncate はコマンドでもありますがシステムコールもあります。というか、truncate コマンドは ftruncate というシステムコールを叩いてます。

で、折角システムコールがあるのに何でコマンド叩くんだろう？と思い、僕は python 書けませんが何とかコマンドを叩いてみた場合とシステムコールを呼んでみた場合とで計測してみることにしました。ちなみに計測用に書いた拙いコードは以下です。
``` python
#!/usr/bin/python

import subprocess
from benchmarker import Benchmarker

def execute(*cmd):
    _PIPE = subprocess.PIPE
    p = subprocess.Popen(cmd, stdin=_PIPE, stdout=_PIPE, close_fds=True,
                         shell=False)
    (stdin, stdout, stderr) = (p.stdin, p.stdout, p.stderr)


def create_sparsed_file_by_command(path):
    execute('/usr/bin/truncate', '-s', '1G', path)

def create_sparsed_file_by_syscall(path):
    file = open(path, 'w')
    file.truncate(1 * 1024 * 1024 * 1024)
    file.close()

loop = 1000
with Benchmarker() as bm:
    with bm('command'):
        for i in xrange(loop):
            create_sparsed_file_by_command('/tmp/test/cmd_truncate%s.img' % i)

    with bm('syscall'):
        for i in xrange(loop):
            create_sparsed_file_by_syscall('/tmp/test/sys_truncate%s.img' % i)
```

で、実行結果が以下。

    $ python ~/create_sparsed_file.py 
    ## benchmarker:       release 3.0.1 (for python)
    ## python platform:   linux2 [GCC 4.8.2]
    ## python version:    2.7.6
    ## python executable: /usr/bin/python
    
    ##                                 user       sys     total      real
    command                          0.3300    0.4000    0.7300    2.2077
    syscall                          0.0100    0.0200    0.0300    0.0283
    
    ## Ranking                         real
    syscall                          0.0283 (100.0%) *************************
    command                          2.2077 (  1.3%) 
    
    ## Ratio Matrix                    real    [01]    [02]
    [01] syscall                     0.0283  100.0% 7790.9%
    [02] command                     2.2077    1.3%  100.0%


うーん、truncate に限った話であれば truncate コマンド叩くよりシステムコール呼んだほうが速いと言えると思うんですが…。まぁ、でも他の dd や qemu-img のほうと足並み揃えたほうがコードのメンテナンス性とか良いんでしょうかね。ちょっと俄にやってみただけの身だと「ほら、システムコールのほうがいいでしょ？」とは断言できないですねσ^^;

とりあえず今日はここまで。

----

[^1]: @enakai00 先生の講義で教えて頂いていたらごめんなさい…><'
[^2]: もちろん他にも理由あるとは思いますが…
[^3]: ちなみに「seek に1Gっていう風に単位つきで値を渡せたらいいのに…」とか思ってたんですが、実は指定できたんですね、これも知りませんでした…orz

