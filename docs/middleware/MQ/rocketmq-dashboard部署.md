## [rocketmq-dashboard](https://github.com/apache/rocketmq-dashboard)

### 1. 检查基础配置

```sh
[root@vmx ~]# echo $ROCKETMQ_HOME
/usr/rocketmq/home
[root@vmx ~]# 
[root@vmx ~]# mvn -v
Apache Maven 3.8.6 (84538c9988a25aec085021c365c560670ad80f63)
Maven home: /usr/maven/home
Java version: 1.8.0_351, vendor: Oracle Corporation, runtime: /usr/java/jdk1.8.0_351/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "4.18.0-348.7.1.el8_5.x86_64", arch: "amd64", family: "unix"
[root@vmx ~]# 
[root@vmx ~]# git --version
git version 2.38.1
```

### 2. 拉取 rocketmq-dashboard 源码

```sh
[root@vmx ~]# git clone https://github.com/apache/rocketmq-dashboard.git $HOME/project/rocketmq-dashboard/
Cloning into '/root/project/rocketmq-dashboard'...
remote: Enumerating objects: 5021, done.
remote: Counting objects: 100% (78/78), done.
remote: Compressing objects: 100% (55/55), done.
remote: Total 5021 (delta 32), reused 49 (delta 11), pack-reused 4943
Receiving objects: 100% (5021/5021), 5.57 MiB | 41.00 KiB/s, done.
Resolving deltas: 100% (2935/2935), done.
```

### 3. 添加启停脚本

***rocketmq.run.sh***

```sh
#!/bin/sh

namesrvAddr=$2
consoleLocation=$HOME/project/rocketmq-dashboard/
consoleFile=rocketmq-dashboard-1.0.1-SNAPSHOT.jar
accessKey=rocketmq
secretKey=rocketmq20170419

if [ -z "$2" ]
then
  namesrvAddr=localhost:9876
fi

case "$1" in
start)
  # 启动 namesrv
  nohup sh mqnamesrv &
  sleep 5s
  # 启动 broker
  nohup sh mqbroker -c "$ROCKETMQ_HOME"/conf/2m-2s-async/broker-a.properties -n $namesrvAddr &
  sleep 4s
  # nohup sh mqbroker -c "$ROCKETMQ_HOME"/conf/2m-2s-async/broker-b.properties -n $namesrvAddr &
  # sleep 4s
  # nohup sh mqbroker -c "$ROCKETMQ_HOME"/conf/2m-2s-async/broker-a-s.properties -n $namesrvAddr &
  # sleep 4s
  # nohup sh mqbroker -c "$ROCKETMQ_HOME"/conf/2m-2s-async/broker-b-s.properties -n $namesrvAddr &
  # sleep 4s
  # 启动 console
  cd "$consoleLocation" || exit
  if [ ! -d target/ ]; then
    mvn clean package -Dmaven.test.skip=true
  fi
  # 运行时生成的 nohup.out 可被 .gitignore 覆盖到
  cd target/ && \
  (nohup java -jar $consoleFile --server.port=6789 --rocketmq.config.namesrvAddr=$namesrvAddr --rocketmq.config.accessKey=$accessKey --rocketmq.config.secretKey=$secretKey &) && \
  cd "$HOME" || exit
  ;;
stop)
  # 关闭 console
  # shellcheck disable=SC2009
  ps -ef | grep $consoleFile | grep -v grep | awk 'NR==1 {print $2}' | xargs kill
  sleep 3s
  # 关闭 broker
  sh mqshutdown broker
  sleep 3s
  # 关闭 namesrv
  sh mqshutdown namesrv
  ;;
*)
  echo "Usage: $0 {start|stop} [<namesrvAddr1\;namesrvAddr2>]" >&2
  exit 1
  ;;
esac

```
