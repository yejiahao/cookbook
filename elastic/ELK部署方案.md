### 查看系统版本
```
[root@localhost ~]# uname -a
Linux localhost.yejh 2.6.32-754.el6.x86_64 #1 SMP Tue Jun 19 21:26:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
[root@localhost ~]# cat /etc/issue
CentOS release 6.10 (Final)
Kernel \r on an \m

```

### 创建普通用户
```
[root@localhost ~]# useradd yejiahao
[root@localhost ~]# passwd yejiahao
Changing password for user yejiahao.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
```

### 检查 JDK (at least Java 8)
```
[root@localhost ~]# java -version
java version "1.8.0_211"
Java(TM) SE Runtime Environment (build 1.8.0_211-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.211-b12, mixed mode)
```

---

### Elasticsearch

#### 1.1 安装 Elasticsearch
***es.inst.sh***
```sh
#!/bin/sh

package=$1.tar.gz
extractPath=/usr/elasticsearch

if [[ "$1" != "" ]]
then
    # download
    echo "downloading ${package}"
    wget -c -P ~/upload/ https://artifacts.elastic.co/downloads/elasticsearch/${package}
    echo "downloaded ${package}"
    # extract
    mkdir -pv ${extractPath}/
    tar -zxvf ~/upload/${package} -C ${extractPath}/
    # (create|update) symbolic link
    ln -snf $1/ ${extractPath}/home

    exit 0
fi

echo "Usage: $0 [version(e.g., elasticsearch-6.8.0)]"
```

#### 1.2 设置环境变量
***/etc/profile (source /etc/profile)***
```properties
# set es profile
ES_HOME=/usr/elasticsearch/home
PATH=$PATH:$ES_HOME/bin
export PATH ES_HOME
```

#### 1.3 为用户添加文件权限
```
[root@localhost ~]# chown -R yejiahao $ES_HOME/
```

#### 1.4 切换用户 (can not run elasticsearch as root)
```
[root@localhost ~]# su yejiahao
```

#### 1.5 修改配置
***$ES_HOME/config/elasticsearch.yml***
```yml
# fix to CentOS 6: CONFIG_SECCOMP not compiled into kernel, CONFIG_SECCOMP and CONFIG_SECCOMP_FILTER are needed
bootstrap.system_call_filter: false

network.host: 0.0.0.0
```

#### 1.6 启动 Elasticsearch
```
[yejiahao@localhost root]$ elasticsearch &
[1] 1234
```

如出现以下报错信息，
```
ERROR: [3] bootstrap checks failed
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
[2]: max number of threads [1024] for user [yejiahao] is too low, increase to at least [4096]
[3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

解决方案如下：
```
[yejiahao@localhost root]$ exit
exit
[root@localhost ~]# 
[root@localhost ~]# echo -e \
> "\n* soft nofile 65535" \
> "\n* hard nofile 65535" \
> "\nyejiahao soft nproc 4096" \
> "\nyejiahao hard nproc 4096" \
> >> /etc/security/limits.conf
[root@localhost ~]# 
[root@localhost ~]# echo -e \
> "\n# Controls the maximum number of memory map, in count" \
> "\nvm.max_map_count = 262144" \
> >> /etc/sysctl.conf
[root@localhost ~]# 
[root@localhost ~]# sysctl -p | egrep "vm.max_map_count"
vm.max_map_count = 262144
[root@localhost ~]# 
[root@localhost ~]# su yejiahao
[yejiahao@localhost root]$ 
[yejiahao@localhost root]$ ulimit -a | egrep "open files|max user processes"
open files                      (-n) 65535
max user processes              (-u) 4096
```

---

### Logstash

#### 2.1 安装 Logstash
***logstash.inst.sh***
```sh
#!/bin/sh

package=$1.tar.gz
extractPath=/usr/logstash

if [[ "$1" != "" ]]
then
    # download
    echo "downloading ${package}"
    wget -c -P ~/upload/ https://artifacts.elastic.co/downloads/logstash/${package}
    echo "downloaded ${package}"
    # extract
    mkdir -pv ${extractPath}/
    tar -zxvf ~/upload/${package} -C ${extractPath}/
    # (create|update) symbolic link
    ln -snf $1/ ${extractPath}/home

    exit 0
fi

echo "Usage: $0 [version(e.g., logstash-6.8.0)]"
```

#### 2.2 设置环境变量
***/etc/profile (source /etc/profile)***
```properties
# set logstash profile
LOGSTASH_HOME=/usr/logstash/home
PATH=$PATH:$LOGSTASH_HOME/bin
export PATH LOGSTASH_HOME
```

#### 2.3 修改配置
***$LOGSTASH_HOME/config/pipelines.yml***
```yml
- pipeline.id: another_test
  queue.type: persisted
  path.config: "/tmp/logstash/*.config"
```

#### 2.4 启动 Logstash
```
[root@localhost ~]# logstash -e 'input { stdin { } } output { stdout {} }'
```

---

### Kibana

#### 3.1 安装 Kibana
***kibana.inst.sh***
```sh
#!/bin/sh

package=$1.tar.gz
extractPath=/usr/kibana

if [[ "$1" != "" ]]
then
    # download
    echo "downloading ${package}"
    wget -c -P ~/upload/ https://artifacts.elastic.co/downloads/kibana/${package}
    echo "downloaded ${package}"
    # extract
    mkdir -pv ${extractPath}/
    tar -zxvf ~/upload/${package} -C ${extractPath}/
    # (create|update) symbolic link
    ln -snf $1/ ${extractPath}/home

    exit 0
fi

echo "Usage: $0 [version(e.g., kibana-6.8.0-linux-x86_64)]"
```

#### 3.2 设置环境变量
***/etc/profile (source /etc/profile)***
```properties
# set kibana profile
KIBANA_HOME=/usr/kibana/home
PATH=$PATH:$KIBANA_HOME/bin
export PATH KIBANA_HOME
```

#### 3.3 修改配置
***$KIBANA_HOME/config/kibana.yml***
```yml
elasticsearch.hosts: ["http://0.0.0.0:9200"]

server.host: "0.0.0.0"
```

#### 3.4 启动 Kibana
```
[root@localhost ~]# kibana &
[1] 1392
```