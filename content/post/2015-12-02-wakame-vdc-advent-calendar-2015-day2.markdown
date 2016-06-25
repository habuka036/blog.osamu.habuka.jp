---
title: "Wakame-vdc LiveDVD 16.1 を作る 2日目"
date: 2015-12-02 20:15:43 +0900
tags: ["Wakame-vdc","AdventCalendar"]
---

Wakame-vdc ユーザの皆さん、磯野家の皆さん、こんにちは。これはオレオレ WakameVdc / OpenVNet Advent Calendar 2015 の2日目のエントリです。

いやぁ、昨日1日目のエントリを投稿したあとで、半袖さんの [@hansode_retro](https://twitter.com/hansode_retro/status/671776916995641344) が「おめぇ、去年とネタが同じじゃねーかwww」って教えてくれたんですよ。

まぁ、去年どころか年がら年中「LiveDVD! LiveDVD! LiveDVD!」って叫んでるのでネタが被るのも当然ですよね？

<!--more-->

さて、今日は昨日作った LiveDVD を起動しようと思うのですが、今これ書いてる環境が携帯のテザリングでネットに繋いでいるせいで、さすがに 1GB 越えの ISO ファイルをローカルに落す気になれません。そんなことしたら明日からの僕の通勤時間は抜け殻タイムになってしまいます。

ということで、昨日作った ISO ファイルは地産地消に則り ConoHa で起動しようと思います。まずは ConoHa に ISO ファイルを登録…

    # conoha-iso download -i http://133.130.96.226/Wakame-vdc.LiveDVD.iso

すげぇ簡単。ConoHa ラブ。

    # conoha-iso list
    (中略)
    [Image3]
    Name:  Wakame-vdc.LiveDVD.iso
    Url:   http://133.130.96.226/Wakame-vdc.LiveDVD.iso
    Path:  /mnt/isos/repos/tenant_iso_data/0c1058ab2ffd46698450ef754f68ec7f/Wakame-vdc.LiveDVD.iso
    Ctime: Tue Dec  1 22:49:51 2015
    Size:  1111490560

さくっと登録完了。

続いてサーバを作成し、

![](/images/wakame-vdc.adventcalendar.2015.1202-01.png )

そしてシャットダウンもしくは強制終了させて以下のように ISO ファイルを挿入します。

    # conoha-iso insert
    [1] 133-130-116-239
    [2] 133-130-96-226
    Please select VPS no. [1-2]: 1
    
    [1] Wakame-vdc.LiveDVD.15.07.iso
    [2] Wakame-vdc.LiveDVD.16.1.iso
    [3] Wakame-vdc.LiveDVD.iso
    [4] coreos_production_iso_image.iso
    Please select ISO no. [1-4]: 3
    INFO[0007] ISO file was inserted and changed boot device. 

で、サーバを起動します。

無事に起動できると以下のようにデスクトップに自動でログインします。`

![](/images/wakame-vdc.adventcalendar.2015.1202-02.png )

で、WebUI でインスタンスを起動して ssh でインスタンスに接続…

![](/images/wakame-vdc.adventcalendar.2015.1202-03.png )

できませんでした…。

ということで明日からデバッグをやっていこうと思います。


