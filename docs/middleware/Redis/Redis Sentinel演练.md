# [Redis Sentinel(哨兵模式)](https://redis.io/topics/sentinel)

### 1. 背景

Redis 作为一款主流的 NoSQL 存储系统，在生产环境中被广泛使用，常用作于分布式锁、缓存、限流器、队列等，其重要性不言而喻。
Redis Sentinel 为 Redis 提供了高可用性，这意味着 Sentinel 在没有人工干预的情况下能为 Redis 抵抗某些类型的失败（如系统宕机、网络波动）

### 2. 部署

#### 2.1. 查看 OS, Redis 版本

```sh
[root@vm1 ~]# uname -a
Linux vm1.yejh 2.6.32-754.el6.x86_64 #1 SMP Tue Jun 19 21:26:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
[root@vm1 ~]# cat /etc/issue
CentOS release 6.10 (Final)
Kernel \r on an \m

[root@vm1 ~]# redis-cli --version
redis-cli 5.0.7
```

#### 2.2. 修改 Redis Server 配置

***/usr/redis/home/redis.conf***

```properties
port 6379
pidfile /var/run/redis_6379.pid
# master, replicas 需配置
masterauth redis20170419
# replicas 需配置
replicaof vm1.yejh.cn 6379
logfile "/usr/redis/home/logs/redis_6379.log"
dbfilename dump_6379.rdb
```

#### 2.3. 新增 Redis Sentinel 配置 (:%s/26379/2638x/gc)

***/usr/redis/home/sentinel_26379.conf***

```properties
protected-mode no
port 26379
# 守护线程，后台运行
daemonize yes
pidfile /var/run/redis-sentinel_26379.pid
# 日志目录需提前创建
logfile "/usr/redis/home/logs/sentinel_26379.log"
dir /tmp
# sentinel monitor <master-group-name> <ip> <port> <quorum>
sentinel monitor mymaster vm1.yejh.cn 6379 2
# sentinel <option_name> <master_name> <option_value>
sentinel auth-pass mymaster redis20170419
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
```

#### 2.4. 启动服务

- 先启动 master，再启动 replicas，最后启动 sentinels

```sh
[root@vm1 ~]# redis-server /usr/redis/home/redis.conf
[root@vm2 ~]# redis-server /usr/redis/home/redis.conf
[root@vm1 ~]# redis-sentinel /usr/redis/home/sentinel_26379.conf
[root@vm2 ~]# redis-sentinel /usr/redis/home/sentinel_26379.conf
```

### 3. Sentinel 模式下 HA 测试演练

**为简化表述，各组件采用以下简称**

- master -> M
- replica -> R
- sentinel -> S
- virtual machine -> VM

| 主机  | 角色            | 地址            | 端口                        |
|-----|---------------|---------------|---------------------------|
| VM1 | M, S1, S2, S3 | 192.168.2.114 | 6379, 26379, 26380, 26381 |
| VM2 | R, S1, S2     | 192.168.2.113 | 6379, 26379, 26380        |

***

#### 场景一：1M1S + 1R1S (quorum = 1)

```sh
VM1:6379> INFO Replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.2.113,port=6379,state=online,offset=9455,lag=0
master_replid:c446e8343067554ee2ff7fab3c471b133ed0f124
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:9596
second_repl_offset:-1

VM2:6379> INFO Replication
# Replication
role:slave
master_host:vm1.yejh.cn
master_port:6379
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:16998
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:c446e8343067554ee2ff7fab3c471b133ed0f124
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:16998

VM1:26379> INFO Sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.2.114:6379,slaves=1,sentinels=2

VM2:26379> INFO Sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.2.114:6379,slaves=1,sentinels=2
```

- M 下线

能实现故障转移，VM2 中的 R 升级为 M，重启 VM1 宕机的 M 后会作为新的 R 备属

***[VM2]/usr/redis/home/logs/sentinel_26379.log***

```log
1630:X 30 Dec 2019 23:40:48.571 # +sdown master mymaster 192.168.2.114 6379
1630:X 30 Dec 2019 23:40:48.571 # +odown master mymaster 192.168.2.114 6379 #quorum 1/1
1630:X 30 Dec 2019 23:40:49.510 # +switch-master mymaster 192.168.2.114 6379 192.168.2.113 6379
1630:X 30 Dec 2019 23:40:49.510 * +slave slave 192.168.2.114:6379 192.168.2.114 6379 @ mymaster 192.168.2.113 6379
1630:X 30 Dec 2019 23:41:19.572 # +sdown slave 192.168.2.114:6379 192.168.2.114 6379 @ mymaster 192.168.2.113 6379
1630:X 30 Dec 2019 23:46:44.552 # -sdown slave 192.168.2.114:6379 192.168.2.114 6379 @ mymaster 192.168.2.113 6379
```

