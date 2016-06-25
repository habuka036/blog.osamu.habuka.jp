---
title: "Dynamo を触ってみる"
date: 2014-08-15 13:12:23 +0900
tags: ["Elixir"]
---

僕は普段プログラムを書かない似非エンジニアでして…っていうか技術者っていうより作業者って名乗ったほうが正しいんじゃないかと思う今日この頃でして、そんなんじゃお客様の前に自信を持って立てないから、少しは技術者のフリができるようにしようと思い、elixir の Dynamo という Sinatra 風の Web Framework を触ってみようと思います。

ちなみに今日のこのエントリーは「失敗編」です。読んでも役に立つ部分が少ないです。

<!--more-->

環境
----

私の環境は

* erlang: Erlang/OTP 17
* elixir: Elixir 0.14.4-dev

という環境です。

インストール
------------

公式ドキュメントに従ってコンパイルをしてみます。

    $ mix do deps.get, compile
    mix.exs:38: warning: System.cmd/1 is deprecated, use System.cmd/3 instead
    All dependencies up to date
    Compiled lib/dynamo/base.ex
    Compiled lib/dynamo.ex
    
    == Compilation error on file lib/dynamo/connection/query_parser.ex ==
    ** (CompileError) lib/dynamo/connection/query_parser.ex:2: undefined function defexception/2
        (elixir) src/elixir.erl:202: :elixir.quoted_to_erl/3
        (stdlib) erl_eval.erl:657: :erl_eval.do_apply/6
        (elixir) src/elixir.erl:170: :elixir.erl_eval/3

はい、Dynamo のコンパイルでエラーが出ます。

で、Dynamo の BTS を見ると、以下のように elixir 0.14.1 以降でコンパイルがコケてるようです。
https://github.com/dynamo/dynamo/issues/239

しかし、stevedomin さんがとりあえず 0.14.1 で通るように直してみたらしいので、stevedomin さんのリポジトリを持ってきて試してみます。


    $ cd ..
    $ git clone https://github.com/stevedomin/dynamo.git stevedomin/dynamo
    (snip)
    $ cd stevedomin/dynamo/
    $ git checkout elixir-0.14.1
    (snip)
    $ mix do deps.get, compile
    mix.exs:38: warning: System.cmd/1 is deprecated, use System.cmd/3 instead
    * Getting mime (git://github.com/dynamo/mime.git)
    Cloning into '/home/osamu/local/stevedomin/dynamo/deps/mime'...
    remote: Counting objects: 63, done.
    (snip)
    Compiled lib/dynamo/connection/utils.ex
    
    == Compilation error on file lib/dynamo/cowboy/body_parser.ex ==
    ** (CompileError) lib/dynamo/cowboy/body_parser.ex:100: function size/1 undefined
        (stdlib) lists.erl:1336: :lists.foreach/2
        (stdlib) erl_eval.erl:657: :erl_eval.do_apply/6
        (elixir) src/elixir.erl:170: :elixir.erl_eval/3
        (elixir) src/elixir.erl:158: :elixir.eval_forms/4
        (elixir) src/elixir_lexical.erl:17: :elixir_lexical.run/2
        (elixir) src/elixir.erl:170: :elixir.erl_eval/3


という感じで、一難さってまた一難っていう感じにコンパイルがまたもやコケます。

うーん、さすがに elixir のことが全然理解していないので、まず先にそっちを理解するほうが先ですね…σ^^;

「成功編」を書く日が来るのかどうかとても疑わしいですが、本日はこんな感じでおしまい。


