---
title: "Wakame-vdc と LXC と LXC の恋"
date: 2015-12-07 05:19:32 +0900
tags: ["Wakame-vdc","AdventCalendar"]
---

Wakame-vdc ユーザの皆さん、磯野家の皆さん、こんにちは。これはオレオレ WakameVdc / OpenVNet Advent Calendar 2015 の7日目のエントリです。

今日は LiveDVD から一旦離れて「そもそも普通の Wakame-vdc で hva の利用するハイパーバイザを lxc として動かすには？」をやってみます。

<!--more-->

参考になる情報として、[@giraffeforestg](https://twitter.com/giraffeforestg) さんが書いた [Wakame-vdc / OpenVNet Advent Calendar 2014](http://www.adventar.org/calendars/577) の [12/21 分のエントリ](http://giraffeforestg.blog.fc2.com/blog-category-29.html) があるので、ますそれを一読します。

で、インストール手順自体は Wakame-vdc の[インストールガイド](http://wakame-vdc.org/installation/) を参考に進めてみます。

[@giraffeforestg](https://twitter.com/giraffeforestg) さんのエントリですと、Wakame-vdc のデモイメージでは lxc のバージョンは CentOS 側の最新ではなく、0.8.0 を使用しているようですが、とりあえず CentOS 6.7 が提供する 1.0.8 で進めてみます。

で、手抜き描写ですみませんがセットアップは以下。

    sudo curl -o /etc/yum.repos.d/wakame-vdc-stable.repo -R https://raw.githubusercontent.com/axsh/wakame-vdc/master/rpmbuild/yum_repositories/wakame-vdc-stable.repo
    sudo yum install -y epel-release
    sudo yum install -y wakame-vdc-dcmgr-vmapp-config
    sudo yum install -y wakame-vdc-hva-lxc-vmapp-config
    sudo yum install -y wakame-vdc-webui-vmapp-config
    cp /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-br0

ifcfg-br0 の記述は以下

    DEVICE=br0
    TYPE=Bridge
    ONBOOT=yes
    NM_CONTROLLED=no
    BOOTPROTO=dhcp
    DHCPV6C=no 
    IPV6INIT=no 
    IPV6_AUTOCONF=no

ifcfg-eth0 の記述は以下

    DEVICE=eth0
    ONBOOT=yes
    BRIDGE=br0
    NM_CONTROLLED=no

で、引き続きセットアップ

    sudo service network restart
    wget http://wakame-vdc.org/scripts/install_guide_demo_data.sh
    sed -i -e "s/openvz/lxc/g" install_guide_demo_data.sh
    chmod +x install_guide_demo_data.sh 
    NETWORK='10.0.0.0' PREFIX='8' DHCP_RANGE_START='10.0.0.100' DHCP_RANGE_END='10.0.0.200' ./install_guide_demo_data.sh
    sudo service rabbitmq-server start
    sudo service mysqld start
    sudo start vdc-dcmgr
    sudo start vdc-collector
    sudo start vdc-hva
    sudo start vdc-webui

で、このあとブラウザで WebUI に接続し、セキュリティグループとキーペアを設定し、install_guide_demo_data.sh で登録された Ubuntu のイメージを指定して起動してみます。

![](/images/wakame-vdc.adventcalendar.2015.1207-01.png )

はい、起動に失敗したようです。
「そもそもホストが CentOS なのに何で Ubuntu イメージ(それも OpenVZ なデモイメージ用の…)を使うんだよ？もしかしてエラー出るようにワザとやってない？」ってツッコまれそうですが…そ、そんなことないです…たぶん、もしそうだとしても天然です。

で、じゃぁ、hva のログは…というと以下な感じ。

    2015-12-07 05:52:38 JobContext thr=LocalStore[0/2] [INFO]: Job start afe0d78dacb3bafa8b8f8141dffed72510d61fa6 (Local ID: 450b54eeb176f67426defc0407d91c1177e6da51)[ run_local_store ]
    I, [2015-12-07T05:52:38.554305 #7987]  INFO -- HvaHandler: Session ID: afe0d78dacb3bafa8b8f8141dffed72510d61fa6: Instance UUID: i-7ujh4q24: Booting i-7ujh4q24: phase 1
    I, [2015-12-07T05:52:38.620281 #7987]  INFO -- HvaHandler: Session ID: afe0d78dacb3bafa8b8f8141dffed72510d61fa6: Instance UUID: i-7ujh4q24: Creating volume vol-atfog2nw from bo-ubuntu14043ple.
    I, [2015-12-07T05:52:38.687657 #7987]  INFO -- LinuxLocalStore: Session ID: afe0d78dacb3bafa8b8f8141dffed72510d61fa6: Deploying image: wmi-ubuntu14043ple from file:///var/lib/wakame-vdc/images/ubuntu-14.04.3-x86_64-30g-passwd-login-enabled.raw.tgz to /var/lib/wakame-vdc/instances/i-7ujh4q24/vol-atfog2nw
    I, [2015-12-07T05:52:38.688701 #7987]  INFO -- LinuxLocalStore: Session ID: afe0d78dacb3bafa8b8f8141dffed72510d61fa6: Executing command: cat /var/lib/wakame-vdc/images/ubuntu-14.04.3-x86_64-30g-passwd-login-enabled.raw.tgz| pv -W -f -p -s 312530432 | tar -zxS -C /var/lib/wakame-vdc/instances/i-7ujh4q24/d20151207-7987-999etl
    D, [2015-12-07T05:52:48.575228 #7987] DEBUG -- LinuxLocalStore: Session ID: afe0d78dacb3bafa8b8f8141dffed72510d61fa6: Command Result: success (exit code=0)
    Command PID: 8168
    ##STDERR=>
    [=======================================================================>] 100%
    2015-12-07 05:52:48 JobContext thr=LocalStore[0/2] [INFO]: Job complete afe0d78dacb3bafa8b8f8141dffed72510d61fa6 (Local ID: 450b54eeb176f67426defc0407d91c1177e6da51)[ run_local_store ]: 10.0355809 sec
    2015-12-07 05:52:48 JobContext thr=JobWorker[0/1] [INFO]: Job start afe0d78dacb3bafa8b8f8141dffed72510d61fa6 (Local ID: 24d2bbd8bc0913354622224e4e179ead2be00f9d)[ run_local_store ]
    I, [2015-12-07T05:52:48.593029 #7987]  INFO -- HvaHandler: Session ID: afe0d78dacb3bafa8b8f8141dffed72510d61fa6: Instance UUID: i-7ujh4q24: Booting instance
    I, [2015-12-07T05:52:48.715200 #7987]  INFO -- Lxc: Session ID: afe0d78dacb3bafa8b8f8141dffed72510d61fa6: lxc-version: 1.0.8
    2015-12-07 05:52:48 JobContext thr=JobWorker[0/1] [ERROR]: Job failed afe0d78dacb3bafa8b8f8141dffed72510d61fa6 [ run_local_store ]: cgroup filesystem is not mounted.
    2015-12-07 05:52:48 JobContext thr=JobWorker[0/1] [ERROR]: Caught RuntimeError: cgroup filesystem is not mounted.
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
    I, [2015-12-07T05:52:48.716150 #7987]  INFO -- HvaHandler: Session ID: afe0d78dacb3bafa8b8f8141dffed72510d61fa6: Instance UUID: i-7ujh4q24: teminating instance
    I, [2015-12-07T05:52:48.723362 #7987]  INFO -- Lxc: Session ID: afe0d78dacb3bafa8b8f8141dffed72510d61fa6: lxc-version: 1.0.8
    E, [2015-12-07T05:52:48.724572 #7987] ERROR -- HvaHandler: Session ID: afe0d78dacb3bafa8b8f8141dffed72510d61fa6: Instance UUID: i-7ujh4q24: Ignoring error: RuntimeError cgroup filesystem is not mounted. from /opt/axsh/wakame-vdc/dcmgr/lib/dcmgr/drivers/hypervisor/linux_hypervisor/linux_container/lxc.rb:54:in `initialize'
    I, [2015-12-07T05:52:48.731985 #7987]  INFO -- LinuxLocalStore: Session ID: afe0d78dacb3bafa8b8f8141dffed72510d61fa6: Executing command: rm '/var/lib/wakame-vdc/instances/i-7ujh4q24/vol-atfog2nw'
    D, [2015-12-07T05:52:48.885889 #7987] DEBUG -- LinuxLocalStore: Session ID: afe0d78dacb3bafa8b8f8141dffed72510d61fa6: Command Result: success (exit code=0)
    Command PID: 8182
    D, [2015-12-07T05:52:48.900115 #7987] DEBUG -- ServiceNetfilter: event caught: hva.demo1/vnic_destroyed: vif-jumhvsv6
    D, [2015-12-07T05:52:48.900236 #7987] DEBUG -- ServiceNetfilter: vnic not found in cache: vif-jumhvsv6

ってことで明日はメーリングリストで教えてもらった CentOS のイメージを使って試してみます。きっと同じようなエラーが出ると信じたい。


