---
title: "Eucalyptus4 のダッシュボードの見た目を変えてみる"
date: 2014-12-09 23:05:42 +0900
tags: ["Eucalyptus","AdventCalendar"]
---
これは [Eucalyptus Advent Calendar 2014](http://www.adventar.org/calendars/547) の9日目のエントリです。昨日は私のエントリ「[Eucalyptus 環境を再起動後にインスタンスが起動しなくなったら](http://blog.osamu.habuka.jp/blog/2014/12/08/eucalyptus-advent-calendar-2014-8th/)」でした。

今日のエントリは Eucalyptus Advent Calendar 2012 JP の13日目の私のエントリ「[Eucalyptus Advent Calendar 2012 JP 十三日目](http://blog.osamu.habuka.jp/blog/2012/12/13/eucalyptus-advent-calendar-2012-jp-13th/)」を思い出したので、Eucalyptus 4 のダッシュボードの見た目も変更してみようと思います。

<!--more-->

といってもあまり難しいことありません。CentOS 6.5 の場合、ダッシュボードである eucaconsole の実体は /usr/lib/python2.6/site-packages/eucaconsole/ 配下にありまして、たとえば css をイジる場合は /usr/lib/python2.6/site-packages/eucaconsole/static/css/ 配下にあるファイルをイジります。

ということで、ちょちょいと変更した結果、

![](/images/eucalyptus.adventcalendar.2014.1209-01.png )

が

![](/images/eucalyptus.adventcalendar.2014.1209-02.png )

って感じになりました。おしまい。


