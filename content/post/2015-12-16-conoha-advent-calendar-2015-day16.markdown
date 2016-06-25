---
title: "ConoHaに作るWakame-vdc LiveDVD CI 環境"
date: 2015-12-16 06:15:38 +0900
tags: ["ConoHa","Wakame-vdc","AdventCalendar"]
---

ConoHa ユーザの皆さま & ConoHa タンこんばんは。これは ConoHa Advent Calendar 2015 の 16 日目のエントリです。16 日目のエントリなのに 16 日を過ぎてからの掲載でごめんなさい。

今日のエントリでは何をやるかというと、Wakame-vdc LiveDVD のビルドとテストを自動化したいという欲求が以前からあり、折角なので Advent Calendar のネタにしようという話です。

(が、思いっきり失敗談です…orz)

<!--more-->

で、どんなものを作ろうとしているかをまず先に説明します。ざっと粗く書くと以下のような感じです。

![2015.1216-00](/images/ConoHa.adventcalendar.2015.1216-00.png)

まずリポジトリは github 上にあるので OK で、LiveDVD Build Macine も既に ConoHa 上に存在し、ConoHa VM は事前に準備するものがないため、残る Drone の準備だけとなります。

drone.io のサービスを利用する方法とローカルに Drone を構築する方法がありますが、今回は ConoHa で起動したサーバ上で動かしてみます。

Drone のドキュメントを読むと手軽に環境を作る方法として Drone が配布している docker のイメージを利用する方法がありますので、まずは ConoHa で起動したサーバに docker をインストールするところから始めます。

    # yum install epel-release.noarch
    # yum install docker-io
    # service docker start

相変わらずの簡単さですね。続いて Drone のイメージを準備します。

    # docker pull drone/drone:0.4

で続いて docker run をするんですが、その前に Drone の設定ファイルを準備します。公式ドキュメントには「dronerc ファイルを用意しろ」って書いてあるだけで具体的な設定例がドキュメント中には見当たらないため、ソースコード中に存在していた [Debian 用の設定サンプル](https://github.com/drone/drone/blob/master/contrib/debian/drone/etc/drone/dronerc) を参考に以下のような設定にしました。一部、github の固有情報はボヤかしてあります f^^;

    #!/bin/bash
    
    # server configuration
    
    #SERVER_ADDR=":80"
    #SERVER_CERT=""
    #SERVER_KEY=""
    
    # database configuration
    
    DATABASE_DRIVER="sqlite3"
    DATABASE_CONFIG="/var/lib/drone/drone.sqlite"
    
    # remote configuration
    
    CLIENT="xxxxxxxxxxxxxxxxxxxx" # oauth2 client. REQUIRED
    SECRET="yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy" # oauth2 secret. REQUIRED
    
    REMOTE_DRIVER="github"
    REMOTE_CONFIG="https://github.com?client_id=$CLIENT&client_secret=$SECRET"
    
    # docker configuration
    
    DOCKER_HOST="unix:///var/run/docker.sock"
    #DOCKER_CERT=""
    #DOCKER_KEY=""
    #DOCKER_CA=""
    
    # plugin configuration
    
    PLUGIN_FILTER="plugins/*"

で、この内容を /etc/drone/dronerc としたら以下のように docker run します。

    docker run \
    --volume /var/lib/drone:/var/lib/drone \
    --volume /var/run/docker.sock:/var/run/docker.sock \
    --env-file /etc/drone/dronerc \
    --restart=always \
    --publish=80:8000 \
    --detach=true \
    --name=drone \
    drone/drone:0.4

で、無事コンテナが起動したら、ブラウザでアクセスします。

![](/images/ConoHa.adventcalendar.2015.1216-01.png)

はい、とてもシンプルな画面が表示されました。続いて LOGIN をクリックすると github のアプリケーション認証画面に飛ばされ…

![](/images/ConoHa.adventcalendar.2015.1216-02.png )

あれ？404だ…何故だ…？と思って URL をよく見ると…

    https://github.com/login/oauth/authorize?client_id=%24CLIENT&redirect_uri=.....(略)

あれ？ client_id が dronerc に設定した値じゃなく変数名がそのまま入ってる…

しょうがないので URL の %24CLIENT の部分に github から払出された値を入れて手で URL を叩いてみると…

![](/images/ConoHa.adventcalendar.2015.1216-03.png )

おぉ、なんとか表示されました。なので [Authorize application] をクリックすると…

![](/images/ConoHa.adventcalendar.2015.1216-04.png )

パスワード入力画面に遷移するので、パスワードを入力すると…

![](/images/ConoHa.adventcalendar.2015.1216-05.png )

う…ん、何故でしょう？うまくいきません。

で調べてみたところ、$CLIENT が設定値に置き換えられずにそのまま渡されるのはどうやら [0.4 のバグ](https://github.com/drone/drone/pull/1369) っぽいです。

っていうことで、仕方ないので docker イメージは諦めて自分でビルドしてみることにします。

まず drone のコードを入手します。

    # export PATH=/opt/go/bin/:$PATH
    # export GOROOT=/opt/go
    # export GOPATH=/usr/local/go
    # cd /usr/local/go/src/github.com/
    # mkdir -p drone
    # cd drone/
    # git clone https://github.com/drone/drone.git
    # cd drone/

ここまでやっておきながらアレですが、素直に go get github.com/drone/drone すりゃぁよかったんじゃないかと…。気にせずに次続けます。

    # make deps gen

エラーになり、PuerkitoBio と gopkg.in/yaml.v2 がゲットできないと言ってコケます。なのでとりあえず手動で入れます。

    # cd /usr/local/go/src/github.com/PuerkitoBio/
    # git clone https://github.com/PuerkitoBio/purell.git
    # go get gopkg.in/yaml.v2

gopkg.in/yaml.v2 が返ってこなくなります。で、調べたところ[どうやら git のバージョンが古いせい](http://qiita.com/wappy100/items/05364b5c095e4bdc860b)らしいです。なので、git を新しくします。

    # yum remove git
    # yum install -y curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
    # cd /usr/local/src/
    # wget https://www.kernel.org/pub/software/scm/git/git-2.6.2.tar.gz
    # tar -xzvf git-2.6.2.tar.gz 
    # cd git-2.6.2
    # make prefix=/usr/local all
    # make prefix=/usr/local install

で再度 yaml.v2 をゲットします。

    # go get gopkg.in/yaml.v2

続いて依存するのもをインストールします。

    # cd /usr/local/go/src/github.com/drone/drone/
    # ./contrib/setup-sqlite.sh 
    # ./contrib/setup-sassc.sh 

うーん、GCC のバージョンが 4.6 以上じゃないと駄目と怒られます… CentOS 6 で GCC を 4.6 以上にするには…[ここ](https://github.com/FezVrasta/ark-server-tools/wiki/Install-of-required-versions-of-glibc-and-gcc-on-RHEL-CentOS)を読むと方法が書いてありますが、16日はタイムアップとなりました…。

中途半端な失敗談で申し訳ないですが、進展があったら継続してこのページを更新することにします。


