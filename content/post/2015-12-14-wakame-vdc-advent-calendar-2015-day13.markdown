---
title: "花咲け！Packer〜ん"
date: 2015-12-14 03:19:27 +0900
tags: ["Wakame-vdc","AdventCalendar"]
---

Wakame-vdc ユーザの皆さん、磯野家の皆さん、こんにちは。これはオレオレ WakameVdc / OpenVNet Advent Calendar 2015 の13日目のエントリです。

Advent Calendar も半分を過ぎましたが、順調に一日遅れの更新から脱却できない今日この頃、皆様如何おすごしですか？

12日目のエントリでコンテナ内に /dev/tty が無くて上手く ssh 接続できないのを何とか解決してみようと試みました。本日はちょっと視点を変えて「先日公開されたばかりの Wakame-vdc の packer プラグインを使ってみたらどうよ？」をやってみたいと思います。(いつもどおり成功するか否かは不明です)

<!--more-->

で、結果だけ書くとあっという間に終わってしまうので、折角なので一から書いてみます。

まずは packer をインストールします。

    # mkdir -p /opt/packer
    # cd /opt/packer
    # wget https://releases.hashicorp.com/packer/0.8.6/packer_0.8.6_linux_amd64.zip
    # unzip packer_0.8.6_linux_amd64.zip

packer のプラグイン用のディレクトリを掘ります。

    # mkdir -p /root/.packer/plugins

プラグインをビルドするために go をインストールします。Wakame-vdc のために golang をインストールする時代が来るとは…面白い。

    # cd /opt
    # wget https://storage.googleapis.com/golang/go1.5.2.linux-amd64.tar.gz
    # tar -xzvf go1.5.2.linux-amd64.tar.gz
    # export PATH=/opt/go/bin/:$PATH
    # export GOROOT=/opt/go
    # export GOPATH=/usr/local/go

Packer の Wakame-vdc プラグインをビルドします。

    # mkdir -p /usr/local/go/src/github.com/axsh
    # cd /usr/local/go/src/github.com/axsh
    # git clone https://github.com/axsh/wakame-vdc.git
    # cd wakame-vdc/
    # git checkout packer-builder
    # cd client/packer-builder-wakamevdc/
    # go get
    # go build

ビルドが成功すると以下のようにプラグインのバイナリが生成されます。

    # ls -al
    合計 16128
    drwxr-xr-x 3 root root     4096 12月 14 03:40 2015 .
    drwxr-xr-x 7 root root     4096 12月 14 03:27 2015 ..
    -rw-r--r-- 1 root root     1533 12月 14 03:27 2015 README.md
    -rw-r--r-- 1 root root      288 12月 14 03:27 2015 main.go
    -rwxr-xr-x 1 root root 16487528 12月 14 03:40 2015 packer-builder-wakamevdc
    -rw-r--r-- 1 root root      535 12月 14 03:27 2015 test.json
    drwxr-xr-x 2 root root     4096 12月 14 03:27 2015 wakamevdc

このバイナリをプラグイン用ディレクトリに置きます。(っていうか、packer と同じ場所に置いちゃ駄目なのかしら？)

    # mv packer-builder-wakamevdc /root/.packer/plugins/

次にイメージを生成するための json ファイルを以下のように書きました。

    # cat lxc-image.json
    {
      "builders": [
        {
          "type": "wakamevdc",
          "api_endpoint": "http://133.130.109.122:9001/api/12.03/",
          "image_id": "wmi-centos1d64",
          "hypervisor": "lxc",
          "cpu_cores": 1,
          "memory_size": 1024,
          "network_id": "nw-demo1",
          "ssh_username": "root",
          "ssh_timeout": "60m"
        }
      ],
      "provisioners": [
        {
          "type": "shell",
          "inline": [
            "echo RUN Provisioner.",
            "echo \"nameserver 8.8.8.8\" >> /etc/resolve.conf",
            "yum install -y httpd"
          ]
        }
      ]
    }

じゃーイメージをビルドしてみましょう。

    # packer build lxc-image.json 

