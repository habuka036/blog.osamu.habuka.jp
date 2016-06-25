---
title: "eucalyptuscala を試してみる"
date: 2014-12-05 23:50:28 +0900
tags: ["Eucalyptus","AdventCalendar"]
---
これは [Eucalyptus Advent Calendar 2014](http://www.adventar.org/calendars/547) の5日目のエントリです。昨日は [@giraffeforestg](https://twitter.com/giraffeforestg) さんの「[Eucalyptus から AWS を管理してみる](http://giraffeforestg.blog.fc2.com/blog-entry-189.html) でした。

今日のエントリは3日目の [@tanacasino](https://twitter.com/tanacasino) さんのエントリ「[Eucalyptus meets Scala !? Scala meets Eucalytpus!?](http://tanacasino.hatenablog.com/entry/2014/12/03/003900)」を参考に eucalyptuscala を使ってみようと思います。

<!--more-->

    scala> euca.describeImages
    res2: com.amazonaws.services.ec2.model.DescribeImagesResult = {Images: [{ImageId: emi-dfe169b4,ImageLocation: ubuntu-12.04.4/ubuntu.raw.manifest.xml,State: available,OwnerId: 952671494305,Public: false,ProductCodes: [],Architecture: x86_64,ImageType: machine,Platform: ,Name: ubuntu-12.04.4,RootDeviceType: instance-store,RootDeviceName: /dev/sda1,BlockDeviceMappings: [],VirtualizationType: hvm,Tags: [],}]}

おぉ、イメージ ID が取得できますね。

次にインスタンスを起動してみようと思います…が、どうやたらいいのかさっぱりわかりません。AWScala ってリファレンスないんすか？(← ヘタレ)

とりあえず、引数なしで叩いてみます。

    scala> euca.runInstances()
    <console>:12: error: not enough arguments for method runInstances: (x$1: com.amazonaws.services.ec2.model.RunInstancesRequest)com.amazonaws.services.ec2.model.RunInstancesResult.
    Unspecified value parameter x$1.
                  euca.runInstances()
                                   ^

なんか RunInstancesRequest な型で渡せということでしょうかね？

    scala> euca.runInstances(com.amazonaws.services.ec2.model.RunInstancesRequest("emi-dfe169b4", 1, 1))
    <console>:12: error: object com.amazonaws.services.ec2.model.RunInstancesRequest is not a value
                  euca.runInstances(com.amazonaws.services.ec2.model.RunInstancesRequest("emi-dfe169b4", 1, 1))
                                                                     ^

ん？値じゃないって？じゃぁ…こう？

    scala> euca.runInstances(new com.amazonaws.services.ec2.model.RunInstancesRequest("emi-dfe169b4", 1, 1))
    res11: com.amazonaws.services.ec2.model.RunInstancesResult = {Reservation: {ReservationId: r-dadc5b05,OwnerId: 952671494305,Groups: [{GroupName: default,GroupId: sg-4f9f263b}],GroupNames: [default],Instances: [{InstanceId: i-7933764d,ImageId: emi-dfe169b4,State: {Code: 0,Name: pending},PrivateDnsName: 192.168.122.242,PublicDnsName: 192.168.122.242,StateTransitionReason: NORMAL:  -- [],AmiLaunchIndex: 0,ProductCodes: [],InstanceType: m1.small,LaunchTime: Sat Dec 06 17:34:57 JST 2014,Placement: {AvailabilityZone: default,},Monitoring: {State: disabled},PrivateIpAddress: 192.168.122.242,PublicIpAddress: 192.168.122.242,Architecture: x86_64,RootDeviceType: instance-store,RootDeviceName: /dev/sda1,BlockDeviceMappings: [],VirtualizationType: hvm,ClientToken: 860b57f0-43f6-4099-b2a1-5749d63336...

おぉっ！

![](/images/eucalyptus.adventcalendar.2014.1205-01.png )

おぉぉ！！ってことで無事起動できました。やったー。

ということで、途中から次の日になってしまいましたが、euca.runInstances() にどうやって値を渡せばよいのかを悩んでいるうちに日付が変わってしまいましたwww

ちなみに AWScala の runInstances() のことを説明しているドキュメントとかリファレンスがどこにあるのかさっぱりわからないので、とりあえず[コード](https://github.com/seratch/AWScala/blob/develop/src/main/scala/awscala/ec2/Requests.scala#L6-L7)を見てみると以下のように書いてあります。

```scala
package awscala.ec2

import scala.collection.JavaConverters._
import com.amazonaws.services.{ ec2 => aws }

case class RunInstancesRequest(imageId: String, min: Int = 1, max: Int = 1)
    extends aws.model.RunInstancesRequest(imageId, min, max) {

  // TODO set...
}
```

さて、インスタンスタイプとかユーザデータとかセキュリティグループとかゾーンとかはどうやって渡せばいいんだろう？これは ec2.model.RunInstancesRequest じゃなく aws.model.RunInstancesRequest を見ないといけないのでしょうけれど、僕は Scala を全く知らないので諦めることにします。:)

----
ということで、6日目になってしまった5日目のエントリでした。6日目のエントリは @shida1234 さんです。お楽しみに。

