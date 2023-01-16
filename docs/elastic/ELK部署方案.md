### 查看系统版本

```sh
[root@vm1 ~]# uname -a
Linux vm1 2.6.32-754.el6.x86_64 #1 SMP Tue Jun 19 21:26:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
[root@vm1 ~]# cat /etc/issue
CentOS release 6.10 (Final)
Kernel \r on an \m

```

### 创建普通用户

```sh
[root@vm1 ~]# useradd es
[root@vm1 ~]# passwd es
Changing password for user es.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
```

### 检查 JDK (at least Java 8)

```sh
[root@vmx ~]# java -version
java version "1.8.0_351"
Java(TM) SE Runtime Environment (build 1.8.0_351-b10)
Java HotSpot(TM) 64-Bit Server VM (build 25.351-b10, mixed mode)
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

echo "Usage: $0 [version(e.g., elasticsearch-6.8.13)]"
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

```sh
[root@vm1 ~]# chown -R es $ES_HOME/
```

#### 1.4 修改配置

***$ES_HOME/config/elasticsearch.yml***

```yml
network.host: 0.0.0.0

# fix to CentOS 6: CONFIG_SECCOMP not compiled into kernel, CONFIG_SECCOMP and CONFIG_SECCOMP_FILTER are needed
bootstrap.system_call_filter: false
```

#### 1.5 启停 Elasticsearch (can not run elasticsearch as root)

```sh
[root@vm1 ~]# su es
[es@vm1 root]$ cd
[es@vm1 ~]$ elasticsearch -d -p es.pid

[es@vm1 ~]$ cat $ES_HOME/es.pid | xargs kill
```

如出现以下报错信息，

```
ERROR: [3] bootstrap checks failed
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
[2]: max number of threads [1024] for user [es] is too low, increase to at least [4096]
[3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

解决方案如下：

```sh
[es@vm1 root]$ exit
exit
[root@vm1 ~]# 
[root@vm1 ~]# echo -e \
> "\n* soft nofile 65535" \
> "\n* hard nofile 65535" \
> "\nes soft nproc 4096" \
> "\nes hard nproc 4096" \
> >> /etc/security/limits.conf
[root@vm1 ~]# 
[root@vm1 ~]# echo -e \
> "\n# Controls the maximum number of memory map, in count" \
> "\nvm.max_map_count = 262144" \
> >> /etc/sysctl.conf
[root@vm1 ~]# 
[root@vm1 ~]# sysctl -p | egrep "vm.max_map_count"
vm.max_map_count = 262144
[root@vm1 ~]# 
[root@vm1 ~]# su es
[es@vm1 root]$ 
[es@vm1 root]$ ulimit -a | egrep "open files|max user processes"
open files                      (-n) 65535
max user processes              (-u) 4096
```

#### 1.7 搭建 Elasticsearch 集群

***$ES_HOME/config/elasticsearch.yml***

```yml
cluster.name: esCluster
# must be unique
node.name: es-node-1
discovery.zen.ping.unicast.hosts: [ "vm1.yejh.cn", "vm2.yejh.cn", "vm3.yejh.cn" ]
discovery.zen.minimum_master_nodes: 2
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

echo "Usage: $0 [version(e.g., logstash-6.8.13)]"
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

```sh
[root@vm1 ~]# logstash -e 'input { stdin { } } output { stdout {} }'
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

echo "Usage: $0 [version(e.g., kibana-6.8.13-linux-x86_64)]"
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
elasticsearch.hosts: [ "http://0.0.0.0:9200" ]

server.host: "0.0.0.0"
```

#### 3.4 启停 Kibana

```sh
[root@vm1 ~]# kibana -p 5601 &
[1] 1392

[root@vm1 ~]# ps -ef | grep kibana | awk -F ' ' 'NR==1 {print $2}' | xargs kill
```

---

### Filebeat

#### 4.1 安装 Filebeat

***filebeat.inst.sh***

```sh
#!/bin/sh

package=$1.tar.gz
extractPath=/usr/filebeat

if [[ "$1" != "" ]]
then
    # download
    echo "downloading ${package}"
    wget -c -P ~/upload/ https://artifacts.elastic.co/downloads/beats/filebeat/${package}
    echo "downloaded ${package}"
    # extract
    mkdir -pv ${extractPath}/
    tar -zxvf ~/upload/${package} -C ${extractPath}/
    # (create|update) symbolic link
    ln -snf $1/ ${extractPath}/home

    exit 0
fi

echo "Usage: $0 [version(e.g., filebeat-6.8.13-linux-x86_64)]"
```

#### 4.2 设置环境变量

***/etc/profile (source /etc/profile)***

```properties
# set filebeat profile
FILEBEAT_HOME=/usr/filebeat/home
PATH=$PATH:$FILEBEAT_HOME
export PATH FILEBEAT_HOME
```

#### 4.3 修改配置

***$FILEBEAT_HOME/filebeat.yml***

```yml
setup.kibana:

  host: "vm1.yejh.cn:5601"

output.elasticsearch:
  # Array of hosts to connect to.
  hosts: [ "vm1.yejh.cn:9200", "vm2.yejh.cn:9200", "vm3.yejh.cn:9200" ]
```

#### 4.4 开启监控模块（修改相应的 $FILEBEAT_HOME/modules.d/\<module\>.yml）

```sh
[root@vm1 ~]# cd $FILEBEAT_HOME
[root@vm1 home]# ./filebeat modules list
[root@vm1 home]# ./filebeat modules enable nginx redis
Enabled nginx
Enabled redis

[root@vm1 home]# ./filebeat setup --dashboards
```

#### 4.5 启动 Filebeat

```sh
[root@vm1 ~]# cd $FILEBEAT_HOME
[root@vm1 home]# ./filebeat &
[2] 1830
```

> [Filebeat 参考文档](https://www.elastic.co/guide/en/beats/filebeat/6.8/filebeat-getting-started.html)
