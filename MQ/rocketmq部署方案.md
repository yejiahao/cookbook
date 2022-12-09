## CentOS 安装 [**Apache RocketMQ**](http://rocketmq.apache.org/)

> Apache RocketMQ™ is an open source distributed messaging and streaming data platform.

### 1.1. 查看系统版本
```
[root@vm1 ~]# uname -a
Linux vm1 2.6.32-754.el6.x86_64 #1 SMP Tue Jun 19 21:26:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
[root@vm1 ~]# cat /etc/issue
CentOS release 6.10 (Final)
Kernel \r on an \m

```

### 1.2. 检查 JDK, Maven
```
[root@vm1 ~]# java -version
java version "1.8.0_271"
Java(TM) SE Runtime Environment (build 1.8.0_271-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.271-b09, mixed mode)
[root@vm1 ~]# 
[root@vm1 ~]# mvn -v
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: /usr/maven/home
Java version: 1.8.0_271, vendor: Oracle Corporation, runtime: /usr/java/jdk1.8.0_271/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "2.6.32-754.el6.x86_64", arch: "amd64", family: "unix"
```

### 2.1. 安装 RocketMQ
***rocketmq.inst.sh***
```sh
#!/bin/sh

package=${1}.zip
extractPath=${2}

if [[ "${2}" == "" ]]
then
  extractPath=/usr/rocketmq
fi

if [[ "${1}" != "" ]]
then
  # download
  echo "downloading ${package}"
  wget -c -P $HOME/upload/ http://mirrors.tuna.tsinghua.edu.cn/apache/rocketmq/${1:13:5}/${package}
  echo "downloaded ${package}"
  # extract
  unzip $HOME/upload/${package} -d ${extractPath}/
  # (create|update) symbolic link
  ln -snf ${1}/ ${extractPath}/home

  exit 0
fi

echo "Usage: $0 [version(e.g., rocketmq-all-4.5.2-(bin|source)-release)]"
```

### 2.2. 编译打包（下载 source 包的情况）
```
[root@vm1 ~]# cd /usr/rocketmq/rocketmq-all-4.5.2-source-release/
[root@vm1 rocketmq-all-4.5.2-source-release]# mvn -Prelease-all -DskipTests clean install -U
```
- ***distribution/target/rocketmq-4.5.2.zip*** 等价于 2.1 中的 bin 包
- ***distribution/target/rocketmq-4.5.2/rocketmq-4.5.2/*** 等价于 2.3 中的 ROCKETMQ_HOME

### 2.3. 配置环境变量
***/etc/profile (source /etc/profile)***
```properties
# set rocketmq profile
ROCKETMQ_HOME=/usr/rocketmq/home
PATH=$PATH:$ROCKETMQ_HOME/bin
export PATH ROCKETMQ_HOME
```

### 3. 启动关闭 namesrv && broker
```
[root@vm1 ~]# nohup sh mqnamesrv &
[root@vm1 ~]# 
[root@vm1 ~]# nohup sh mqbroker -c $ROCKETMQ_HOME/conf/broker.conf -n localhost:9876 &
[root@vm1 ~]# 
[root@vm1 ~]# jps | grep -v Jps
9298 BrokerStartup
9239 NamesrvStartup
[root@vm1 ~]# 
[root@vm1 ~]# sh mqshutdown broker
The mqbroker(9298) is running...
Send shutdown request to mqbroker(9298) OK
[root@vm1 ~]# 
[2]+  Exit 143                nohup sh mqbroker -c $ROCKETMQ_HOME/conf/broker.conf -n localhost:9876
[root@vm1 ~]# sh mqshutdown namesrv
The mqnamesrv(9239) is running...
Send shutdown request to mqnamesrv(9239) OK
[root@vm1 ~]# 
[1]+  Exit 143                nohup sh mqnamesrv
```

### 4. 客户端依赖
***pom.xml***
```xml
<!-- RocketMQ -->
<dependency>
	<groupId>org.apache.rocketmq</groupId>
	<artifactId>rocketmq-client</artifactId>
	<version>${rocketmq.version}</version>
</dependency>
```

### 5. 其它问题
- #### 内存初始值过大，导致 broker 启动失败
***$ROCKETMQ_HOME/bin/runserver.sh***

~~JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"~~
```sh
JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx1g -Xmn256m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```
~~JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"~~
```sh
JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx1g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```
***$ROCKETMQ_HOME/bin/runbroker.sh***

~~JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g"~~
```sh
JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx1g"
```

- #### 客户端外网访问 RocketMQ 失败
***$ROCKETMQ_HOME/conf/broker.conf***
```properties
# 多个地址以;分隔
# namesrvAddr = 1.2.3.4:9876;
brokerIP1 = 1.2.3.4
```

- #### 同一机器启动多个 broker 实例失败
***$ROCKETMQ_HOME/conf/broker.conf***
```properties
storePathRootDir=/root/store/broker-a
listenPort=10911
enablePropertyFilter=true
```
> ```echo -e 'storePathRootDir=/root/store/broker-a\nlistenPort=10911\nenablePropertyFilter=true' >> $ROCKETMQ_HOME/conf/broker.conf```

- #### 删除 RocketMQ 数据
```
[root@vm1 ~]# rm -rf logs/ nohup.out store/
```