+++
date = "2011-04-27T01:05:29+09:00"
title = "AsakusaSatellite の WebSocket"
tags = ["AsakusaSatellite"]
+++

CentOS 5.5 に入れた AsakusaSatellite で WebSocket を起動してシェルから抜けると WebSocket が終了しちゃうので、daemontools で起動することにした。って、本当はどうやるのがベストなんだろう？
```
 # AsakusaSatellite のパスは適宜読み替えてください
 cd /var/opt/AsakusaSatellite
 
 # daemontools 用のディレクトリを作成
 mkdir -p ./daemontools/log
 
 # WebSocket を起動するスクリプトの設置
 cat <<EOF> ./daemontools/run
 #!/bin/bash
 exec 2>&1
 exec setuidgid apache /usr/bin/ruby /var/opt/AsakusaSatellite/websocket/server.rb
 EOF
 
 # ロガーを起動するスクリプトの設置
 cat <<EOF> ./daemontools/log/run
 #!/bin/sh
 exec setuidgid apache multilog t ./main
 EOF
 
 # パーミッションの設定
 chown -R apache:apache ./daemontools/
 chmod +x ./daemontools/run
 chmod +x ./daemontools/log/run
 
 # daemontools の監視対象に設定
 cd /service/
 ln -sf /var/opt/AsakusaSatellite/daemontools as_websock
```
