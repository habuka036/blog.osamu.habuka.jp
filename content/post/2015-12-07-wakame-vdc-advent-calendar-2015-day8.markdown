---
title: "8にちめ"
date: 2015-12-07 17:43:14 +0900
tags: ["Wakame-vdc","AdventCalendar"]
---

Wakame-vdc ユーザの皆さん、磯野家の皆さん、こんにちは。これはオレオレ WakameVdc / OpenVNet Advent Calendar 2015 の8日目のエントリです。

今日は昨日の続きで、マシンイメージを CentOS かつ lxc 版なものに変更してインスタンスを起動してみます。

<!--more-->

先日、メーリングリストで「lxc 用のイメージってどこぉ？」って質問をしたら Potter さんがリンクを教えてくれたので、まずそれをダウンロードします。

    # wget http://dlc2.wakame.axsh.jp/demo/1box/vmimage/centos-6.6.x86_64.lxc.md.raw.tar.gz

で、ダウンロードしたらマシンイメージの情報を得るため、以下を実行します。

    # tar -xzvf centos-6.6.x86_64.lxc.md.raw.tar.gz 
    centos-6.6.x86_64.lxc.md.raw
    # LANG=C ls -kl centos-6.6.x86_64.lxc.md.raw.tar.gz centos-6.6.x86_64.lxc.md.raw
    -rw-r--r-- 1 root root 4194304 Apr 19  2015 centos-6.6.x86_64.lxc.md.raw
    -rw-r--r-- 1 root root  317193 Apr 19  2015 centos-6.6.x86_64.lxc.md.raw.tar.gz
    # md5sum centos-6.6.x86_64.lxc.md.raw.tar.gz
    45e35271d4b45e274b4952240799d9d4  centos-6.6.x86_64.lxc.md.raw.tar.gz
    # kpartx -v -a centos-6.6.x86_64.lxc.md.raw
    add map loop0p1 (253:2): 0 8386498 linear /dev/loop0 63
    # blkid | grep loop0p1
    /dev/mapper/loop0p1: LABEL="root" UUID="eb69a6cf-fc3f-42cd-9b21-c70ad78f6d9e" TYPE="ext4" 

で、ダウンロードしたマシンイメージを登録します。

    # /opt/axsh/wakame-vdc/dcmgr/bin/vdc-manage backupobject add \
    > --uuid bo-centos66 --display-name "CentOS 6.6 x86_64 root partition" \
    > --storage-id bkst-local --object-key centos-6.6.x86_64.lxc.md.raw.tar.gz \
    > --size 317193 --allocation-size 4194304 --container-format gz \
    > --checksum  45e35271d4b45e274b4952240799d9d4
    bo-centos66
    
    # /opt/axsh/wakame-vdc/dcmgr/bin/vdc-manage image add local bo-centos66 \
    > --account-id a-shpoolxx --uuid wmi-centos66 \
    > --root-device uuid:eb69a6cf-fc3f-42cd-9b21-c70ad78f6d9e \
    > --display-name "CentOS 6.6 x86_64"
    wmi-centos66

backupobject add の「--size」には centos-6.6.x86_64.lxc.md.raw.tar.gz のサイズ(KB)の値を、「--allocation-size」には tar玉を展開したファイル「centos-6.6.x86_64.lxc.md.raw」のサイズ(KB)の値を、「--checksum」にはtar玉の md5sum の値を入れます。たぶん。

image add local の「--root-device」には blkid で確認した UUID を指定します。

![](/images/wakame-vdc.adventcalendar.2015.1208-01.png )

