## CentOS 安装[ Apache Kafka](http://kafka.apache.org/)

> Kafka® is used for building real-time data pipelines and streaming apps. It is horizontally scalable, fault-tolerant,
> wicked fast, and runs in production in thousands of companies.

### 1.1. 查看系统版本

```sh
[root@localhost ~]# uname -a
Linux localhost.yejh 2.6.32-754.el6.x86_64 #1 SMP Tue Jun 19 21:26:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
[root@localhost ~]# cat /etc/issue
CentOS release 6.10 (Final)
Kernel \r on an \m

```

### 1.2. 检查 JDK

```sh
[root@localhost ~]# java -version
java version "1.8.0_211"
Java(TM) SE Runtime Environment (build 1.8.0_211-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.211-b12, mixed mode)
```

### 2.1. 安装 kafka

***kafka.inst.sh***

```sh
#!/bin/sh

package=$1.tgz
extractPath=/usr/kafka

if [[ "$1" != "" ]]
then
    # download
    echo "downloading ${package}"
    wget -c -P ~/upload/ http://mirrors.tuna.tsinghua.edu.cn/apache/kafka/${1##*-}/${package}
    echo "downloaded ${package}"
    # extract
    if [[ ! -d "${extractPath}/" ]]
    then
        mkdir -pv ${extractPath}/
    fi
    tar -zxvf ~/upload/${package} -C ${extractPath}/
    # (create|update) symbolic link
    ln -snf $1/ ${extractPath}/home

    exit 0
fi

echo "Usage: $0 [version(e.g., kafka_2.12-2.2.1)]"
```

***/etc/profile (source /etc/profile)***

```properties
# set kafka profile
KAFKA_HOME=/usr/kafka/home
PATH=$PATH:$KAFKA_HOME/bin
export PATH KAFKA_HOME
```

### 2.2. 修改配置（避免 kafka-server-start.sh 报 UnknownHostException）

***$KAFKA_HOME/config/server.properties***

```properties
listeners = PLAINTEXT://172.16.22.201:9092
```

### 2.3. 启动 Zookeeper & kafka

***kafka.run.sh***

```sh
#!/bin/sh

if [[ "$1" = "start" ]]
then
    ($KAFKA_HOME/bin/zookeeper-server-start.sh $KAFKA_HOME/config/zookeeper.properties &) && \
    ($KAFKA_HOME/bin/kafka-server-start.sh $KAFKA_HOME/config/server.properties &)
elif [[ "$1" = "stop" ]]
then
    ($KAFKA_HOME/bin/kafka-server-stop.sh &) && \
    ($KAFKA_HOME/bin/zookeeper-server-stop.sh &)
else
    echo "Usage: $0 start|stop"
fi
```