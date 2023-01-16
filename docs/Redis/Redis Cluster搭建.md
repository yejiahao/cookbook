# Redis Cluster(集群模式)

> Redis 集群模式的特点
> - 去中心化架构，数据散列在不同主节点
> - 支持多主多从
> - 集群各个节点互相监视，无须引入哨兵实例实现故障转移（主从切换）
> - 默认情况下主节点分摊写请求，全节点分摊读请求

### 参考资料

* [官方文档1](https://redis.io/topics/cluster-tutorial)
* [官方文档2](https://redis.io/topics/cluster-spec)

### 1. 集群搭建

#### 1.1. 查看 OS, Redis 版本

|         OS          |      Redis      |
|:-------------------:|:---------------:|
| CentOS release 6.10 | redis-cli 5.0.9 |

#### 1.2. 编辑脚本

***/usr/redis/home/utils/create-cluster/create-cluster***

```sh
#!/bin/bash

# Settings
IP=172.16.22.230
PORT=30000
TIMEOUT=2000
NODES=6
REPLICAS=1
PASSWORD=redis20170419

# You may want to put the above config parameters into config.sh in order to
# override the defaults without modifying this script.

if [ -a config.sh ]
then
    source "config.sh"
fi

# Computed vars
ENDPORT=$((PORT+NODES))

if [ "$1" == "start" ]
then
    while [ $((PORT < ENDPORT)) != "0" ]; do
        PORT=$((PORT+1))
        echo "Starting $PORT"
        redis-server --port $PORT --cluster-enabled yes --cluster-config-file nodes-${PORT}.conf --cluster-node-timeout $TIMEOUT --appendonly yes --appendfilename appendonly-${PORT}.aof --dbfilename dump-${PORT}.rdb --logfile ${PORT}.log --daemonize yes\
        --protected-mode no --requirepass $PASSWORD --masterauth $PASSWORD
    done
    exit 0
fi

if [ "$1" == "create" ]
then
    HOSTS=""
    while [ $((PORT < ENDPORT)) != "0" ]; do
        PORT=$((PORT+1))
        HOSTS="$HOSTS $IP:$PORT"
    done
    redis-cli --cluster create $HOSTS --cluster-replicas $REPLICAS -a $PASSWORD
    exit 0
fi

if [ "$1" == "stop" ]
then
    while [ $((PORT < ENDPORT)) != "0" ]; do
        PORT=$((PORT+1))
        echo "Stopping $PORT"
        redis-cli -p $PORT -a $PASSWORD shutdown nosave
    done
    exit 0
fi

if [ "$1" == "watch" ]
then
    PORT=$((PORT+1))
    while [ 1 ]; do
        clear
        date
        redis-cli -p $PORT -a $PASSWORD cluster nodes | head -30
        sleep 1
    done
    exit 0
fi

if [ "$1" == "check" ]
then
    INSTANCE=$2
    PORT=$((PORT+INSTANCE))
    redis-cli --cluster check $IP:$PORT -a $PASSWORD
    exit 0
fi

if [ "$1" == "tail" ]
then
    INSTANCE=$2
    PORT=$((PORT+INSTANCE))
    tail -f ${PORT}.log
    exit 0
fi

if [ "$1" == "call" ]
then
    while [ $((PORT < ENDPORT)) != "0" ]; do
        PORT=$((PORT+1))
        redis-cli -p $PORT -a $PASSWORD $2 $3 $4 $5 $6 $7 $8 $9
    done
    exit 0
fi

if [ "$1" == "clean" ]
then
    rm -rf *.log
    rm -rf appendonly*.aof
    rm -rf dump*.rdb
    rm -rf nodes*.conf
    exit 0
fi

if [ "$1" == "clean-logs" ]
then
    rm -rf *.log
    exit 0
fi

echo "Usage: $0 [start|create|stop|watch|check|tail|clean]"
echo "start       -- Launch Redis Cluster instances."
echo "create      -- Create a cluster using redis-cli --cluster create."
echo "stop        -- Stop Redis Cluster instances."
echo "watch       -- Show CLUSTER NODES output (first 30 lines) of first node."
echo "check <id>  -- Test the health of the cluster."
echo "tail <id>   -- Run tail -f of instance at base port + ID."
echo "clean       -- Remove all instances data, logs, configs."
echo "clean-logs  -- Remove just instances logs."

```

### 2. 集群使用

#### 2.1. 创建并启动集群

```sh
[root@vm3 ~]# cd /usr/redis/home/utils/create-cluster/
[root@vm3 create-cluster]# 
[root@vm3 create-cluster]# ./create-cluster start
Starting 30001
Starting 30002
Starting 30003
Starting 30004
Starting 30005
Starting 30006
[root@vm3 create-cluster]# 
[root@vm3 create-cluster]# ./create-cluster create
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 172.16.22.230:30005 to 172.16.22.230:30001
Adding replica 172.16.22.230:30006 to 172.16.22.230:30002
Adding replica 172.16.22.230:30004 to 172.16.22.230:30003
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: e1cd165729cc5c43c6454a1f90d09e0f682a713a 172.16.22.230:30001
   slots:[0-5460] (5461 slots) master
M: 38fa93ab8575dbe4ff2ba3fbd64f880847df612a 172.16.22.230:30002
   slots:[5461-10922] (5462 slots) master
M: 999a9cad4b2b084e5170f8b5905b45a2b3aeb204 172.16.22.230:30003
   slots:[10923-16383] (5461 slots) master
S: 46c9b710441bcd9cc4d83642888bb91f34d1b0cc 172.16.22.230:30004
   replicates e1cd165729cc5c43c6454a1f90d09e0f682a713a
S: e1e559a5df3661d384dc993c3903864b6d8deee0 172.16.22.230:30005
   replicates 38fa93ab8575dbe4ff2ba3fbd64f880847df612a
S: e97670174c28b3c05edaf4e95786763cf28ade48 172.16.22.230:30006
   replicates 999a9cad4b2b084e5170f8b5905b45a2b3aeb204
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.
>>> Performing Cluster Check (using node 172.16.22.230:30001)
M: e1cd165729cc5c43c6454a1f90d09e0f682a713a 172.16.22.230:30001
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 46c9b710441bcd9cc4d83642888bb91f34d1b0cc 172.16.22.230:30004
   slots: (0 slots) slave
   replicates e1cd165729cc5c43c6454a1f90d09e0f682a713a
M: 38fa93ab8575dbe4ff2ba3fbd64f880847df612a 172.16.22.230:30002
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
M: 999a9cad4b2b084e5170f8b5905b45a2b3aeb204 172.16.22.230:30003
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: e97670174c28b3c05edaf4e95786763cf28ade48 172.16.22.230:30006
   slots: (0 slots) slave
   replicates 999a9cad4b2b084e5170f8b5905b45a2b3aeb204
S: e1e559a5df3661d384dc993c3903864b6d8deee0 172.16.22.230:30005
   slots: (0 slots) slave
   replicates 38fa93ab8575dbe4ff2ba3fbd64f880847df612a
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

#### 2.2. 脚本参考手册

```sh
[root@vm3 create-cluster]# ./create-cluster help
Usage: ./create-cluster [start|create|stop|watch|check|tail|clean]
start       -- Launch Redis Cluster instances.
create      -- Create a cluster using redis-cli --cluster create.
stop        -- Stop Redis Cluster instances.
watch       -- Show CLUSTER NODES output (first 30 lines) of first node.
check <id>  -- Test the health of the cluster.
tail <id>   -- Run tail -f of instance at base port + ID.
clean       -- Remove all instances data, logs, configs.
clean-logs  -- Remove just instances logs.
```

#### 2.3. 集群测试

```sh
[root@vm3 ~]# redis-cli -c -p 30001 -a redis20170419
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
172.16.22.230:30001> SET aaa bbb
-> Redirected to slot [10439] located at 172.16.22.230:30002
OK
172.16.22.230:30002> MSET {yjh}aaa bbb {yjh}ccc ddd
-> Redirected to slot [4054] located at 172.16.22.230:30001
OK
172.16.22.230:30001> GET aaa
-> Redirected to slot [10439] located at 172.16.22.230:30002
"bbb"
172.16.22.230:30002> MGET {yjh}aaa {yjh}ccc
-> Redirected to slot [4054] located at 172.16.22.230:30001
1) "bbb"
2) "ddd"
```

### 3. 备注集群配置

***/usr/redis/home/redis.conf***

```properties
################################ REDIS CLUSTER  ###############################

