---
title: "LXC と情熱のあいだ"
date: 2015-12-06 05:23:52 +0900
tags: ["Wakame-vdc","AdventCalendar"]
---

Wakame-vdc ユーザの皆さん、磯野家の皆さん、こんにちは。これはオレオレ WakameVdc / OpenVNet Advent Calendar 2015 の5日目のエントリです。

順調に「日々更新ができなくなってきている」状態ですが、細かいことは気にせずマイペースで行きたいと思います。

今日も小粒なネタです。3日目の続きを。

<!--more-->

Wakame-vdc LiveDVD の設定を KVM から lxc に作り変え、イメージも lxc 用を登録してみました。

    I, [2015-12-05T21:16:58.697938 #3499]  INFO -- LinuxLocalStore: Session ID: 5b9118b596ebe1a208ba74e569fed366b96e21a8: Deploying image: wmi-centos66 from http://192.0.2.1:8000/images/centos-6.6.x86_64.lxc.md.raw.gz to /var/lib/wakame-vdc/instances/i-ungzbmvf/vol-iz4ojoqc
    I, [2015-12-05T21:16:58.698242 #3499]  INFO -- LinuxLocalStore: Session ID: 5b9118b596ebe1a208ba74e569fed366b96e21a8: Executing command: curl -fsS http://192.0.2.1:8000/images/centos-6.6.x86_64.lxc.md.raw.gz| gunzip | pv -W -f -p -s 321491 | cp --sparse=always /dev/stdin /var/lib/wakame-vdc/instances/i-ungzbmvf/vol-iz4ojoqc
    D, [2015-12-05T21:17:24.326757 #3499] DEBUG -- LinuxLocalStore: Session ID: 5b9118b596ebe1a208ba74e569fed366b96e21a8: Command Result: success (exit code=0)
    Command PID: 4462
    ##STDERR=>
    [=====================================================================] 100000%
    2015-12-05 21:20:11 ThreadPool thr=InstanceMonitor[0/1] [ERROR]: Caught Isono::NodeModules::RpcChannel::RpcError: timeout
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/node_modules/rpc_channel.rb:466:in `wait'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/node_modules/rpc_channel.rb:153:in `request'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/node_modules/instance_monitor.rb:31:in `check_instance'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/node_modules/instance_monitor.rb:12:in `block (3 levels) in <class:InstanceMonitor>'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/thread_pool.rb:32:in `call'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/thread_pool.rb:32:in `block (2 levels) in initialize'
    2015-12-05 21:20:31 JobContext thr=LocalStore[0/2] [ERROR]: Job failed 5b9118b596ebe1a208ba74e569fed366b96e21a8 [ run_local_store ]: timeout
    2015-12-05 21:20:31 JobContext thr=LocalStore[0/2] [ERROR]: Caught Isono::NodeModules::RpcChannel::RpcError: timeout
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/node_modules/rpc_channel.rb:466:in `wait'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/node_modules/rpc_channel.rb:153:in `request'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/rpc/hva_handler.rb:168:in `update_volume_state'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/rpc/local_store_handler.rb:49:in `deploy_local_volume'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/rpc/local_store_handler.rb:109:in `block (2 levels) in <class:LocalStoreHandler>'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/rpc/local_store_handler.rb:21:in `call'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/rpc/local_store_handler.rb:21:in `block in each_all_local_volumes'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/rpc/local_store_handler.rb:20:in `each'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/rpc/local_store_handler.rb:20:in `each_all_local_volumes'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/rpc/local_store_handler.rb:108:in `block in <class:LocalStoreHandler>'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/runner/rpc_server.rb:69:in `instance_eval'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/runner/rpc_server.rb:69:in `block in job'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/rack/proc.rb:25:in `instance_eval'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/rack/proc.rb:25:in `call'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/rack/map.rb:52:in `call'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/rack/map.rb:52:in `call'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/rack/builder.rb:36:in `call'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/rack/job.rb:59:in `block (2 levels) in call'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/node_modules/job_worker.rb:67:in `call'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/node_modules/job_worker.rb:67:in `block in start'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/thread_pool.rb:32:in `call'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/thread_pool.rb:32:in `block (2 levels) in initialize'
    I, [2015-12-05T21:20:31.544538 #3499]  INFO -- HvaHandler: Session ID: 5b9118b596ebe1a208ba74e569fed366b96e21a8: Instance UUID: i-ungzbmvf: Cleaning volume vol-iz4ojoqc
    I, [2015-12-05T21:20:31.544769 #3499]  INFO -- LinuxLocalStore: Session ID: 5b9118b596ebe1a208ba74e569fed366b96e21a8: Executing command: rm '/var/lib/wakame-vdc/instances/i-ungzbmvf/vol-iz4ojoqc'
    D, [2015-12-05T21:20:31.668489 #3499] DEBUG -- LinuxLocalStore: Session ID: 5b9118b596ebe1a208ba74e569fed366b96e21a8: Command Result: success (exit code=0)
    Command PID: 4559
    D, [2015-12-05T21:20:38.593790 #3499] DEBUG -- ServiceNetfilter: event caught: hva.node16aff8e215e1/vnic_destroyed: vif-8glbtzu1
    D, [2015-12-05T21:20:38.593938 #3499] DEBUG -- ServiceNetfilter: vnic not found in cache: vif-8glbtzu1
    2015-12-05 21:20:38 JobContext thr=LocalStore[1/2] [INFO]: Job start c39a19a2b883928928b7fe6df7345c3b6c86c678 (Local ID: c39a19a2b883928928b7fe6df7345c3b6c86c678)[ delete_volume ]
    W, [2015-12-05T21:20:38.660525 #3499]  WARN -- HvaHandler: Session ID: c39a19a2b883928928b7fe6df7345c3b6c86c678: Instance UUID: i-ungzbmvf: Unexpected state deleted of vol-iz4ojoqc. But try to delete.
    W, [2015-12-05T21:20:38.660895 #3499]  WARN -- HvaHandler: Session ID: c39a19a2b883928928b7fe6df7345c3b6c86c678: Instance UUID: i-ungzbmvf: Failed to delete non-existing file: /var/lib/wakame-vdc/instances/i-ungzbmvf/vol-iz4ojoqc
    2015-12-05 21:20:38 JobContext thr=LocalStore[1/2] [INFO]: Job complete c39a19a2b883928928b7fe6df7345c3b6c86c678 (Local ID: c39a19a2b883928928b7fe6df7345c3b6c86c678)[ delete_volume ]: 0.072499295 sec

うぅ〜ん？起動失敗です…。一旦 LiveDVD じゃなく普通の Wakame-vdc 環境を作ってそこで lxc を試さないといけないですね…。また明日。

