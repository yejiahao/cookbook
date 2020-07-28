## [**rocketmq-console**](https://github.com/apache/rocketmq-externals/tree/master/rocketmq-console)

### 1. 检查基础配置
```
[root@vm1 ~]# echo $ROCKETMQ_HOME
/usr/rocketmq/home
[root@vm1 ~]# 
[root@vm1 ~]# mvn -v
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: /usr/maven/home
Java version: 1.8.0_241, vendor: Oracle Corporation, runtime: /usr/java/jdk1.8.0_241/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "2.6.32-754.el6.x86_64", arch: "amd64", family: "unix"
[root@vm1 ~]# 
[root@vm1 ~]# git --version
git version 2.26.0
```

### 2. 拉取 rocketmq-console 源码
```
[root@vm1 ~]# git clone https://github.com/apache/rocketmq-externals.git $HOME/project/rocketmq-externals/
Cloning into '/root/project/rocketmq-externals'...
remote: Enumerating objects: 40, done.
remote: Counting objects: 100% (40/40), done.
remote: Compressing objects: 100% (29/29), done.
remote: Total 18789 (delta 4), reused 23 (delta 1), pack-reused 18749
Receiving objects: 100% (18789/18789), 33.22 MiB | 513.00 KiB/s, done.
Resolving deltas: 100% (7352/7352), done.
Updating files: 100% (6193/6193), done.
```

### 3. 添加启停脚本
***rocketmq.run.sh***
```sh
#!/bin/sh

mqAddr=localhost:9876
consoleLocation=$HOME/project/rocketmq-externals/rocketmq-console/
consoleFile=rocketmq-console-ng-2.0.0.jar

case "$1" in
start)
  # 启动 namesrv
  nohup sh mqnamesrv &
  sleep 5s
  # 启动 broker
  nohup sh mqbroker -c "$ROCKETMQ_HOME"/conf/broker.conf -n $mqAddr &
  sleep 10s
  # 启动 console
  cd "$consoleLocation" || exit
  if [ ! -d target/ ]; then
    mvn clean package -Dmaven.test.skip=true
  fi
  # 运行时生成的 nohup.out 可被 .gitignore 覆盖到
  cd target/ && \
  (nohup java -jar $consoleFile --server.port=6789 --rocketmq.config.namesrvAddr=$mqAddr &) && \
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
  echo "Usage: $0 {start|stop}" >&2
  exit 1
  ;;
esac

```
