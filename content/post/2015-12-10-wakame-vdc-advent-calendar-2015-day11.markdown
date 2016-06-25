---
title: "振り返れば RabbitMQ"
date: 2015-12-10 01:20:28 +0900
tags: ["Wakame-vdc","AdventCalendar"]
---

Wakame-vdc ユーザの皆さん、磯野家の皆さん、こんにちは。これはオレオレ WakameVdc / OpenVNet Advent Calendar 2015 の11日目のエントリです。

「学習能力ないのか！」とか「何で予測できないんだ！」とか「ねぇ？ワザとやってる？ねぇ？」とか言われそうな LiveDVD 作成奮闘記。

インスタンスを起動したときに isono がタイムアウトになるのって度々遭遇していたことをすっかり忘れていました。そう[このチケット](https://github.com/axsh/iso-no-wakame/issues/11)で。

<!--more-->

RabbitMQ のデフォルト値では disk_free_limit が 1GB なんですよね。なので、

    [root@host-133-130-109-122 wakame]# df -h
    Filesystem           Size  Used Avail Use% Mounted on
    rootfs               4.0G  3.0G  960M  76% /
    devtmpfs             3.9G  196K  3.9G   1% /dev
    tmpfs                4.0G   76K  4.0G   1% /dev/shm
    /dev/sr0             1.2G  1.2G     0 100% /dev/.initramfs/live
    /dev/mapper/live-rw  4.0G  3.0G  960M  76% /

起動した直後で既に 1GB を下回っているので、いつリソース不足になって上記の件が再現してもおかしくない状況でした。事実、インスタンスを起動すると当然ながらディスクの空き容量は減っていき…

    [root@host-133-130-109-122 wakame]# df -h
    Filesystem           Size  Used Avail Use% Mounted on
    rootfs               4.0G  3.6G  334M  92% /
    devtmpfs             3.9G  196K  3.9G   1% /dev
    tmpfs                4.0G   76K  4.0G   1% /dev/shm
    /dev/sr0             1.2G  1.2G     0 100% /dev/.initramfs/live
    /dev/mapper/live-rw  4.0G  3.6G  334M  92% /
    
    [root@host-133-130-109-122 wakame]# rabbitmqctl list_connections
    Listing connections ...
    guest	127.0.0.1	33257	blocked
    guest	127.0.0.1	33260	blocked
    guest	127.0.0.1	33277	blocking
    ...done.

ってな感じで blocked になってしまってました。

なので、以下を /etc/rabbitmq/rabbitmq.config として保存し、RabbitMQ を再起動します。

    [
      {rabbit, [{disk_free_limit, 100000000}]}
    ].

    # service rabbitmq-server restart
    Restarting rabbitmq-server: SUCCESS
    rabbitmq-server.

忘れずに cgroup もマウントします

    # mkdir -p /lxc/cgroup
    # mount -t cgroup lxc /lxc/cgroup
    # mount
    rootfs on / type rootfs (rw)
    proc on /proc type proc (rw,relatime)
    sysfs on /sys type sysfs (rw,relatime)
    devtmpfs on /dev type devtmpfs (rw,relatime,size=4085268k,nr_inodes=1021317,mode=755)
    devpts on /dev/pts type devpts (rw,relatime,gid=5,mode=620,ptmxmode=000)
    tmpfs on /dev/shm type tmpfs (rw,relatime)
    /dev/sr0 on /dev/.initramfs/live type iso9660 (ro,relatime)
    /dev/mapper/live-rw on / type ext3 (rw,relatime,errors=continue,user_xattr,acl,barrier=1,data=ordered)
    /proc/bus/usb on /proc/bus/usb type usbfs (rw,relatime)
    none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw,relatime)
    lxc on /lxc/cgroup type cgroup (rw,relatime,net_prio,perf_event,blkio,net_cls,freezer,devices,memory,cpuacct,cpu,ns,cpuset)