うーん、実行したのはいいがコマンドが全然返ってきません。

    # ls -al
    合計 28
    drwxr-xr-x 3 root root 4096 12月 14 04:05 2015 .
    drwxr-xr-x 7 root root 4096 12月 14 03:27 2015 ..
    -rw-r--r-- 1 root root 1533 12月 14 03:27 2015 README.md
    -rw-r--r-- 1 root root    0 12月 14 04:05 2015 build.hwm
    -rw-r--r-- 1 root root    0 12月 14 04:05 2015 build.pwd
    -rw-r--r-- 1 root root    0 12月 14 04:05 2015 build.pwi
    -rw-r--r-- 1 root root  538 12月 14 04:04 2015 lxc-image.json
    -rw-r--r-- 1 root root  288 12月 14 03:27 2015 main.go
    -rw-r--r-- 1 root root  535 12月 14 03:27 2015 test.json
    drwxr-xr-x 2 root root 4096 12月 14 03:27 2015 wakamevdc

実行したディレクトリに build.hwm、build.pwd、build.pwi という3つのファイルが出きてます…何だこれ？
packer を初めて叩くのがバレバレですね…f^^;

ふと「あれ？ packer を /opt/packer に配置したけど PATH を通した覚えがないのに何でコマンドが実行できるんだ？と思い確認しますと…

    # which packer
    /usr/sbin/packer
    # ls -al /usr/sbin/packer
    lrwxrwxrwx. 1 root root 15  8月 21 10:41 2015 /usr/sbin/packer -> cracklib-packer
    # rpm -qf /usr/sbin/packer
    cracklib-dicts-2.8.16-4.el6.x86_64

あれ？他人の空似だった…。

気を取り直してもう一度実行します。

    # /opt/packer/packer build lxc-image.json 
    Failed to initialize build 'wakamevdc': builder type not found: wakamevdc
    wakamevdc output will be in this color.
    
    
    ==> Builds finished but no artifacts were created.

あれ？ wakamevdc が builder type として認識されていない…。面倒なのでプラグインを packer のディレクトリに移動して再度実行してみます。

    # mv /root/.packer/plugins/packer-builder-wakamevdc /opt/packer/
    # /opt/packer/packer build lxc-image.json 
    wakamevdc output will be in this color.
    
    ==> wakamevdc: Creating SSH Key...
    ==> wakamevdc: New SSH Key: ssh-ct81jnos
    ==> wakamevdc: Creating Security Group...
    ==> wakamevdc: New Security Group: sg-o0q0dfjn
    ==> wakamevdc: Creating instance...
    ==> wakamevdc: Error creating instance: API Error: Type: Dcmgr::Endpoints::Errors::InvalidImageID, Code: 137, Message: Dcmgr::Endpoints::Errors::InvalidImageID, (HTTP 400)
    ==> wakamevdc: Deleting temporary ssh key...
    ==> wakamevdc: Deleting temporary security group...
    Build 'wakamevdc' errored: Error creating instance: API Error: Type: Dcmgr::Endpoints::Errors::InvalidImageID, Code: 137, Message: Dcmgr::Endpoints::Errors::InvalidImageID, (HTTP 400)
    
    ==> Some builds didn't complete successfully and had errors:
    --> wakamevdc: Error creating instance: API Error: Type: Dcmgr::Endpoints::Errors::InvalidImageID, Code: 137, Message: Dcmgr::Endpoints::Errors::InvalidImageID, (HTTP 400)
    
    ==> Builds finished but no artifacts were created.

あぁ…っ、ひどい勘違いをしていたことに気付きました…。もしかしてこの packer プラグインはベースとなるイメージから作るものなのかな？って。ちょっと Wakame-vdc の中の人に聞いてみることにします。

一応 image_id に Wakame-vdc LiveDVD に登録されている image_id を指定して再実行してみます。(が、12日目のエントリに書いたように ssh 接続できないから無理だと思いますが…)

    # /opt/packer/packer build lxc-image.json 
    wakamevdc output will be in this color.
    
    ==> wakamevdc: Creating SSH Key...
    ==> wakamevdc: New SSH Key: ssh-uvqidt9o
    ==> wakamevdc: Creating Security Group...
    ==> wakamevdc: New Security Group: sg-2z9mjrrj
    ==> wakamevdc: Creating instance...
    ==> wakamevdc: Waiting for instance to become running......
    ==> wakamevdc: Waiting for SSH to become available...

はい、このまま数十分待っても返ってきません。やっぱりログインできるマシンイメージを用意することが最重要なので、これも Wakame-vdc の中の人に質問することにします。

今日はこれまで。