- VM1 下线

故障转移失败，Redis 服务不可用

***[VM2]/usr/redis/home/logs/sentinel_26379.log***

```log
1600:X 01 Jan 2020 21:15:48.247 # +sdown master mymaster 192.168.2.114 6379
1600:X 01 Jan 2020 21:15:48.247 # +odown master mymaster 192.168.2.114 6379 #quorum 1/1
1600:X 01 Jan 2020 21:15:48.247 # +new-epoch 1
1600:X 01 Jan 2020 21:15:48.247 # +try-failover master mymaster 192.168.2.114 6379
1600:X 01 Jan 2020 21:15:48.273 # +vote-for-leader c6bff605b803d240b7393050a7d28b124436443d 1
1600:X 01 Jan 2020 21:15:48.274 # +sdown sentinel 57bb3ca9cdd37ac545a191e28b545807eb5c7648 192.168.2.114 26379 @ mymaster 192.168.2.114 6379
1600:X 01 Jan 2020 21:15:58.906 # -failover-abort-not-elected master mymaster 192.168.2.114 6379
1600:X 01 Jan 2020 21:15:58.997 # Next failover delay: I will not start a failover before Wed Jan  1 21:21:48 2020
```

- R 下线

记录宕机的 R

***[VM1]/usr/redis/home/logs/sentinel_26379.log***

```log
1979:X 01 Jan 2020 20:59:03.574 # +sdown slave 192.168.2.113:6379 192.168.2.113 6379 @ mymaster 192.168.2.114 63799
```

- VM2 下线

记录宕机的 R, S

***[VM1]/usr/redis/home/logs/sentinel_26379.log***

```log
2082:X 01 Jan 2020 21:09:30.158 # +sdown slave 192.168.2.113:6379 192.168.2.113 6379 @ mymaster 192.168.2.114 6379
2082:X 01 Jan 2020 21:09:30.158 # +sdown sentinel 7977a971849c0da46427c0c4302b35483d77fbb5 192.168.2.113 26379 @ mymaster 192.168.2.114 6379
```

***

#### 场景二：1M1S + 1R2S (quorum = 2)

- M 下线

能实现故障转移

- VM1 下线

能实现故障转移

***

#### 场景三：1M3S + 1R2S (quorum = 2)

- M 下线

能实现故障转移

- VM1 下线

故障转移失败，Redis 服务不可用

***

#### 场景四：1M3S + 1R2S (quorum = 5)

- M 下线

能实现故障转移

***[VM1]/usr/redis/home/logs/sentinel_26381.log***

```log
2057:X 01 Jan 2020 23:10:13.241 # +sdown master mymaster 192.168.2.114 6379
2057:X 01 Jan 2020 23:10:13.303 # +odown master mymaster 192.168.2.114 6379 #quorum 5/5
2057:X 01 Jan 2020 23:10:13.303 # +new-epoch 1
2057:X 01 Jan 2020 23:10:13.303 # +try-failover master mymaster 192.168.2.114 6379
2057:X 01 Jan 2020 23:10:13.380 # +vote-for-leader f81f260fc14759a7879ea00481facf401c4665c9 1
2057:X 01 Jan 2020 23:10:13.387 # 3439e4c70f5792361c58d7ed0e48467a814ab9ae voted for f81f260fc14759a7879ea00481facf401c4665c9 1
2057:X 01 Jan 2020 23:10:13.388 # c5f6b92a8760f7c0133a4fb6f3122d5c50bad5f1 voted for f81f260fc14759a7879ea00481facf401c4665c9 1
2057:X 01 Jan 2020 23:10:13.391 # b21bac7eb0fa59dc601b4b4b68935a70cc76cdba voted for f81f260fc14759a7879ea00481facf401c4665c9 1
2057:X 01 Jan 2020 23:10:13.391 # 9af74775e55f259b7bb67c8dda8e8acffcc68513 voted for f81f260fc14759a7879ea00481facf401c4665c9 1
2057:X 01 Jan 2020 23:10:13.465 # +elected-leader master mymaster 192.168.2.114 6379
2057:X 01 Jan 2020 23:10:13.465 # +failover-state-select-slave master mymaster 192.168.2.114 6379
2057:X 01 Jan 2020 23:10:13.524 # +selected-slave slave 192.168.2.113:6379 192.168.2.113 6379 @ mymaster 192.168.2.114 6379
2057:X 01 Jan 2020 23:10:13.524 * +failover-state-send-slaveof-noone slave 192.168.2.113:6379 192.168.2.113 6379 @ mymaster 192.168.2.114 6379
2057:X 01 Jan 2020 23:10:13.600 * +failover-state-wait-promotion slave 192.168.2.113:6379 192.168.2.113 6379 @ mymaster 192.168.2.114 6379
2057:X 01 Jan 2020 23:10:13.956 # +promoted-slave slave 192.168.2.113:6379 192.168.2.113 6379 @ mymaster 192.168.2.114 6379
2057:X 01 Jan 2020 23:10:13.956 # +failover-state-reconf-slaves master mymaster 192.168.2.114 6379
2057:X 01 Jan 2020 23:10:14.051 # +failover-end master mymaster 192.168.2.114 6379
2057:X 01 Jan 2020 23:10:14.051 # +switch-master mymaster 192.168.2.114 6379 192.168.2.113 6379
2057:X 01 Jan 2020 23:10:14.051 * +slave slave 192.168.2.114:6379 192.168.2.114 6379 @ mymaster 192.168.2.113 6379
2057:X 01 Jan 2020 23:10:44.119 # +sdown slave 192.168.2.114:6379 192.168.2.114 6379 @ mymaster 192.168.2.113 6379
```