で、インスタンスを起動してみました…が起動はまたもや失敗しました。以下がそのログ。

    2015-12-09 17:17:27 JobContext thr=LocalStore[0/2] [INFO]: Job start 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6 (Local ID: 2175efba40f1f3db3a16d06422e38587931e32bd)[ run_local_store ]
    I, [2015-12-09T17:17:27.080397 #4706]  INFO -- HvaHandler: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Instance UUID: i-ahmv9hlu: Booting i-ahmv9hlu: phase 1
    I, [2015-12-09T17:17:27.161798 #4706]  INFO -- HvaHandler: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Instance UUID: i-ahmv9hlu: Creating volume vol-fx5ipori from bo-centos66.
    I, [2015-12-09T17:17:27.187042 #4706]  INFO -- LinuxLocalStore: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Deploying image: wmi-centos66 from http://192.0.2.1:8000/images/centos-6.6.x86_64.lxc.md.raw.gz to /var/lib/wakame-vdc/instances/i-ahmv9hlu/vol-fx5ipori
    I, [2015-12-09T17:17:27.187352 #4706]  INFO -- LinuxLocalStore: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Executing command: curl -fsS http://192.0.2.1:8000/images/centos-6.6.x86_64.lxc.md.raw.gz| gunzip | pv -W -f -p -s 321491 | cp --sparse=always /dev/stdin /var/lib/wakame-vdc/instances/i-ahmv9hlu/vol-fx5ipori
    D, [2015-12-09T17:17:52.871466 #4706] DEBUG -- LinuxLocalStore: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Command Result: success (exit code=0)
    Command PID: 4802
    ##STDERR=>
    [=====================================================================] 100000%
    2015-12-09 17:17:52 JobContext thr=LocalStore[0/2] [INFO]: Job complete 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6 (Local ID: 2175efba40f1f3db3a16d06422e38587931e32bd)[ run_local_store ]: 25.803586863 sec
    2015-12-09 17:17:52 JobContext thr=JobWorker[0/1] [INFO]: Job start 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6 (Local ID: ce60e0faf6159d41fa1c1dfbe7bf29e481dae4f9)[ run_local_store ]
    I, [2015-12-09T17:17:52.886761 #4706]  INFO -- HvaHandler: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Instance UUID: i-ahmv9hlu: Booting instance
    I, [2015-12-09T17:17:52.923605 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: lxc-version: 1.0.8
    I, [2015-12-09T17:17:52.924086 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Setting up metadata drive image:i-ahmv9hlu
    I, [2015-12-09T17:17:52.924258 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Executing command: /usr/bin/truncate -s 10m '/var/lib/wakame-vdc/instances/i-ahmv9hlu/metadata.img'
    D, [2015-12-09T17:17:52.929614 #4706] DEBUG -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Command Result: success (exit code=0)
    Command PID: 4832
    I, [2015-12-09T17:17:52.929811 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Executing command: parted /var/lib/wakame-vdc/instances/i-ahmv9hlu/metadata.img < /opt/axsh/wakame-vdc/dcmgr/templates/linux/metadata.parted
    D, [2015-12-09T17:17:52.942831 #4706] DEBUG -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Command Result: success (exit code=0)
    Command PID: 4833
    ##STDOUT=>
    GNU Parted 2.1
    Using /var/lib/wakame-vdc/instances/i-ahmv9hlu/metadata.img
    Welcome to GNU Parted! Type 'help' to view a list of commands.
    (parted) mklabel msdos                                                    
    (parted) mkpart primary fat32 1 10m                                       
    (parted) quit                                                             
    I, [2015-12-09T17:17:52.942999 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Executing command: kpartx -av /var/lib/wakame-vdc/instances/i-ahmv9hlu/metadata.img
    D, [2015-12-09T17:17:52.991573 #4706] DEBUG -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Command Result: success (exit code=0)
    Command PID: 4835
    ##STDOUT=>
    add map loop5p1 (253:4): 0 18432 linear /dev/loop5 2048
    I, [2015-12-09T17:17:52.991727 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Executing command: udevadm settle
    D, [2015-12-09T17:17:53.050253 #4706] DEBUG -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Command Result: success (exit code=0)
    Command PID: 5098
    I, [2015-12-09T17:17:53.050422 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Executing command: mkfs -t vfat -n METADATA /dev/mapper/loop5p1
    D, [2015-12-09T17:17:53.059890 #4706] DEBUG -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Command Result: success (exit code=0)
    Command PID: 5229
    ##STDOUT=>
    mkfs.vfat 3.0.9 (31 Jan 2010)
    unable to get drive geometry, using default 255/63
    I, [2015-12-09T17:17:53.060104 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Executing command: /bin/mount -t vfat /dev/mapper/loop5p1 '/var/lib/wakame-vdc/instances/i-ahmv9hlu/tmp'
    D, [2015-12-09T17:17:53.085562 #4706] DEBUG -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Command Result: success (exit code=0)
    Command PID: 5231
    I, [2015-12-09T17:17:53.087868 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Executing command: /bin/umount -l /var/lib/wakame-vdc/instances/i-ahmv9hlu/tmp
    D, [2015-12-09T17:17:53.097684 #4706] DEBUG -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Command Result: success (exit code=0)
    Command PID: 5239
    I, [2015-12-09T17:17:53.100518 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Executing command: dmsetup info /dev/mapper/loop5p1
    D, [2015-12-09T17:17:53.106061 #4706] DEBUG -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Command Result: success (exit code=0)
    Command PID: 5243
    ##STDOUT=>
    Name:              loop5p1
    State:             ACTIVE
    Read Ahead:        256
    Tables present:    LIVE
    Open count:        0
    Event number:      0
    Major, minor:      253, 4
    Number of targets: 1
    UUID: part1-loop5
    I, [2015-12-09T17:17:53.106330 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Executing command: dmsetup remove /dev/mapper/loop5p1
    D, [2015-12-09T17:17:53.130505 #4706] DEBUG -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Command Result: success (exit code=0)
    Command PID: 5245
    I, [2015-12-09T17:17:53.130607 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Detached partition from devmapper: /dev/mapper/loop5p1
    I, [2015-12-09T17:17:53.130694 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Executing command: udevadm settle
    D, [2015-12-09T17:17:53.136577 #4706] DEBUG -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Command Result: success (exit code=0)
    Command PID: 5249
    I, [2015-12-09T17:17:53.136757 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Executing command: losetup -d /dev/loop5
    D, [2015-12-09T17:17:53.141609 #4706] DEBUG -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Command Result: success (exit code=0)
    Command PID: 5251
    I, [2015-12-09T17:17:53.141725 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Detached from loop device: /dev/loop5
    I, [2015-12-09T17:17:54.166413 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Executing command: lxc-create -f /var/lib/wakame-vdc/instances/i-ahmv9hlu/lxc.conf -n i-ahmv9hlu
    D, [2015-12-09T17:17:54.172032 #4706] DEBUG -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Command Result: fail (exit code=1)
    Command PID: 5307
    ##STDERR=>
    A template must be specified.
    Use "none" if you really want a container without a rootfs.
    E, [2015-12-09T17:17:54.172207 #4706] ERROR -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Dcmgr::Helpers::CliHelper::ShellRunner::CommandError Unexpected exit code=1: lxc-create -f /var/lib/wakame-vdc/instances/i-ahmv9hlu/lxc.conf -n i-ahmv9hlu
    Command PID: 5307
    ##STDERR=>
    A template must be specified.
    Use "none" if you really want a container without a rootfs. from /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/helpers/cli_helper.rb:191:in `block in run!'
    2015-12-09 17:17:54 JobContext thr=JobWorker[0/1] [ERROR]: Job failed 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6 [ run_local_store ]: Unexpected exit code=1: lxc-create -f /var/lib/wakame-vdc/instances/i-ahmv9hlu/lxc.conf -n i-ahmv9hlu
    Command PID: 5307
    ##STDERR=>
    A template must be specified.
    Use "none" if you really want a container without a rootfs.
    2015-12-09 17:17:54 JobContext thr=JobWorker[0/1] [ERROR]: Caught Dcmgr::Helpers::CliHelper::ShellRunner::CommandError: Unexpected exit code=1: lxc-create -f /var/lib/wakame-vdc/instances/i-ahmv9hlu/lxc.conf -n i-ahmv9hlu
    Command PID: 5307
    ##STDERR=>
    A template must be specified.
    Use "none" if you really want a container without a rootfs.
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/helpers/cli_helper.rb:191:in `block in run!'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/helpers/cli_helper.rb:189:in `tap'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/helpers/cli_helper.rb:189:in `run!'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/helpers/cli_helper.rb:106:in `sh'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/hypervisor/linux_hypervisor/linux_container/lxc.rb:65:in `run_instance'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/task.rb:178:in `call'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/task.rb:178:in `invoke!'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/task.rb:249:in `invoke'
    	/opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/rpc/hva_handler.rb:323:in `block in <class:HvaHandler>'
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
    I, [2015-12-09T17:17:54.173472 #4706]  INFO -- HvaHandler: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Instance UUID: i-ahmv9hlu: teminating instance
    I, [2015-12-09T17:17:54.173666 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Executing command: lxc-stop -n i-ahmv9hlu
    D, [2015-12-09T17:17:54.198187 #4706] DEBUG -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Command Result: fail (exit code=2)
    Command PID: 5308
    ##STDERR=>
    i-ahmv9hlu is not running
    I, [2015-12-09T17:17:54.198349 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Executing command: lxc-wait -n i-ahmv9hlu -s STOPPED
    D, [2015-12-09T17:17:54.208155 #4706] DEBUG -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Command Result: success (exit code=0)
    Command PID: 5310
    I, [2015-12-09T17:17:54.208413 #4706]  INFO -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Executing command: umount /var/lib/wakame-vdc/instances/i-ahmv9hlu/rootfs/metadata
    D, [2015-12-09T17:17:54.212985 #4706] DEBUG -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Command Result: fail (exit code=1)
    Command PID: 5314
    ##STDERR=>
    umount: /var/lib/wakame-vdc/instances/i-ahmv9hlu/rootfs/metadata: not found
    E, [2015-12-09T17:17:54.213209 #4706] ERROR -- Lxc: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Dcmgr::Helpers::CliHelper::ShellRunner::CommandError Unexpected exit code=1: umount /var/lib/wakame-vdc/instances/i-ahmv9hlu/rootfs/metadata
    Command PID: 5314
    ##STDERR=>
    umount: /var/lib/wakame-vdc/instances/i-ahmv9hlu/rootfs/metadata: not found from /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/helpers/cli_helper.rb:191:in `block in run!'
    E, [2015-12-09T17:17:54.213340 #4706] ERROR -- HvaHandler: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Instance UUID: i-ahmv9hlu: Ignoring error: Dcmgr::Helpers::CliHelper::ShellRunner::CommandError Unexpected exit code=1: umount /var/lib/wakame-vdc/instances/i-ahmv9hlu/rootfs/metadata
    Command PID: 5314
    ##STDERR=>
    umount: /var/lib/wakame-vdc/instances/i-ahmv9hlu/rootfs/metadata: not found from /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/helpers/cli_helper.rb:191:in `block in run!'
    I, [2015-12-09T17:17:54.220496 #4706]  INFO -- LinuxLocalStore: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Executing command: rm '/var/lib/wakame-vdc/instances/i-ahmv9hlu/vol-fx5ipori'
    D, [2015-12-09T17:17:54.334738 #4706] DEBUG -- LinuxLocalStore: Session ID: 69fba0c3cfb52b57c1fbaf92a0793258ff918ab6: Command Result: success (exit code=0)
    Command PID: 5315
    D, [2015-12-09T17:17:54.345632 #4706] DEBUG -- ServiceNetfilter: event caught: hva.nodeca2c0d6acf66/vnic_destroyed: vif-z5cywgjf
    D, [2015-12-09T17:17:54.345747 #4706] DEBUG -- ServiceNetfilter: vnic not found in cache: vif-z5cywgjf

うん、長いですね。

lxc-create -f /var/lib/wakame-vdc/instances/i-ahmv9hlu/lxc.conf -n i-ahmv9hlu を実行したのに対して、

    A template must be specified.
    Use "none" if you really want a container without a rootfs.

だそうです。いやいや、rootfs 無しなコンテナを作りたいんじゃないんですけどね。で、どうやらメッセージによると template は must be specified らしいので、lxc-create 自体を改造しなきゃならないのかしら〜？って途方に暮れかけてたところ、[「-t /bin/true で試してみろよ」っていう男気溢れる回答があった](http://stackoverflow.com/questions/26442257/how-to-create-a-lxc-container-without-rootfs) ので、[hva の lxc.rb](https://github.com/axsh/wakame-vdc/blob/v16.1.0/dcmgr/lib/dcmgr/drivers/hypervisor/linux_hypervisor/linux_container/lxc.rb#L65) をイジって、無理矢理「-t /bin/true」を付けてみます。

![](/images/wakame-vdc.adventcalendar.2015.1211-01.png )

はい、UI 上は起動したように見えます。では ssh で接続してみましょう。 えーと、結論だけ言うと、ホスト側の環境が変な挙動をするようになってしまいました f^^;

明日はこれらの変更点を LiveDVD に反映させてみます。