こんな感じで登録されました。
で、早速起動できるかを試してみましたが、速攻で terminated になってました。

    I, [2015-12-08T02:23:54.192699 #2918]  INFO -- LinuxLocalStore: Session ID: 1b8debb3740a08af60b9e02dfb0c43ed8b80a4b6: Executing command: cat /var/lib/wakame-vdc/images/centos-6.6.x86_64.lxc.md.raw.tar.gz| gunzip | pv -W -f -p -s 317193 | cp --sparse=always /dev/stdin /var/lib/wakame-vdc/instances/i-ysmhdpjf/vol-8a0wk702
    D, [2015-12-08T02:23:54.209337 #2918] DEBUG -- LinuxLocalStore: Session ID: 1b8debb3740a08af60b9e02dfb0c43ed8b80a4b6: Command Result: success (exit code=0)
    Command PID: 3230
    ##STDERR=>
    cat: /var/lib/wakame-vdc/images/centos-6.6.x86_64.lxc.md.raw.tar.gz: No such file or directory 
    gzip: stdin: unexpected end of file

ん？登録した tar 玉がない？

    # ls -l /var/lib/wakame-vdc/images/
    total 305212
    -rw-r--r-- 1 root root 312530604 Dec  7 05:41 ubuntu-14.04.3-x86_64-30g-passwd-login-enabled.raw.tgz

うん、確かにない……あ！あぁっ！そうでした、Eucalyptus や OpenStack に毒されてすっかり忘れてましたが、Wakame-vdc での backupobject add や image add はマシンイメージの情報を登録するだけで、実体のファイルは自分でストレージの種類に合わせてアップロードする必要があります。たぶん。

ってことで、/var/lib/wakame-vdc/images/ 配下に tar 玉を置きます。

    # mv centos-6.6.x86_64.lxc.md.raw.tar.gz /var/lib/wakame-vdc/images/
    # ls -l /var/lib/wakame-vdc/images/
    total 622412
    -rw-r--r-- 1 root root 324805362 Apr 19  2015 centos-6.6.x86_64.lxc.md.raw.tar.gz
    -rw-r--r-- 1 root root 312530604 Dec  7 05:41 ubuntu-14.04.3-x86_64-30g-passwd-login-enabled.raw.tgz

さて、もう一度起動してみます。

……また terminated になりました。

    I, [2015-12-08T03:34:19.002068 #2918]  INFO -- HvaHandler: Session ID: aa2084f3b25f316ed5e7e06bd9c013ff9ffb136c: Instance UUID: i-9nm5c1tx: Booting i-9nm5c1tx: phase 1
    I, [2015-12-08T03:34:19.043404 #2918]  INFO -- HvaHandler: Session ID: aa2084f3b25f316ed5e7e06bd9c013ff9ffb136c: Instance UUID: i-9nm5c1tx: Creating volume vol-44wrtwwu from bo-centos66.
    I, [2015-12-08T03:34:19.044296 #2918]  INFO -- LinuxLocalStore: Session ID: aa2084f3b25f316ed5e7e06bd9c013ff9ffb136c: Deploying image: wmi-centos66 from file:///var/lib/wakame-vdc/images/centos-6.6.x86_64.lxc.md.raw.tar.gz to /var/lib/wakame-vdc/instances/i-9nm5c1tx/vol-44wrtwwu
    I, [2015-12-08T03:34:19.045121 #2918]  INFO -- LinuxLocalStore: Session ID: aa2084f3b25f316ed5e7e06bd9c013ff9ffb136c: Executing command: cat /var/lib/wakame-vdc/images/centos-6.6.x86_64.lxc.md.raw.tar.gz| gunzip | pv -W -f -p -s 317193 | cp --sparse=always /dev/stdin /var/lib/wakame-vdc/instances/i-9nm5c1tx/vol-44wrtwwu
    D, [2015-12-08T03:34:25.204525 #2918] DEBUG -- LinuxLocalStore: Session ID: aa2084f3b25f316ed5e7e06bd9c013ff9ffb136c: Command Result: success (exit code=0)
    Command PID: 4172
    ##STDERR=>
    [=====================================================================] 100000%
    2015-12-08 03:34:25 JobContext thr=LocalStore[1/2] [INFO]: Job complete aa2084f3b25f316ed5e7e06bd9c013ff9ffb136c (Local ID: 84c5aa808ca2c8e5716491cfc3abe77fcadb2710)[ run_local_store ]: 6.211798189 sec
    2015-12-08 03:34:25 JobContext thr=JobWorker[0/1] [INFO]: Job start aa2084f3b25f316ed5e7e06bd9c013ff9ffb136c (Local ID: fc17b9446d7201916f03da8f2859a006515e3065)[ run_local_store ]
    I, [2015-12-08T03:34:25.218415 #2918]  INFO -- HvaHandler: Session ID: aa2084f3b25f316ed5e7e06bd9c013ff9ffb136c: Instance UUID: i-9nm5c1tx: Booting instance
    I, [2015-12-08T03:34:25.255464 #2918]  INFO -- Lxc: Session ID: aa2084f3b25f316ed5e7e06bd9c013ff9ffb136c: lxc-version: 1.0.8
    2015-12-08 03:34:25 JobContext thr=JobWorker[0/1] [ERROR]: Job failed aa2084f3b25f316ed5e7e06bd9c013ff9ffb136c [ run_local_store ]: cgroup filesystem is not mounted.
    2015-12-08 03:34:25 JobContext thr=JobWorker[0/1] [ERROR]: Caught RuntimeError: cgroup filesystem is not mounted.
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/hypervisor/linux_hypervisor/linux_container/lxc.rb:54:in `initialize'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/hypervisor/linux_hypervisor/linux_container/lxc.rb:162:in `new'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/hypervisor/linux_hypervisor/linux_container/lxc.rb:162:in `block in <class:Lxc>'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/task.rb:95:in `call'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/task.rb:95:in `new_task_class'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/task.rb:243:in `invoke'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/rpc/hva_handler.rb:212:in `setup_metadata_drive'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/rpc/hva_handler.rb:315:in `block in <class:HvaHandler>'
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
    I, [2015-12-08T03:34:25.256199 #2918]  INFO -- HvaHandler: Session ID: aa2084f3b25f316ed5e7e06bd9c013ff9ffb136c: Instance UUID: i-9nm5c1tx: teminating instance
    I, [2015-12-08T03:34:25.262402 #2918]  INFO -- Lxc: Session ID: aa2084f3b25f316ed5e7e06bd9c013ff9ffb136c: lxc-version: 1.0.8
    E, [2015-12-08T03:34:25.262739 #2918] ERROR -- HvaHandler: Session ID: aa2084f3b25f316ed5e7e06bd9c013ff9ffb136c: Instance UUID: i-9nm5c1tx: Ignoring error: RuntimeError cgroup filesystem is not mounted. from /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/hypervisor/linux_hypervisor/linux_container/lxc.rb:54:in `initialize'
    I, [2015-12-08T03:34:25.270003 #2918]  INFO -- LinuxLocalStore: Session ID: aa2084f3b25f316ed5e7e06bd9c013ff9ffb136c: Executing command: rm '/var/lib/wakame-vdc/instances/i-9nm5c1tx/vol-44wrtwwu'
    D, [2015-12-08T03:34:25.398113 #2918] DEBUG -- LinuxLocalStore: Session ID: aa2084f3b25f316ed5e7e06bd9c013ff9ffb136c: Command Result: success (exit code=0)
    Command PID: 4184

う〜ん、「cgroup filesystem is not mounted」って言ってますが、続きはまた明日。