- VM1 下线

未开启故障转移，Redis 服务不可用

***[VM2]/usr/redis/home/logs/sentinel_26380.log***

```log
1915:X 01 Jan 2020 23:27:52.679 # +sdown master mymaster 192.168.2.114 6379
1915:X 01 Jan 2020 23:27:52.679 # +sdown sentinel f6ccdaa03fbb74b7556497ab0a77e4f1cc9bfe01 192.168.2.114 26379 @ mymaster 192.168.2.114 6379
1915:X 01 Jan 2020 23:27:52.679 # +sdown sentinel 634f1fe23b79f5a8be590b279f3df871fd3ba7da 192.168.2.114 26381 @ mymaster 192.168.2.114 6379
1915:X 01 Jan 2020 23:27:52.679 # +sdown sentinel 9551fde004447f571e4471b9427bf2c44fc6600a 192.168.2.114 26380 @ mymaster 192.168.2.114 6379
```

***

### 4. 演练分析

从上述场景可以看出，一次成功的哨兵模式主备切换分为两个阶段：

1. sentinel 对 master 不可达{@see down-after-milliseconds}会对其标识 "+sdown"，sentinel 相互及时交换信息；当 "+sdown" 次数达到
   **quorum** 值时，标识 master 为 "+odown", 尝试故障转移(failover)
2. sentinel 之间通过投票选举出 `leader`，当 `leader` 票数达到 MAX(“quorum 值”, “过半数 sentinel 实例数”)
   时，开启故障转移（master-replica切换，通知各成员修改配置等）

**关于 [quorum](https://zh.wikipedia.org/wiki/Quorum_(%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F)) 值，Redis
官网有以下说明**

> - The quorum is the number of Sentinels that need to agree about the fact the master is not reachable, in order to
    really mark the master as failing, and eventually start a failover procedure if possible.
> - However the quorum is only used to detect the failure. In order to actually perform a failover, one of the Sentinels
    need to be elected leader for the failover and be authorized to proceed. This only happens with the vote of the
    majority of the Sentinel processes.

### 5. 总结

**1. "+odown" 仅针对 master 下线情况，而 replica 和 sentinel 下线时，sentinel 集群只会标识 "+sdown"
，但不会移除相关配置信息，保留将来重新上线的可能性**

**2. 各 sentinel 实例的配置应尽可能保持一致，减少不确定因素**

**3.
高可用性的核心体现在主备切换，而主备切换强依赖于哨兵之间的仲裁规则，因此，应至少在三台独立机器中部署至少三个哨兵，且每台机器哨兵数必须小于(
哨兵总数 + 1) / 2，[反例：1S + 1S, 1S + 1S + 2S]**

**4. Redis 客户端只需连接 sentinel 集群即可，不需要关心由 sentinel 集群屏蔽的读写分发、故障转移等操作**

**5. Replica 默认不支持写请求**

```sh
VM2:6379> SET myKey myValue
(error) READONLY You can't write against a read only replica.
```

**所以本质上 Redis Sentinel
无法解决写压力的问题，针对写入压力瓶颈，请参考 [Redis Cluster](https://redis.io/topics/cluster-tutorial)**