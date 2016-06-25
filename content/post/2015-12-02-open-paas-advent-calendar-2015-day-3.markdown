---
title: "empire で PaaS デビューしてみたい！"
date: 2015-12-02 23:21:20 +0900
tags: ["PaaS","AdventCalendar","empire"]
---

PaaS 大好きっ娘の皆さんこんばんわ。IaaS しか触ったことのないおちぶれ技術者の羽深です。

今日はそんな PaaS 素人が、empire っていう名前の PaaS 基盤ソフトウェアをインストールしてみて触ってみようという無謀なことを [Open PaaS Advent Calendar 2015](http://www.adventar.org/calendars/803) の 12/3 分のエントリでやらかそうと思います。

<!--more-->

で、「empire って何よ？」っていうかたも居ると思います。README には

> Empire is a control layer on top of Amazon EC2 Container Service (ECS) that provides a Heroku like workflow.

って書いてあります。うん、どうしよう、僕 Heroku 触ったことないや…。でもまぁ、[quickstart install ガイド](http://empire.readthedocs.org/en/latest/quickstart_installing/) でも見ながらやってみようかと思います。

ちなみに日本語情報ないかなぁ？って検索してみたら、半年も前に、とっくにクラスメソッドさんが[ブログ](http://dev.classmethod.jp/cloud/ecs-with-empire/) に書いてました。クラスメソッドさんすげぇよ。っていうかあれ？このクラスメソッドさんのブログを見たら僕の出る幕なんかないんじゃないかしら？ってジワジワと思えてきました。

とは言っても今更他の PaaS を探して触るのも僕にとっては辛いため、恥も外聞もかなぐり捨てて、クラスメソッドさんのブログと同じことをやってみようと思います。あぁ、我乍ら酷い…。

awscli 入れたり環境を整えたりして、一番最初の手順を実行します。

    $ ./bin/bootstrap 
    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    !! This script produces an insecure demo or dev environment and is   !!
    !! not meant to be ran in production. As well it is exposed to the   !!
    !! internet and as such should not be left running without further   !!
    !! locking down of the security groups.                              !!
    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    Do you want to continue? (y/N)y
    ./bin/bootstrap: line 48: /usr/sbin/md5sum: No such file or directory


ふぅむ。which md5sum によると当方の環境は /usr/sbin/md5sum `ではなく /usr/bin/md5sum っぽいので、スクリプトを修正しま…あれ？

    # This variable will be used as the cloudformation stack name and the associated
    # ECS cluster.
    SUFFIX=""
    [ -x /sbin/md5 ] && SUFFIX=$(date | /sbin/md5 | head -c 8)
    [ -x /usr/bin/md5sum ] && SUFFIX=$(date | /usr/sbin/md5sum | head -c 8)
    if [ -z ${SUFFIX} ]
    then

うん、そりゃぁ駄目だね、/usr/bin/md5sum の存在を確認しておきながら、実行側はタイポだ…

で、パスを修正して再度実行します。

    $ ./bin/bootstrap 
    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    !! This script produces an insecure demo or dev environment and is   !!
    !! not meant to be ran in production. As well it is exposed to the   !!
    !! internet and as such should not be left running without further   !!
    !! locking down of the security groups.                              !!
    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    Do you want to continue? (y/N)y
    AWS SSH KeyName: empire-20151203
    Do you have a docker account & want to use it for private repo access? [y/N] N
    ==> Launching empire in AZs: us-east-1a us-east-1b, Cloudformation Stack empire-28cc3b96
    ==> Waiting for stack to complete
    ==> Status: CREATE_IN_PROGRESS
    ==> Stack empire-28cc3b96 complete.
    ==> Now run the following commands - when asked for a username, enter 'fake'. The password is blank:
    $ export EMPIRE_API_URL=http://empire-28-LoadBala-XXXXXXXXXXXX-XXXXXXXXXX.us-east-1.elb.amazonaws.com/
    $ emp login

って、綺麗に成功したように書きますが、実はここまでに以下の失敗をしています。

* docs/cloudformation.json 内で定義されている AMI ID を最新の ami-5449393e に変更してなくてコケる
* IAM ユーザの権限が限定的すぎて CloudFormation で権限不足でコケる
* 作成した鍵が東京リージョンだったため、指定した鍵が存在しなくてコケる

で、次に出力にあるように emp コマンドでログインします。

    $ ./emp login
    Enter email: fake
    Enter password: 
    Logged in.
    $ ./emp deploy remind101/acme-inc:latest
    latest: Pulling from remind101/acme-inc
    8dd8107abd2e: Pull complete 
    6f71594c9578: Pull complete 
    164cae5cf718: Pull complete 
    0cfc281bff60: Pull complete 
    79b9dd3dde72: Pull complete 
    2f07490e4ceb: Pull complete 
    563123d53314: Pull complete 
    Digest: sha256:7949da90576b184e6358af8b041ca4ae96cc5f45ddf9d1b546861c6c7f77bde9
    Status: Downloaded newer image for remind101/acme-inc:latest
    Status: Created new release v1 for acme-inc
    $ ./emp apps
    acme-inc      Dec  3 05:09
    $ ./emp ps -a acme-inc
    v1.web.be294837-0398-47ae-820b-417f57993866  1X  RUNNING  50s  "acme-inc server"

ってな感じで上手く出来たようなんですが、この作成された acme-inc にどうやってアクセスすりゃいいんだろう…？別途手動で Public Hosted Zone を Route53 に作成してあげないと駄目なんですかね…ちょっと調査不足 & 時間不足でした。

機会があれば引き続き試したいところですが、本日のところはタイムアップです。

