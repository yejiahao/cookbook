# Redis

### Redis 简介

#### redis是一个使用 ANSI C 语言编写、支持网络、可基于内存亦可持久化的日志型的高性能 key-value 数据库，完全开源免费，遵守 BSD 协议。
- 支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用
- 不仅仅支持简单的 key-value 类型的数据，同时还提供 list，set，zset，hash 等数据结构的存储
- 特点：易扩展、高性能、高可用
- 应用场景：缓存、任务队列、分布式 session 分离

#### [Redis 官网](https://redis.io/)

#### [Redis Desktop Manager 官网](https://redisdesktop.com/)

***

### Redis 安装与卸载

#### 1. 查看系统版本
```
[root@vmx ~]# uname -a
Linux localhost.yejh 2.6.32-754.el6.x86_64 #1 SMP Tue Jun 19 21:26:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
[root@vmx ~]# cat /etc/issue
CentOS release 6.10 (Final)
Kernel \r on an \m

```

#### 2. 安装 Redis
***redis.inst.sh***
```sh
#!/bin/sh

package=$1.tar.gz
extractPath=/usr/redis

if [[ "$1" != "" ]]
then
    yum -y install gcc tcl
    # download
    echo "downloading ${package}"
    wget -c -P ~/upload/ http://download.redis.io/releases/${package}
    echo "downloaded ${package}"
    # extract and install
    if [[ ! -d "${extractPath}/" ]]
    then
        mkdir -pv ${extractPath}/
    fi
    tar -zxvf ~/upload/${package} -C ${extractPath}/ && cd ${extractPath}/$1/
    make clean
    make MALLOC=libc && cd ${extractPath}/$1/src/
    make test && make install
    # (create|update) symbolic link
    ln -snf $1/ ${extractPath}/home

    exit 0
fi

echo "Usage: $0 [version(e.g., redis-5.0.5)]"
```

***/usr/redis/home/redis.conf***
```properties
# bind 127.0.0.1 # 远程连接
protected-mode no # 远程无认证
daemonize yes # 守护线程，后台运行
```

#### 3. 后台启动关闭 Redis 服务端
```
[root@vmx ~]# redis-server /usr/redis/home/redis.conf
9019:C 03 Jun 2019 14:07:50.888 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
9019:C 03 Jun 2019 14:07:50.888 # Redis version=5.0.5, bits=64, commit=00000000, modified=0, pid=9019, just started
9019:C 03 Jun 2019 14:07:50.888 # Configuration loaded
[root@vmx ~]# 
[root@vmx ~]# pkill redis-server
```

#### 4. 启动关闭 Redis 客户端
```
[root@vmx ~]# redis-cli
127.0.0.1:6379> SET k1 v1
OK
127.0.0.1:6379> GET k1
"v1"
127.0.0.1:6379> EXIT
[root@vmx ~]# 
[root@vmx ~]# redis-cli SHUTDOWN
```

#### 5. 查看端口情况
```
[root@vmx ~]# netstat -anop | grep 6379
tcp        0      0 0.0.0.0:6379                0.0.0.0:*                   LISTEN      9020/redis-server * off (0.00/0/0)
tcp        0      0 :::6379                     :::*                        LISTEN      9020/redis-server * off (0.00/0/0)
[root@vmx ~]# 
[root@vmx ~]# lsof -i:6379
COMMAND    PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
redis-ser 9020 root    6u  IPv6  38989      0t0  TCP *:6379 (LISTEN)
redis-ser 9020 root    7u  IPv4  38990      0t0  TCP *:6379 (LISTEN)
```

#### 6. 卸载 Redis
6.1. 停止 Redis 服务器（如上）

6.2. 删除 redis installed 文件
```
[root@vmx ~]# rm -f /usr/local/bin/redis*
```

6.3. 删除 redis extracted 文件
```
[root@vmx ~]# rm -rf /usr/redis/
```

### 7. 设置自启动
7.1 添加启动文件

***/etc/init.d/redis***
```sh
#!/bin/sh

# chkconfig: 3 10 90
# description: start and stop Redis

port=6379
redisServer=/usr/local/bin/redis-server
redisClient=/usr/local/bin/redis-cli
pidfile=/var/run/redis_6379.pid
conf="/usr/redis/home/redis.conf"

case "$1" in
start)
  if [ -f $pidfile ]; then
    echo "$pidfile exists, process is already running or crashed."
  else
    echo "Starting Redis server..."
    $redisServer $conf
  fi
  if [ "$?" = "0" ]; then
    echo "Redis is running..."
  fi
  ;;
stop)
  if [ ! -f $pidfile ]; then
    echo "$pidfile not exists, process is not running."
  else
    echo "Stopping Redis server..."
    $redisClient -p $port SHUTDOWN
    while [ -x $pidfile ]; do
      echo "Waiting for Redis to shutdown..."
      sleep 1
    done
    echo "Redis stopped"
  fi
  ;;
restart)
  ${0} stop
  ${0} start
  ;;
*)
  echo "Usage: /etc/init.d/redis {start|stop|restart}" >&2
  exit 1
  ;;
esac
```

7.2 配置启动
```
[root@vmx ~]# chmod +x /etc/init.d/redis
[root@vmx ~]# chkconfig --add redis
[root@vmx ~]# chkconfig --list redis
redis           0:off   1:off   2:off   3:on    4:off   5:off   6:off
[root@vmx ~]# service redis
Usage: /etc/init.d/redis {start|stop|restart}
```

***

### jedis 的使用
***pom.xml***
```xml
<!-- redis -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.0.1</version>
</dependency>
```
***JedisTest.java***
```java
// 连接本地的redis服务
try (Jedis jedis = new Jedis("172.16.22.201", 6379)) {
    System.out.println("连接成功");
    // 查看服务是否运行
    System.out.println("服务正在运行: " + jedis.ping());
    // 设置 Redis 字符串
    jedis.set("key", "value");
    // 获取存储的数据并输出
    System.out.println("Redis 存储的字符串为: " + jedis.get("key"));
}
```

***

### Redis 持久化存储(RDB or AOF)
- **RDB(redis database)**
数据按一定规则定时从内存保存到硬盘数据库文件上，全量同步

***/usr/redis/home/redis.conf***
```properties
save 300 10 # 300秒内至少发生10次键值修改
stop-writes-on-bgsave-error yes # 出错时阻塞变更
rdbcompression yes # 压缩
dbfilename dump.rdb # 文件名
dir ./ # 目录
```

- **AOF(append-only file)**
Redis 启动时读取日志文本，运行时操作都记录到日志文件上，增量追加

***/usr/redis/home/redis.conf***
```properties
appendonly yes # 开启AOF
appendfilename "appendonly.aof" # 日志文件名
appendfsync always # 按修改同步
no-appendfsync-on-rewrite no # aof-rewrite时不暂停同步
auto-aof-rewrite-percentage 100 # aof-rewrite最小值
auto-aof-rewrite-min-size 64mb # aof-rewrite增长率
```

**对比**

| 持久化方式 | 特征 |
|-----|-----|
| RDB | 开启单独子进程来进行持久化，速度快，性能较好 |
| AOF | 可修改日志文本，不易丢失数据，更高的数据完整性 |

***

### 分布式 session 共享
[ Spring Session 和 Redis 解决分布式 session ](https://blog.csdn.net/xlgen157387/article/details/57406162)
> 用 Spring Session 支持的 HttpSession 实现来替换容器本身 HttpSession 实现

> 不是 Tomcat 创建好了 session 之后，我们获取 session 然后再上传到 Redis 等进行存储，而是直接创建 session