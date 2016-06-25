+++
title = "SkyDNSのバックエンドにDynamoDBを使ってみる"
date = "2015-12-19"
tags = ["AWS","AdventAalendar"]
+++

これは [AWS Advent Calendar 2015](http://qiita.com/advent-calendar/2015/aws) の 18 日目のエントリです。

今日は SkyDNS のバックエンドに etcd でなく DyamoDBを使ってみようという話です。

<!--more-->

皆さん大好き SkyDNS ですが、これを AWS の EC2 上で構築したときに必ず以下の課題に遭遇すると思います。

* 冗長性を担保するために複数台構成にしよう
* 耐故障性を上げためにマルチ AZ 構成にしよう
* あれ？ AZ 1つが丸々落ちても過半数を割らないようにするためには奇数にして…
* うーん、マジョリティ側の AZ が落ちたらアウトだ…

そーすると耐故障性を上げるどころの話じゃなくなってしまい、楽をするために導入しようとした SkyDNS が足枷になったりします。

でも思い出してみてください、AWS には DynamoDB という素敵なサービスがあります。SkyDNS にとっての etcd を単なるデータストアと割り切って見た場合、etcd の代用として DynamoDB が使えると思いませんか？いや、むしろ「代用」なんかじゃなく、SkyDNS を AWS で動かす場合には無くてはならない存在だと言えます。

じゃー SkyDNS は DynamoDB をデータストアとして使うことできるのか？と言うと、残念ながらそのままでは SkyDNS は DynamoDB をデータストアとして使うことができません。

しかし、我らが SkyDNS、ちゃんと [etcd の部分を pluggable](https://github.com/skynetservices/skydns/tree/master/backends) にしてあります。

ということで、etcd の代わりに DynamoDB を利用するコードを backend/ 配下に書いてみます。

が、すみませんがタイムアップでして、続きはまたこのエントリを更新します。

