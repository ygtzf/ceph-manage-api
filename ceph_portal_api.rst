Ceph Manage Portal API
#######################

介绍
_______________________
ceph_portal_api为ceph manage portal提供API, 调用了salt_api_client的接口, 对ceph
命令的返回结果做了一些处理, 只保留用户关心的字段.

有以下几个类:

* CephBase
* CephMon
* CephOsd
* CephPool
* CephCrush

其中, CephBase为基类, 其它的类都继承了CephBase.

使用说明
_______________________
1. 在salt_api_setting中配置salt-master的信息.

::

   SALT_MASTER_IP = "14.14.14.196"
   SALT_MASTER_PORT = "8080"
   SALT_MASTER_USER = "ceph"
   SALT_MASTER_PASSWD = "123456"

2. 声明相应对象, 调用相应方法.

::

   举例:
   ceph = CephBase(SALT_MASTER_IP, SALT_MASTER_PORT, SALT_MASTER_USER,
        SALT_MASTER_PASSWD, cluster_name='ceph')

   ret = ceph.fsid('ceph-mon2')

   执行结果:
   {
        'result': True,
        'messages': {
            'comment': 'cmd run ok',
            'fsid': u'd0b9a680-4ea2-4343-a00f-6992eda514e4'
        }
   }

接口说明
_________________________

1. 实例化对象时, 需要如下参数:
 
* SALT_MASTER_IP
* SALT_MASTER_PORT
* SALT_MASTER_USER
* SALT_MASTER_PASSWD
* cluster_name 

2. 所有的方法都有参数minion_id.

3. 所有方法的返回值都是dict, 形式如下:

::

  {
      "result": True,
      "messages": {
          "comment": "cmd run ok",
      }
  }
  注:
  result: salt执行命令返回的结果, True: 命令执行成功, False: 命令执行失败. 
  messages: dict, 具体的返回信息,
     comment: result为True时, 简单的显示cmd run ok; result为False时, 会显示错误的原因.
     如果result为True时, 每个方法有自己的一个key, 包含ceph集群的一些信息, 具体的
     key在接下来的方法说明中详细说明.

CephBase
+++++++++++++++++++++++++
* fsid(minion_id) 

::

  获取ceph集群的fsid.
  返回值: fsid: string 

* health(minion_id)

::

  获取ceph集群的健康状况.
  返回值: health: dict, 包含如下key:
    'serverity': string, 健康级别
    'details': list, ceph具体的健康信息

* status(minion_id)

::

  获取ceph集群的状态.
  返回值: status: dict, 包含如下key:
    'health': dict, 详细的集群信息
    'fsid': string, 
    'monmap': dict, 集群的monitor节点拓扑信息
    'osdmap': dict, 集群的osd节点拓扑信息
    'pgmap': dict, 集群的pg拓扑信息


CephMon
+++++++++++++++++++++++++
* list(minion_id)

::

  列出ceph集群中的monitor节点.
  返回值: mons: list, ceph集群的mon节点列表
    
* details(minion_id)

::

  获取ceph集群中monitor的详细信息.
  返回值: details: dict, 包含如下key:
    'election_epoch': int, mon节点的选举版本
    'quorum_names': list, 参加选举的mon节点列表 
    'quorun_leader_name': string, 选举出的leader
    'monmap': dict, 具体的ceph mon节点拓扑

CephOsd
+++++++++++++++++++++++++
* list(minion_id)

::

  列出ceph集群中的osd节点.
  返回值: osds: list, ceph集群的osd节点列表

* osd_map_details(minion_id)

::

  获取ceph集群中osd map的详细信息.
  返回值: details: list, 集群中的osd详细信息列表, 列表每项为dict, 包含key:
    'name': string, osd名称
    'exists': string, osd是否在集群中
    'status': string, osd状态

* disk_usage(minion_id)

::

  获取ceph集群中的osd的磁盘使用情况.
  返回值: disk_usage: dict, 包含如下key:
    'nodes': list, 每个osd上的磁盘使用情况, 每一项为dict, 包含如下key:
       'name': string, osd名称
       'kb_total': int, osd的总磁盘大小
       'kb_used': int, osd上使用的磁盘大小
       'kb_avail': int, osd上剩余磁盘大小
       'utilization': float, osd上磁盘使用率
    'summary': dict, 整个ceph集群中osd的磁盘使用情况, 包含key:
       'total_kb': int, 所有的osd节点的磁盘空间大小
       'total_kb_used': int, 所有的osd节点的磁盘已使用空间大小
       'total_kb_avail': int, 所有osd节点的剩余空间大小
       'average_utilization': float, 所有osd节点的平均使用率

