---
title: "Wakame-VDC LiveDVD 軽量版"
date: 2014-12-25 20:41:24 +0900
tags: ["Wakame-vdc","AdventCalendar"]
---
Wakame ユーザの皆さん、磯野家の皆さん、こんにちは。これは Wakame-VDC / OpenVNet Advent Calendar 2014 の25日目のエントリです。

先日、Wakame-VDC LiveDVD with X 版をリリースしましたが、

<!--more-->

* 最低でもメモリが 6GB 必要
* デフォで X が有効になっているので、もしかしたら環境によっては画面が見れない

という課題がありましたので、この2点の対策を行なった軽量な Wakame-VDC LiveDVD として、X を除いたものを作りました。

ただしそれだけだと芸がないので、起動時のスプラッシュ画面をちょっとだけ変更してみました。この「ちょっとした変更」のために二日費しました…すみません。

画像を勝手に使いました。もし適切な絵がありましたらください。 > 磯野家の皆様

![](/images/wakame-vdc.adventcalendar.2014.1225-01.png )

プログレスバーの色も変えました。この色を変更するために plymouth のソース RPM を取得して色変更のためにパッチ書いて RPM をビルドしなおして…ちょっと plymouth のことが嫌いになりそうでした…。

![](/images/wakame-vdc.adventcalendar.2014.1225-02.png )

あ、ちなみに「Wakame-VDC LiveDVD 14.12」っていう表記になっていますが、あまりバージョン番号に意味はありません。

肝心の最低メモリ容量ですが、4GB あれば動くようになってます。

* [Wakame-VDC LiveDVD without X 版 ISO ファイルのダウンロード](https://s3-ap-northeast-1.amazonaws.com/iso-no-wakwame/Wakame-VDC.LiveDVD.iso)
* [Wakame-VDC LiveDVD without X 版の使い方](https://github.com/axsh/iso-no-wakame/wiki/LiveDVD%E3%81%AE%E4%BD%BF%E3%81%84%E6%96%B9)

さて、そろそろ自動で ISO ファイルが生成される仕組みを作らないとなぁ。

ということで、最終日のエントリなのに内容が薄くて申し訳ありませんが、メリークリスマス！

