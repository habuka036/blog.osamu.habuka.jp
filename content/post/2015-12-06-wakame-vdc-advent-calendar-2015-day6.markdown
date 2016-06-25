---
title: "ホストめう"
date: 2015-12-06 06:27:27 +0900
tags: ["Wakame-vdc","AdventCalendar"]
---

Wakame-vdc ユーザの皆さん、磯野家の皆さん、こんにちは。これはオレオレ WakameVdc / OpenVNet Advent Calendar 2015 の6日目のエントリです。

順調に「日々更新ができなくなってきている」状態ですが、細かいことは気にせずマイペースで行きたいと思います。

今日も小粒なネタで、「Wakame-vdc LiveDVD を起動したときにホストにグローバル IP が付いてると Xfce4 のログイン画面で確認のダイアログが出てログインが止まるんですけどぉ？」っていう話です。

<!--more-->

何を言っているかというと、これです。

![](/images/wakame-vdc.adventcalendar.2015.1206-01.png )

まー、たいしたこと言ってるわけじゃなく確認事項なので普段なら「Continue anyway をクリックしてね」で済むんですが、先々 CI 環境を作ってテストを回そうとしたときに邪魔になりますよね、間違いなく。(もちろん、LiveDVD の CI で GUI 側をテストしなきゃいいっていう話もありますが、できれば GUI 側もテストしたいじゃないですか？)

ってことで、これに対処するために LiveDVD の /etc/rc.local に以下を追記しました。(実際は以下を実行するスクリプトを書いて、/etc/rc.local で起動する形に)

    hostname -s | awk 'BEGIN {FS="-";OFS="."} {print $2,$3,$4,$5" "$0}' >> /etc/hosts

もっと良い方法があるとは思うんですが、まぁこれでやってみます。(まぁ、当然ながらホスト名の形式が host-GL-OB-AL-IP な場合にしか通用しないので、このままだと酷いですが、とりあえず今だけ)

![](/images/wakame-vdc.adventcalendar.2015.1206-02.png )

ん？「... localhost」？あぁ、これはホスト名が設定される前に /etc/rc.local が叩かれているからか…。

ということで、/etc/rc.local の最後のほうで呼ぶようにしたらちゃんと上手くいきました。

明日は LiveDVD じゃない普通の Wakame-vdc を lxc で動かしてみる話です。




