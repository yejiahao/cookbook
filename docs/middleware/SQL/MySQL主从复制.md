## MySQL 主从复制(master-slave)

> 基于 **mysql-8.0.16-linux-glibc2.12-x86_64.tar.xz**

- 主库(master): vm1.yejh.cn:3306
- 从库(slave): vm2.yejh.cn:3306

### 修改从库配置

***/etc/my.cnf***

```properties
[mysqld]
server-id=150
relay-log=relay-bin
relay-log-index=relay-bin.index
#replicate-do-db=mydb
replicate-ignore-db=mysql
replicate-ignore-db=sys
```

### 重启从库

```sh
[root@vm2 ~]# service mysql restart
Shutting down MySQL.. SUCCESS! 
Starting MySQL.. SUCCESS!
```

### 复制同步

```sh
[root@vm2 ~]# mysql -h vm1.yejh.cn -P 3306 -u root -pmysql20170419
...
mysql> SHOW MASTER STATUS;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000011 |      155 |              |                  |                   |
+---------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> EXIT;
Bye
[root@vm2 ~]# mysql -h vm2.yejh.cn -P 3306 -u root -pmysql20170419
...
mysql> STOP SLAVE;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> CHANGE MASTER TO
    -> MASTER_HOST = 'vm1.yejh.cn',
    -> MASTER_PORT = 3306,
    -> MASTER_USER = 'ROOT',
    -> MASTER_PASSWORD = 'mysql20170419',
    -> MASTER_LOG_FILE = 'binlog.000011',
    -> MASTER_LOG_POS = 155;
Query OK, 0 rows affected, 1 warning (0.09 sec)

mysql> START SLAVE USER = 'root' PASSWORD = 'mysql20170419';
Query OK, 0 rows affected, 1 warning (0.07 sec)

mysql> EXIT;
Bye
```

### 其它指令

```sh
mysql> SHOW SLAVE STATUS;
mysql> RESET SLAVE;
mysql> SHOW VARIABLES LIKE 'slave%';
mysql> SET GLOBAL slave_compressed_protocol = 1;
...
```

***

### 更多信息参考[ MySQL8.0 Replication](https://dev.mysql.com/doc/refman/8.0/en/replication.html)

***

## MySQL V5.7 配置主从同步

### 1. 准备环境

|            | master                                                                                               | slave                                                                                            |
|:----------:|------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
|   domain   | vm0.yejh.cn                                                                                          | vm5.yejh.cn                                                                                      |
|     OS     | windows 11                                                                                           | CentOS 8.3                                                                                       |
|   MySQL    | F:\mysql-5.7.43-winx64\bin\mysqld.exe  Ver 5.7.43 for Win64 on x86_64 (MySQL Community Server (GPL)) | mysqld  Ver 5.7.43 for linux-glibc2.12 on x86_64 (MySQL Community Server (GPL))                  |
| dependency | 防火墙打开 `3307` 端口                                                                                      | 加入两个类库文件 ```ln -s /usr/lib64/libncurses{libtinfo}.so.6.1 /usr/lib64/libncurses{libtinfo}.so.5``` |
|    DDL     | ```CREATE DATABASE `sync_vm`;```                                                                     | ```CREATE DATABASE `sync_vm`;```                                                                 |

### 2. 配置主库

#### 2.1. 主库配置文件

***%MYSQL_HOME%\my.ini***

```properties
[mysqld]
character-set-server = UTF8MB4
# 绑定 IPv4 和 3307 端口
bind-address = 0.0.0.0
port = 3307
# 设置 mysql 的安装目录
basedir = F:\mysql-5.7.43-winx64
# 设置 mysql 数据库的数据的存放目录
datadir = F:\mysql-5.7.43-winx64\data
# 允许最大连接数
max_connections = 200

server-id = 1
# 提前创建 logs/
log_bin = F:\mysql-5.7.43-winx64\logs\mysql-bin.log
binlog_do_db = sync_vm
sync_binlog = 1
[mysql]
default-character-set = UTF8MB4
[mysql.server]
default-character-set = UTF8MB4
[mysqld_safe]
default-character-set = UTF8MB4
[client]
default-character-set = UTF8MB4
```

#### 2.2. 重启后执行命令

```sql
-- 配置主从同步用户
GRANT
REPLICATION
SLAVE ON *.* TO 'slave_root'@'%' IDENTIFIED BY 'slave_mysql20170419';
FLUSH
PRIVILEGES;
-- 查看 File, Position
SHOW
MASTER STATUS;
```

### 3. 配置从库

#### 3.1. 从库配置文件

***/etc/my.cnf***

```properties
[mysqld]
character-set-server = UTF8MB4
port = 3307
# 设置安装目录
basedir = /usr/mysql/home
# 设置数据存放目录
datadir = /usr/mysql/home/data
# 允许最大连接数
max_connections = 200
user = mysql
socket = /usr/mysql/home/mysqld.sock
log-error = /usr/mysql/home/mysqld.log
pid-file = /usr/mysql/home/mysqld.pid

server-id = 2
# 不能加 logs/
relay-log = /usr/mysql/home/mysql-relay-bin.log
# 不能加 logs/
log_bin = /usr/mysql/home/mysql-bin.log
binlog_do_db = sync_vm
read_only = 1

[mysql]
default-character-set = UTF8MB4
socket = /usr/mysql/home/mysqld.sock

[client]
default-character-set = UTF8MB4
socket = /usr/mysql/home/mysqld.sock
```

#### 3.2. 重启后执行命令

```sql
-- 设置主服务器信息
CHANGE
MASTER TO MASTER_HOST = 'vm0.yejh.cn',
    MASTER_PORT = 3307,
    MASTER_USER = 'root',
    MASTER_PASSWORD = 'mysql20170419',
    MASTER_LOG_FILE = 'mysql-bin.000002',
    MASTER_LOG_POS = 154;
-- 启停主从同步
START
SLAVE USER = 'slave_root' PASSWORD = 'slave_mysql20170419';
STOP
SLAVE;
-- 查看 Slave_IO_Running, Slave_SQL_Running
SHOW
SLAVE STATUS;
```