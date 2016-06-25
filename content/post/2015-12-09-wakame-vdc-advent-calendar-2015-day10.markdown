---
title: "Ten てこまい"
date: 2015-12-09 04:52:51 +0900
tags: ["Wakame-vdc","AdventCalendar"]
---

Wakame-vdc ユーザの皆さん、磯野家の皆さん、こんにちは。これはオレオレ WakameVdc / OpenVNet Advent Calendar 2015 の10日目のエントリです。

<!--more-->

むぅ…先日とは違うけど、やっぱりタイムアウトになって起動できず…。

    # cat /var/log/wakame-vdc/hva.log
    D, [2015-12-08T18:36:00.942307 #3501] DEBUG -- ServiceNetfilter: Subscribing to: hva.node2ebd61a1c668/vnic_created
    D, [2015-12-08T18:36:00.942558 #3501] DEBUG -- ServiceNetfilter: Subscribing to: hva.node2ebd61a1c668/vnic_destroyed
    D, [2015-12-08T18:36:00.942689 #3501] DEBUG -- ServiceNetfilter: Subscribing to: broadcast/debug/vnet
    2015-12-08 18:36:00 Node thr=#<Thread:0x007f400c993d58> [INFO]: Started : AMQP Server=amqp://127.0.0.1/, ID=hva.node2ebd61a1c668, token=4a2c3
    I, [2015-12-08T18:36:00.943946 #3501]  INFO -- NetfilterCache: updating cache from database
    2015-12-08 18:38:53 JobContext thr=LocalStore[1/2] [INFO]: Job start 38e342ad524c2ced688c86964e8986b89d418c37 (Local ID: 34c12b506bed8ad02241dae5d432ba5754c82a04)[ run_local_store ]
    I, [2015-12-08T18:38:53.751621 #3501]  INFO -- HvaHandler: Session ID: 38e342ad524c2ced688c86964e8986b89d418c37: Instance UUID: i-idhvphne: Booting i-idhvphne: phase 1
    I, [2015-12-08T18:38:53.811237 #3501]  INFO -- HvaHandler: Session ID: 38e342ad524c2ced688c86964e8986b89d418c37: Instance UUID: i-idhvphne: Creating volume vol-834kwo5p from bo-centos66.
    I, [2015-12-08T18:38:53.840009 #3501]  INFO -- LinuxLocalStore: Session ID: 38e342ad524c2ced688c86964e8986b89d418c37: Deploying image: wmi-centos66 from http://192.0.2.1:8000/images/centos-6.6.x86_64.lxc.md.raw.gz to /var/lib/wakame-vdc/instances/i-idhvphne/vol-834kwo5p
    I, [2015-12-08T18:38:53.840360 #3501]  INFO -- LinuxLocalStore: Session ID: 38e342ad524c2ced688c86964e8986b89d418c37: Executing command: curl -fsS http://192.0.2.1:8000/images/centos-6.6.x86_64.lxc.md.raw.gz| gunzip | pv -W -f -p -s 321491 | cp --sparse=always /dev/stdin /var/lib/wakame-vdc/instances/i-idhvphne/vol-834kwo5p
    D, [2015-12-08T18:39:21.558943 #3501] DEBUG -- LinuxLocalStore: Session ID: 38e342ad524c2ced688c86964e8986b89d418c37: Command Result: success (exit code=0)
    Command PID: 4370
    ##STDERR=>
    [=====================================================================] 100000%
    2015-12-08 18:42:10 ThreadPool thr=InstanceMonitor[0/1] [ERROR]: Caught Isono::NodeModules::RpcChannel::RpcError: timeout
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/node_modules/rpc_channel.rb:466:in `wait'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/node_modules/rpc_channel.rb:153:in `request'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/node_modules/instance_monitor.rb:31:in `check_instance'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/node_modules/instance_monitor.rb:12:in `block (3 levels) in <class:InstanceMonitor>'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/thread_pool.rb:32:in `call'
    	/opt/axsh/wakame-vdc/dcmgr/vendor/bundle/ruby/2.0.0/gems/isono-0.2.20/lib/isono/thread_pool.rb:32:in `block (2 levels) in initialize'
    2015-12-08 18:42:30 JobContext thr=LocalStore[1/2] [ERROR]: Job failed 38e342ad524c2ced688c86964e8986b89d418c37 [ run_local_store ]: timeout
    2015-12-08 18:42:30 JobContext thr=LocalStore[1/2] [ERROR]: Caught Isono::NodeModules::RpcChannel::RpcError: timeout
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
    I, [2015-12-08T18:42:30.097961 #3501]  INFO -- HvaHandler: Session ID: 38e342ad524c2ced688c86964e8986b89d418c37: Instance UUID: i-idhvphne: Cleaning volume vol-834kwo5p
    I, [2015-12-08T18:42:30.098506 #3501]  INFO -- LinuxLocalStore: Session ID: 38e342ad524c2ced688c86964e8986b89d418c37: Executing command: rm '/var/lib/wakame-vdc/instances/i-idhvphne/vol-834kwo5p'
    D, [2015-12-08T18:42:30.252159 #3501] DEBUG -- LinuxLocalStore: Session ID: 38e342ad524c2ced688c86964e8986b89d418c37: Command Result: success (exit code=0)
    Command PID: 4491
    2015-12-08 18:42:36 JobContext thr=LocalStore[0/2] [INFO]: Job start 4899f11c00630167af8a89f641432b9fd0017db6 (Local ID: 4899f11c00630167af8a89f641432b9fd0017db6)[ delete_volume ]
    D, [2015-12-08T18:42:36.611734 #3501] DEBUG -- ServiceNetfilter: event caught: hva.node2ebd61a1c668/vnic_destroyed: vif-ac9mzzo0
    D, [2015-12-08T18:42:36.611876 #3501] DEBUG -- ServiceNetfilter: vnic not found in cache: vif-ac9mzzo0
    W, [2015-12-08T18:42:36.667704 #3501]  WARN -- HvaHandler: Session ID: 4899f11c00630167af8a89f641432b9fd0017db6: Instance UUID: i-idhvphne: Unexpected state deleted of vol-834kwo5p. But try to delete.
    W, [2015-12-08T18:42:36.668136 #3501]  WARN -- HvaHandler: Session ID: 4899f11c00630167af8a89f641432b9fd0017db6: Instance UUID: i-idhvphne: Failed to delete non-existing file: /var/lib/wakame-vdc/instances/i-idhvphne/vol-834kwo5p
    2015-12-08 18:42:36 JobContext thr=LocalStore[0/2] [INFO]: Job complete 4899f11c00630167af8a89f641432b9fd0017db6 (Local ID: 4899f11c00630167af8a89f641432b9fd0017db6)[ delete_volume ]: 0.069073835 sec

っことで、ログ的に問題が発生する直前から何が起きているかを追っていく旅に出ます。

    I, [2015-12-08T18:38:53.840360 #3501]  INFO -- LinuxLocalStore: Session ID: 38e342ad524c2ced688c86964e8986b89d418c37: Executing command: curl -fsS http://192.0.2.1:8000/images/centos-6.6.x86_64.lxc.md.raw.gz| gunzip | pv -W -f -p -s 321491 | cp --sparse=always /dev/stdin /var/lib/wakame-vdc/instances/i-idhvphne/vol-834kwo5p
    D, [2015-12-08T18:39:21.558943 #3501] DEBUG -- LinuxLocalStore: Session ID: 38e342ad524c2ced688c86964e8986b89d418c37: Command Result: success (exit code=0)
    Command PID: 4370
    ##STDERR=>
    [=====================================================================] 100000%

これがコードのどの部分かをまず探します。LinuxLocalStore って出てるので、おそらくコード的にはここら辺だと思います。[/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/local_store/linux_local_store.rb](https://github.com/axsh/wakame-vdc/blob/v16.1.0/dcmgr/lib/dcmgr/drivers/local_store/linux_local_store.rb)

見ていくと[curl -fsS http://192.0.2.1:8000/images/centos-6.6.x86_64.lxc.md.raw.gz| gunzip | pv -W -f -p -s 321491 | cp --sparse=always /dev/stdin /var/lib/wakame-vdc/instances/i-idhvphne/vol-834kwo5p](https://github.com/axsh/wakame-vdc/blob/v16.1.0/dcmgr/lib/dcmgr/drivers/local_store/linux_local_store.rb#L89-L93) は無事に実行できているので、手動でもちゃんと動くか試してみます。

    # curl -fsS http://192.0.2.1:8000/images/centos-6.6.x86_64.lxc.md.raw.gz| gunzip | pv -W -f -p -s 321491 | cp --sparse=always /dev/stdin /var/lib/wakame-vdc/instances/i-idhvphne/vol-834kwo5p
    [============================================================================================] 100000%
    
    # file /var/lib/wakame-vdc/instances/i-idhvphne/vol-834kwo5p 
    /var/lib/wakame-vdc/instances/i-idhvphne/vol-834kwo5p: x86 boot sector; partition 1: ID=0x83, starthead 1, startsector 63, 8386498 sectors, code offset 0xb8
    [root@host-133-130-109-122 wakame]# fdisk -l /var/lib/wakame-vdc/instances/i-idhvphne/vol-834kwo5p
    You must set cylinders.
    You can do this from the extra functions menu.
    
    Disk /var/lib/wakame-vdc/instances/i-idhvphne/vol-834kwo5p: 0 MB, 0 bytes
    4 heads, 32 sectors/track, 0 cylinders
    Units = cylinders of 128 * 512 = 65536 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000e34e5
    
                                                    Device Boot      Start         End      Blocks   Id  System
    /var/lib/wakame-vdc/instances/i-idhvphne/vol-834kwo5p1               1       65521     4193249   83  Linux
    Partition 1 has different physical/logical endings:
     phys=(1023, 3, 32) logical=(65520, 0, 1)

ファイル的にも問題なさそうに見えます。で、処理はこのあとこの deploy_volume() を抜けて deploy_image() に戻り、そこも抜けるので、LinuxLocalStore::deploy_image() を呼んでいるところを探します。

    # grep deploy_image -r /opt/axsh/
    /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/local_store.rb:      def deploy_image(inst,ctx)
    /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/local_store/dummy_local_store.rb:      def deploy_image(inst,ctx)
    /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/local_store/esxi_local_store.rb:      def deploy_image(inst,ctx)
    /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/local_store/linux_local_store.rb:      def deploy_image(inst, ctx)

あれ？誰も呼んでない…？
じゃ、じゃぁ LinuxLocalStore::deploy_volume() が直接呼ばれてる？

    # grep deploy_volume -r /opt/axsh/wakame-vdc/dcmgr/
    /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/local_store.rb:      def deploy_volume(hva_ctx, volume, backup_object, opts={})
    /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/backing_store/raw.rb:        # Basically, deploy_volume() deals with image file as:
    /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/backing_store/raw.rb:        def deploy_volume_from_backup_object(sta_ctx, local_backup_path=nil)
    /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/backing_store/raw.rb:        deploy_volume_from_backup_object(ctx, backup_real_path(@backup_object[:object_key]))
    /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/backing_store/raw.rb:        deploy_volume_from_backup_object(ctx)
    /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/backing_store/raw.rb:            deploy_volume_from_backup_object(ctx, backup_real_path(backup_key))
    /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/backing_store/raw.rb:            deploy_volume_from_backup_object(ctx)
    /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/local_store/dummy_local_store.rb:      def deploy_volume(hva_ctx, volume, backup_object, opts={})
    /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/local_store/linux_local_store.rb:      def deploy_volume(hva_ctx, volume, backup_object, opts={:cache=>false})
    /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/local_store/linux_local_store.rb:            deploy_volume(@ctx, v, inst[:image][:backup_object], {:cache=>inst[:image][:is_cacheable]})
    /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/local_store/linux_local_store.rb:            deploy_volume(@ctx, v, v[:backup_object])
    /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/rpc/local_store_handler.rb:                              :deploy_volume, [@hva_ctx, v, v[:backup_object], opts])
    /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/rpc/local_store_handler.rb:      job :deploy_volume_and_attach, proc {

ってことで、[/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/rpc/local_store_handler.rb](https://github.com/axsh/wakame-vdc/blob/v16.1.0/dcmgr/lib/dcmgr/rpc/local_store_handler.rb) を追います。

ログで LinuxLocalStore を呼ぶ前が HvaHandler で "Creating volume" と言っているので、

    I, [2015-12-08T18:38:53.811237 #3501]  INFO -- HvaHandler: Session ID: 38e342ad524c2ced688c86964e8986b89d418c37: Instance UUID: i-idhvphne: Creating volume vol-834kwo5p from bo-centos66.
    I, [2015-12-08T18:38:53.840009 #3501]  INFO -- LinuxLocalStore: Session ID: 38e342ad524c2ced688c86964e8986b89d418c37: Deploying image: wmi-centos66 from http://192.0.2.1:8000/images/centos-6.6.x86_64.lxc.md.raw.gz to /var/lib/wakame-vdc/instances/i-idhvphne/vol-834kwo5p

コードの[この場所](https://github.com/axsh/wakame-vdc/blob/v16.1.0/dcmgr/lib/dcmgr/rpc/local_store_handler.rb#L41-L42) が deploy_volume を呼んでいるところになります。

なのでそこの処理が抜けると次は[ここ](https://github.com/axsh/wakame-vdc/blob/v16.1.0/dcmgr/lib/dcmgr/rpc/local_store_handler.rb#L49) に入って、[ここ](https://github.com/axsh/wakame-vdc/blob/v16.1.0/dcmgr/lib/dcmgr/rpc/hva_handler.rb#L166-L184) が呼ばれています。

でそこを抜けると戻って [ここ](https://github.com/axsh/wakame-vdc/blob/v16.1.0/dcmgr/lib/dcmgr/rpc/local_store_handler.rb#L66) が叩かれて…

とちょっと追う様を解説していくのが辛くなってきました。本日のところはこれまで。


