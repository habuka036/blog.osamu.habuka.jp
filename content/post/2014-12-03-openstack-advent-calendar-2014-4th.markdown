---
title: "OpenStack Advent Calendar 2014 4日目"
date: 2014-12-04 08:08:08 +0900
tags: ["OpenStack"]
---
これは [OpenStack Advent Calendar 2014](http://www.adventar.org/calendars/569) の4日目のエントリです。昨日は日本 Eucalyptus ユーザ会のエントリ [OpenStack on Eucalyptus](http://eucalyptus-users.jp/blog/2014/12/03/openstack-on-eucalyptus-slash-openstack-advent-calendar-2014-dot-12-dot-03/) でした。

今日は題して「Eucalyptus ユーザが解説する Nova API 改造指南」です。例年ですと私の OpenStack ネタのエントリは失敗するのがお約束ですが、今日も無事失敗できるかどうかドキドキです。

<!--more-->

で、このエントリの目標は「API にパラメータを何か追加してみて、ちゃんとそれが処理されるか否か」という緩い目標設定となっております。(あまりマトモな目標設定をすると失敗する以前にまず何もできなかったりするので…」

出力される項目を増やす
----------------------

まぁ、誰もがやってみる改造ですね。例えば nova list の出力にはそのインスタンスが「どの nova-compute で起動しているか？」なんて情報は含まれていません。知りたい場合は DB を覗かないと知れません(ホント？)、これはとても面倒臭い…ということで、nova list の結果にその情報が含まれるようにしちゃいましょう。

    $ nova list
    +--------------------------------------+------+--------+------------+-------------+------------------+
    | ID                                   | Name | Status | Task State | Power State | Networks         |
    +--------------------------------------+------+--------+------------+-------------+------------------+
    | 98f7c56e-310f-4e57-9c9a-c0c5aa48cf08 | vm01 | ACTIVE | -          | Running     | private=10.0.0.2 |
    +--------------------------------------+------+--------+------------+-------------+------------------+

まず、DB「nova」のテーブル「instances」を確認すると、以下のフィールドがあります。

    +--------------------------+-----------------------+------+-----+---------+----------------+
    | Field                    | Type                  | Null | Key | Default | Extra          |
    +--------------------------+-----------------------+------+-----+---------+----------------+
    | created_at               | datetime              | YES  |     | NULL    |                |
    | updated_at               | datetime              | YES  |     | NULL    |                |
    | deleted_at               | datetime              | YES  |     | NULL    |                |
    | id                       | int(11)               | NO   | PRI | NULL    | auto_increment |
    | internal_id              | int(11)               | YES  |     | NULL    |                |
    | user_id                  | varchar(255)          | YES  |     | NULL    |                |
    | project_id               | varchar(255)          | YES  | MUL | NULL    |                |
    | image_ref                | varchar(255)          | YES  |     | NULL    |                |
    | kernel_id                | varchar(255)          | YES  |     | NULL    |                |
    | ramdisk_id               | varchar(255)          | YES  |     | NULL    |                |
    | launch_index             | int(11)               | YES  |     | NULL    |                |
    | key_name                 | varchar(255)          | YES  |     | NULL    |                |
    | key_data                 | mediumtext            | YES  |     | NULL    |                |
    | power_state              | int(11)               | YES  |     | NULL    |                |
    | vm_state                 | varchar(255)          | YES  |     | NULL    |                |
    | memory_mb                | int(11)               | YES  |     | NULL    |                |
    | vcpus                    | int(11)               | YES  |     | NULL    |                |
    | hostname                 | varchar(255)          | YES  |     | NULL    |                |
    | host                     | varchar(255)          | YES  | MUL | NULL    |                |
    | user_data                | mediumtext            | YES  |     | NULL    |                |
    | reservation_id           | varchar(255)          | YES  | MUL | NULL    |                |
    | scheduled_at             | datetime              | YES  |     | NULL    |                |
    | launched_at              | datetime              | YES  |     | NULL    |                |
    | terminated_at            | datetime              | YES  | MUL | NULL    |                |
    | display_name             | varchar(255)          | YES  |     | NULL    |                |
    | display_description      | varchar(255)          | YES  |     | NULL    |                |
    | availability_zone        | varchar(255)          | YES  |     | NULL    |                |
    | locked                   | tinyint(1)            | YES  |     | NULL    |                |
    | os_type                  | varchar(255)          | YES  |     | NULL    |                |
    | launched_on              | mediumtext            | YES  |     | NULL    |                |
    | instance_type_id         | int(11)               | YES  |     | NULL    |                |
    | vm_mode                  | varchar(255)          | YES  |     | NULL    |                |
    | uuid                     | varchar(36)           | YES  | UNI | NULL    |                |
    | architecture             | varchar(255)          | YES  |     | NULL    |                |
    | root_device_name         | varchar(255)          | YES  |     | NULL    |                |
    | access_ip_v4             | varchar(39)           | YES  |     | NULL    |                |
    | access_ip_v6             | varchar(39)           | YES  |     | NULL    |                |
    | config_drive             | varchar(255)          | YES  |     | NULL    |                |
    | task_state               | varchar(255)          | YES  | MUL | NULL    |                |
    | default_ephemeral_device | varchar(255)          | YES  |     | NULL    |                |
    | default_swap_device      | varchar(255)          | YES  |     | NULL    |                |
    | progress                 | int(11)               | YES  |     | NULL    |                |
    | auto_disk_config         | tinyint(1)            | YES  |     | NULL    |                |
    | shutdown_terminate       | tinyint(1)            | YES  |     | NULL    |                |
    | disable_terminate        | tinyint(1)            | YES  |     | NULL    |                |
    | root_gb                  | int(11)               | YES  |     | NULL    |                |
    | ephemeral_gb             | int(11)               | YES  |     | NULL    |                |
    | cell_name                | varchar(255)          | YES  |     | NULL    |                |
    | node                     | varchar(255)          | YES  |     | NULL    |                |
    | deleted                  | int(11)               | YES  |     | NULL    |                |
    | locked_by                | enum('owner','admin') | YES  |     | NULL    |                |
    | cleaned                  | int(11)               | YES  |     | NULL    |                |
    | ephemeral_key_uuid       | varchar(36)           | YES  |     | NULL    |                |
    +--------------------------+-----------------------+------+-----+---------+----------------+

で、これらのフィールドのなかで、host と node がそれっぽいのですが、それらのどちらが nova-compute のホスト名なのかは、にわかユーザにはわからないので、どちらも表示させることにします。

……

えっと…まるで初心者のような──いや、僕は確かに初心者なので間違ってはいないのですが、不思議なことが起きました。何かと言うと、/opt/stack/nova が消えました。とりあえずコマンド履歴を途中から確認してみました。

    ubuntu@ubuntu:/opt/stack/nova$ vim nova/db/sqlalchemy/api.py
    ubuntu@ubuntu:/opt/stack/nova$ vim nova/api/openstack/compute/servers.py
    ubuntu@ubuntu:/opt/stack/nova$ vim nova/compute/api.py
    ubuntu@ubuntu:/opt/stack/nova$ vim /usr/local/lib/python2.7/dist-packages/py[TAB]
    ubuntu@ubuntu:/opt/stack/nova$ vim /usr/local/lib/python2.7/dist-packages/py[TAB]
    ubuntu@ubuntu:/opt/stack/nova$ vim /usr/local/lib/python2.7/dist-packages/py[TAB]
    ubuntu@ubuntu:/opt/stack/nova$ vim /usr/local/lib/python2.7/dist-packages/novaclient/[TAB]
    ubuntu@ubuntu:/opt/stack/nova$ vim /usr/local/lib/python2.7/dist-packages/novaclient/v1_1/[TAB]
    ubuntu@ubuntu:/opt/stack/nova$ vim /usr/local/lib/python2.7/dist-packages/novaclient/v1_1/[TAB]
    ubuntu@ubuntu:/opt/stack/nova$ vim /usr/local/lib/python2.7/dist-packages/novaclient/v1_1/[TAB]
    ubuntu@ubuntu:/opt/stack/nova$ vim /usr/local/lib/python2.7/dist-packages/novaclient/v1_1/[TAB]
    ubuntu@ubuntu:/opt/stack/nova$ vim /usr/local/lib/python2.7/dist-packages/novaclient/v1_1/se[TAB]
    ubuntu@ubuntu:/opt/stack/nova$ vim /usr/local/lib/python2.7/dist-packages/novaclient/v1_1/serv[TAB]
    ubuntu@ubuntu:/opt/stack/nova$ vim /usr/local/lib/python2.7/dist-packages/novaclient/v1_1/server[TAB]
    ubuntu@ubuntu:/opt/stack/nova$ vim /usr/local/lib/python2.7/dist-packages/novaclient/v1_1/servers.py
    ubuntu@ubuntu:/opt/stack/nova$ vim /usr/local/lib/python2.7/dist-packages/novaclient/v1_1/servers.py
    ubuntu@ubuntu:/opt/stack/nova$ vim nova/db/sqlalchemy/api.py
    ubuntu@ubuntu:/opt/stack/nova$ vim nova/db/sqlalchemy/api.py
    ubuntu@ubuntu:/opt/stack/nova$ pwd
    /opt/stack/nova
    ubuntu@ubuntu:/opt/stack/nova$ ls -al
    total 0
    ubuntu@ubuntu:/opt/stack/nova$ df -h
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/vda1        16G  4.8G   11G  33% /
    none            4.0K     0  4.0K   0% /sys/fs/cgroup
    udev            2.0G  4.0K  2.0G   1% /dev
    tmpfs           396M  432K  395M   1% /run
    none            5.0M     0  5.0M   0% /run/lock
    none            2.0G     0  2.0G   0% /run/shm
    none            100M     0  100M   0% /run/user
    ubuntu@ubuntu:/opt/stack/nova$ ls -al /opt/
    total 12
    drwxr-xr-x  3 root   root 4096 Dec  4 15:10 .
    drwxr-xr-x 22 root   root 4096 Jul  6 11:45 ..
    drwxr-xr-x 14 ubuntu root 4096 Dec  4 19:10 stack
    ubuntu@ubuntu:/opt/stack/nova$ ls -al /opt/stack/
    total 56
    drwxr-xr-x 14 ubuntu root   4096 Dec  4 19:10 .
    drwxr-xr-x  3 root   root   4096 Dec  4 15:10 ..
    drwxr-xr-x 10 ubuntu ubuntu 4096 Dec  4 17:18 cinder
    drwxr-xr-x  6 ubuntu root   4096 Dec  4 17:32 data
    drwxr-xr-x  9 ubuntu ubuntu 4096 Dec  4 17:18 glance
    drwxr-xr-x 12 ubuntu ubuntu 4096 Dec  4 17:23 heat
    drwxr-xr-x  7 ubuntu ubuntu 4096 Dec  4 17:22 heat-cfntools
    drwxr-xr-x 10 ubuntu ubuntu 4096 Dec  4 17:22 heat-templates
    drwxr-xr-x 11 ubuntu ubuntu 4096 Dec  4 17:24 horizon
    drwxr-xr-x 12 ubuntu ubuntu 4096 Dec  4 17:18 keystone
    drwxr-xr-x  9 ubuntu ubuntu 4096 Dec  4 16:40 noVNC
    drwxr-xr-x  5 ubuntu ubuntu 4096 Dec  4 16:16 requirements
    drwxr-xr-x  3 ubuntu ubuntu 4096 Dec  4 17:23 status
    drwxr-xr-x  8 ubuntu ubuntu 4096 Dec  4 17:32 tempest

いやぁ〜、いくら僕が OpenStack 初心者だからと言って、こんな現象に遭遇するとは…。一体どんな操作ミスをしたらこんなことになるんでしょう…不思議です。

まぁ、でも消えちゃったものは仕方ないので、./unstack.sh 叩いてもう一度 ./stack.sh を叩くことにします。

で、気をとりなおして再挑戦…と思って調査したんですが、[api/openstack/compute/servers.py のここ](https://github.com/openstack/nova/blob/master/nova/api/openstack/compute/servers.py#L604-L610) でインスタンスのリストを取得している風というのは分かるんですが、この結果を nova list 用に整形している [この部分](https://github.com/openstack/nova/blob/master/nova/api/openstack/compute/servers.py#L622) から先の[ここあたり](https://github.com/openstack/nova/blob/master/nova/api/openstack/compute/views/servers.py#L117-L120)で追うのに行き詰まりました。

ここら辺が非 OpenStacker の限界ということでしょうか…orz

ちなみに nova list ではなく、nova baremetal-node-list の出力項目を変更したい場合は [この node_fields の要素をイジって](https://github.com/openstack/nova/blob/master/nova/api/openstack/compute/contrib/baremetal_nodes.py#L32-L33)、更に novaclient の [v1_1/contrib/baremetal.py の _print_baremetal_nodes_list のここ](https://github.com/openstack/python-novaclient/blob/master/novaclient/v1_1/contrib/baremetal.py#L247-L259)をイジれば変更できます…が、たぶんそんなこと OpenStacker の皆さんなら周知の事実ですよね…すみません…orz

----
そのあと元木さんから「それ、改造しなくても admin なら --fields オプションで OS-EXT-SRV-ATTR: で引けるよ？」と教えてもらいましたwwww (同じく真壁さんからもw)

    $ nova list --all-tenants --fields name,networks,OS-EXT-SRV-ATTR:host,OS-EXT-SRV-ATTR:node
    +--------------------------------------+------+------------------+-----------------------+-----------------------+
    | ID                                   | Name | Networks         | OS-EXT-SRV-ATTR: Host | OS-EXT-SRV-ATTR: Node |
    +--------------------------------------+------+------------------+-----------------------+-----------------------+
    | 9bc0c1fb-d55c-481f-84fb-fadd21ec1d35 | vm01 | private=10.0.0.2 | ubuntu                |                       |
    +--------------------------------------+------+------------------+-----------------------+-----------------------+

novaclient のサブコマンドに引数を持たないオプションを追加する方法
-----------------------------------------------------------------

例えば、nova baremetal-node-delete のオプションに引数を持たない --force というオプションを追加したい場合は [v1_1/contrib/baremetal.py](https://github.com/openstack/python-novaclient/blob/master/novaclient/v1_1/contrib/baremetal.py) の以下を

```python
@utils.arg(
    'node',
    metavar='<node>',
    help=_('ID of the node to delete.'))
def do_baremetal_node_delete(cs, args):
    """Remove a baremetal node and any associated interfaces."""
    node = _find_baremetal_node(cs, args.node)
    cs.baremetal.delete(node)
```

以下のように変更するだけです。

```python
@utils.arg(
    'node',
    metavar='<node>',
    help=_('ID of the node to delete.'))
@utils.arg('--force',
    action="store_true",
    default=False,
    help='force delete baremetal machine.')
def do_baremetal_node_delete(cs, args):
    """Remove a baremetal node and any associated interfaces."""
    node = _find_baremetal_node(cs, args.node)
    cs.baremetal.delete(node)
```

あとは do_baremetal_node_delete の中でオプションの値を参照する場合は args.force を見れば OK です。


----
明日の [OpenStack Advent Calendar 2014 5日目](http://www.adventar.org/calendars/569) は Shogos3 さんのエントリです。
