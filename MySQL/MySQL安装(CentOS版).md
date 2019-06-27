### 查看系统版本
```
[root@localhost ~]# uname -a
Linux localhost.yejh 2.6.32-754.el6.x86_64 #1 SMP Tue Jun 19 21:26:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
[root@localhost ~]# cat /etc/issue
CentOS release 6.10 (Final)
Kernel \r on an \m

```

### 安装依赖
```
[root@localhost ~]# yum -y install numactl
```

### 创建用户
```
[root@localhost ~]# useradd mysql
```

### 下载并解压 MySQL glibc 安装包
```
[root@localhost ~]# wget -c -P $HOME/upload/ https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.16-linux-glibc2.12-x86_64.tar.xz
[root@localhost ~]# mkdir -pv /usr/mysql/
mkdir: created directory `/usr/mysql/'
[root@localhost ~]# tar -Jxvf $HOME/upload/mysql-8.0.16-linux-glibc2.12-x86_64.tar.xz -C /usr/mysql/
[root@localhost ~]# ln -snf mysql-8.0.16-linux-glibc2.12-x86_64/ /usr/mysql/home
```

### 设置环境变量
***/etc/profile (source /etc/profile)***
```properties
# set mysql profile
MYSQL_HOME=/usr/mysql/home
PATH=$PATH:$MYSQL_HOME/bin
export PATH MYSQL_HOME
```

### 查看 MySQL 版本
```
[root@localhost ~]# mysqld --version
/usr/mysql/mysql-8.0.16-linux-glibc2.12-x86_64/bin/mysqld  Ver 8.0.16 for linux-glibc2.12 on x86_64 (MySQL Community Server - GPL)
```

### 修改配置文件
***/etc/my.cnf***
```properties
[mysqld]
character-set-server = UTF8MB4
port = 3306
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

[mysql]
default-character-set = UTF8MB4
socket = /usr/mysql/home/mysqld.sock

[client]
default-character-set = UTF8MB4
socket = /usr/mysql/home/mysqld.sock
```

### 初始化 MySQL
```
[root@localhost ~]# chown -R mysql:mysql $MYSQL_HOME/
[root@localhost ~]# mysqld -I
[root@localhost ~]# cat $MYSQL_HOME/mysqld.log
2019-06-27T08:23:26.313949Z 0 [System] [MY-013169] [Server] /usr/mysql/mysql-8.0.16-linux-glibc2.12-x86_64/bin/mysqld (mysqld 8.0.16) initializing of server in progress as process 1246
2019-06-27T08:23:30.934768Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: sf)mBgczk4ae
2019-06-27T08:23:34.528672Z 0 [System] [MY-013170] [Server] /usr/mysql/mysql-8.0.16-linux-glibc2.12-x86_64/bin/mysqld (mysqld 8.0.16) initializing of server has completed
```

### 启动 MySQL
```
[root@localhost ~]# mysqld_safe -D
2019-06-27T08:24:05.565399Z mysqld_safe Logging to '/usr/mysql/home/mysqld.log'.
2019-06-27T08:24:05.610734Z mysqld_safe Starting mysqld daemon with databases from /usr/mysql/home/data
2019-06-27T08:24:06.936967Z mysqld_safe A mysqld process with pid=1484 is already running. Aborting!!
```

### 停止 MySQL
```
[root@localhost ~]# cat $MYSQL_HOME/mysqld.pid | xargs kill -9
```

### 开机自启动
```
[root@localhost ~]# cp $MYSQL_HOME/support-files/mysql.server /etc/init.d/mysql
[root@localhost ~]# chkconfig --add mysql
[root@localhost ~]# chkconfig --level 245 mysql off
[root@localhost ~]# service mysql
Usage: mysql  {start|stop|restart|reload|force-reload|status}  [ MySQL server options ]
```

### 修改初始密码
```
[root@localhost ~]# mysql -h localhost -u root -p'sf)mBgczk4ae'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.16

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '20170419';
Query OK, 0 rows affected (0.02 sec)

mysql> RENAME USER 'root'@'localhost' TO 'root'@'%';
Query OK, 0 rows affected (0.01 sec)

mysql> EXIT;
Bye
```