# Normal Redis instances can't be part of a Redis Cluster; only nodes that are
# started as cluster nodes can. In order to start a Redis instance as a
# cluster node enable the cluster support uncommenting the following:
#
# cluster-enabled yes

# Every cluster node has a cluster configuration file. This file is not
# intended to be edited by hand. It is created and updated by Redis nodes.
# Every Redis Cluster node requires a different cluster configuration file.
# Make sure that instances running in the same system do not have
# overlapping cluster configuration file names.
#
# cluster-config-file nodes-6379.conf

# Cluster node timeout is the amount of milliseconds a node must be unreachable
# for it to be considered in failure state.
# Most other internal time limits are multiple of the node timeout.
#
# cluster-node-timeout 15000

# A replica of a failing master will avoid to start a failover if its data
# looks too old.
#
# There is no simple way for a replica to actually have an exact measure of
# its "data age", so the following two checks are performed:
#
# 1) If there are multiple replicas able to failover, they exchange messages
#    in order to try to give an advantage to the replica with the best
#    replication offset (more data from the master processed).
#    Replicas will try to get their rank by offset, and apply to the start
#    of the failover a delay proportional to their rank.
#
# 2) Every single replica computes the time of the last interaction with
#    its master. This can be the last ping or command received (if the master
#    is still in the "connected" state), or the time that elapsed since the
#    disconnection with the master (if the replication link is currently down).
#    If the last interaction is too old, the replica will not try to failover
#    at all.
#
# The point "2" can be tuned by user. Specifically a replica will not perform
# the failover if, since the last interaction with the master, the time
# elapsed is greater than:
#
#   (node-timeout * replica-validity-factor) + repl-ping-replica-period
#
# So for example if node-timeout is 30 seconds, and the replica-validity-factor
# is 10, and assuming a default repl-ping-replica-period of 10 seconds, the
# replica will not try to failover if it was not able to talk with the master
# for longer than 310 seconds.
#
# A large replica-validity-factor may allow replicas with too old data to failover
# a master, while a too small value may prevent the cluster from being able to
# elect a replica at all.
#
# For maximum availability, it is possible to set the replica-validity-factor
# to a value of 0, which means, that replicas will always try to failover the
# master regardless of the last time they interacted with the master.
# (However they'll always try to apply a delay proportional to their
# offset rank).
#
# Zero is the only value able to guarantee that when all the partitions heal
# the cluster will always be able to continue.
#
# cluster-replica-validity-factor 10

