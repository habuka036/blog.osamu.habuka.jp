---
title: "ブロックストレージサービスで車輪の再開発(4)"
date: 2014-08-19 01:05:38 +0900
comments: true
tags: ["binder"]
---

言語
----

今更ですが「車輪の再開発」じゃなくて「車輪の再発明」ですよね…(// ▽ //)

<!--more-->

当初は勉強をかねて Elixir で書こうと思っていたんですが、会社の同僚(若者)が「Go どうよ？」「これからツールとかユーティリティ書くなら Go 楽じゃね？」「あー、でもデーモンとかは Go 辛くね？」と、満足にプログラミング一つできない老害な僕に囁いてきたので、なんか時代に乗り遅れてるとか哀れんだ目で見られないように、ちょっと今回は Go で行こうかと思います。

そして折角 Go を使うので、設定やデータは etcd に放り込むことにします。あと WebAPI を実装するために martini を使います。

WebAPI
------

「そろそろ実装しながら」とか言いながらまだ一行もコード書いてなくてすみません。

とりあえず API は以下のようにします。(初めて書く Go とはいえ、もうちょっとそれなりなものを書けよと自分に言いたい…orz) 

``` go
package main

import (
    "github.com/go-martini/martini"
)

func main() {
   m := martini.Classic()
   m.Group("/volume", func(r martini.Router) {
      r.Get("/(:id)*", list_volume)
      r.Post("/", create_volume)
      r.Delete("/:id", delete_volume)
   })
   m.Run()
}
func list_volume(params martini.Params) {
   println("list_volume() invoked")
   if "" != params["id"] {
      println("list_volume(): id =", params["id"])
   }
}

func create_volume(params martini.Params) {
   println("create_volume() invoked")

}

func delete_volume(params martini.Params) {
   println("delete_volume() invoked")
}
```

で、とりあえずこれを実行すると、127.0.0.1:3000 で起動するので、以下のように叩くとログに吐かれます

    curl http://127.0.0.1:3000/volume/
    curl http://127.0.0.1:3000/volume/1
    curl -X POST -H "Content-type: application/json" -d '{"volume_size":1,"volume_name":"volume1"}' http://127.0.0.1:3000/volume/
    curl -X DELETE http://127.0.0.1:3000/volume/1

本当は自分で書いたコードにたいして「こーゆーつもりで書いたんですよ」とか恥を晒し…もとい説明しながら進めるべきなんですが、ちょっとコードがスッカスカすぎて何も説明することがないので、とりあえず今日はこれまで。