* latency(minion_id)

::

  获取ceph集群中的osd的延迟情况.
  返回值: osd_latency: list, 每个osd节点的延迟信息, 每一项为dict, 包含key:
    'id': int, osd id
    'commit_latency_ms': int 
    'apply_latency_ms': int

* list_config(minion_id)

::

  列出ceph集群中的osd的相关配置项.
  返回值: configs: dict, 每个key为配置项

* get_config(minion_id, option)

::

  获取ceph集群中的osd的某个配置项的值.
  返回值: value: string, 配置项的值

CephPool
+++++++++++++++++++++++++
* list(minion_id)

::

  列出所有的pool.
  返回值: pools: list, 集群中所有pool的列表 

* details(minion_id)

::

  获取所有的pool的详细详细.
  返回值: pool_details: list, 所有的pool的详细信息,list的每一项为dict, 包含如下
  key:
    'pool_name': string
    'flag_names': string
    'type': int, pool类型
    'size': int, pool备份数
    'min_size': int, pool最小备份数
    'crush_ruleset': int
    'pg_num': int, pool pg数目
    'pgp_num': int, pool pgp数目
    'last_change': string, 最后改变的版本号
    'quota_max_bytes': int, 最大存储bytes配额
    'cache_mode': string
    'target_max_bytes': int
    'target_max_objects': int
    'erasure_code_profile': stirng
    'min_read_recency_for_promote': int
    'string_sidth': int 

* get_replica_size(minion_id, poolname)

::

  获取某个pool的备份数.
  返回值: value: string

* get_replica_min_size(minion_id, poolname)

::

  获取某个pool的最小备份数.
  返回值: value: string

* get_pg_num(minion_id, poolname)

::

  获取某个pool的pg数量.
  返回值: value: string

* get_pgp_num(minion_id, poolname)

::

  获取某个pool的pgp数目.
  返回值: value: string

* get_erasure_profile(minion_id, poolname)

::

  获取纠删码的配置文件.
  返回值: erasure_profile: dict, 包含如下key:
    'erasure_code_profile': string

* get_quota(minion_id, poolname)

::

  获取某个pool的配额.
  返回值: quota: dict, 包含如下key:
    'quota_max_objects': int, pool的最多objects限制
    'quota_max_bytes': int, pool的最大bytes限制

* pg_stat(minion_id)

::

  获取pg的状态.
  返回值: pg_stats: dict, 包含如下key:
    'num_pg_by_state': list, 处于某种状态的pg数 
    'version': int, pg
    'num_pgs': int, pg总数, 不包括备份pg
    'num_bytes': int, 当前pg中byte大小
    'raw_bytes_used': int
    'raw_bytes_avail': int 
    'raw_bytes': int

* pg_dump_stuck(minion_id, stat)

::

  获取处于某种状态的pg.
  返回值: pgs: list, 处于某种状态的所有pg列表

* pg_distribution(minion_id)

::

  获取pg在整个ceph集群中的分布详细信息.
  返回值: pgs: list, 每个list项为dict, 包含如下key:
    'osd': dict, 每个osd中的pg数
    'osd_total_pgs': int
    'pool_id': string

CephCrush
+++++++++++++++++++++++++
* show_crush_map(minion_id)

::

  获取ceph crush map.
  返回值: crush: list, 所有root列表, list项为dict, 包含以下key:
    'root': string, root名称
    'weight': float, 整个root权重
    'hosts': list, 所有host的列表, 每一项为dict, 包含key:
       'host': string
       'weight': float
       'osds': list, 该host上的所有osd的crush信息

* show_crush_rules(minion_id)

::

  显示ceph crush的规则.
  返回值: rules: list, crush rule列表, 列表每一项为dict, 包含以下key:
    'rule_id': int, crush规则id
    'rule_name': string, crush规则名称
    'ruleset': int
    'type': int, crush规则类型
    'min_size': int
    'max_size': int
    'steps': list, 规则执行的步骤, 每一项为list 