# Cluster replicas are able to migrate to orphaned masters, that are masters
# that are left without working replicas. This improves the cluster ability
# to resist to failures as otherwise an orphaned master can't be failed over
# in case of failure if it has no working replicas.
#
# Replicas migrate to orphaned masters only if there are still at least a
# given number of other working replicas for their old master. This number
# is the "migration barrier". A migration barrier of 1 means that a replica
# will migrate only if there is at least 1 other working replica for its master
# and so forth. It usually reflects the number of replicas you want for every
# master in your cluster.
#
# Default is 1 (replicas migrate only if their masters remain with at least
# one replica). To disable migration just set it to a very large value.
# A value of 0 can be set but is useful only for debugging and dangerous
# in production.
#
# cluster-migration-barrier 1

# By default Redis Cluster nodes stop accepting queries if they detect there
# is at least an hash slot uncovered (no available node is serving it).
# This way if the cluster is partially down (for example a range of hash slots
# are no longer covered) all the cluster becomes, eventually, unavailable.
# It automatically returns available as soon as all the slots are covered again.
#
# However sometimes you want the subset of the cluster which is working,
# to continue to accept queries for the part of the key space that is still
# covered. In order to do so, just set the cluster-require-full-coverage
# option to no.
#
# cluster-require-full-coverage yes

# This option, when set to yes, prevents replicas from trying to failover its
# master during master failures. However the master can still perform a
# manual failover, if forced to do so.
#
# This is useful in different scenarios, especially in the case of multiple
# data center operations, where we want one side to never be promoted if not
# in the case of a total DC failure.
#
# cluster-replica-no-failover no

# In order to setup your cluster make sure to read the documentation
# available at http://redis.io web site.

########################## CLUSTER DOCKER/NAT support  ########################

# In certain deployments, Redis Cluster nodes address discovery fails, because
# addresses are NAT-ted or because ports are forwarded (the typical case is
# Docker and other containers).
#
# In order to make Redis Cluster working in such environments, a static
# configuration where each node knows its public address is needed. The
# following two options are used for this scope, and are:
#
# * cluster-announce-ip
# * cluster-announce-port
# * cluster-announce-bus-port
#
# Each instruct the node about its address, client port, and cluster message
# bus port. The information is then published in the header of the bus packets
# so that other nodes will be able to correctly map the address of the node
# publishing the information.
#
# If the above options are not used, the normal Redis Cluster auto-detection
# will be used instead.
#
# Note that when remapped, the bus port may not be at the fixed offset of
# clients port + 10000, so you can specify any port and bus-port depending
# on how they get remapped. If the bus-port is not set, a fixed offset of
# 10000 will be used as usually.
#
# Example:
#
# cluster-announce-ip 10.1.1.5
# cluster-announce-port 6379
# cluster-announce-bus-port 6380
```