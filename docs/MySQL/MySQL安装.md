## Windows 10 安装[ MySQL](https://www.mysql.com/cn/)

> The world's most popular open source database

### 1. 下载并解压 mysql-8.0.16-winx64.zip

### 2. 配置环境变量

| 变量         | 值                      |
|------------|------------------------|
| MYSQL_HOME | D:\mysql-8.0.16-winx64 |
| Path       | ...;%MYSQL_HOME%\bin   |

### 3. 查看 MySQL 版本

```
C:\WINDOWS\system32>mysqld --version
D:\mysql-8.0.16-winx64\bin\mysqld.exe  Ver 8.0.16 for Win64 on x86_64 (MySQL Community Server - GPL)
```

### 4. 根目录生成 data 文件夹

~~mysqld --initialize-insecure --user=mysql # V5.7.29~~

```
C:\WINDOWS\system32>mysqld --initialize --console
2019-06-27T08:57:52.722441Z 0 [System] [MY-013169] [Server] D:\mysql-8.0.16-winx64\bin\mysqld.exe (mysqld 8.0.16) initializing of server in progress as process 7560
2019-06-27T08:58:12.928744Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: nlkiBaL%G8jt
2019-06-27T08:58:20.361624Z 0 [System] [MY-013170] [Server] D:\mysql-8.0.16-winx64\bin\mysqld.exe (mysqld 8.0.16) initializing of server has completed
```

### 5. 添加初始化文件

***%MYSQL_HOME%\my.ini***

```properties
[mysqld]
character-set-server = UTF8MB4
# 绑定 IPv4 和 3306 端口
bind-address = 0.0.0.0
port = 3306
# 设置 mysql 的安装目录
basedir = D:\mysql-8.0.16-winx64
# 设置 mysql 数据库的数据的存放目录
datadir = D:\mysql-8.0.16-winx64\data
# 允许最大连接数
max_connections = 200
[mysql]
default-character-set = UTF8MB4
[mysql.server]
default-character-set = UTF8MB4
[mysqld_safe]
default-character-set = UTF8MB4
[client]
default-character-set = UTF8MB4
```

### 6. 安装 MySQL 服务

```
C:\WINDOWS\system32>mysqld install MySQL --defaults-file="%MYSQL_HOME%\my.ini" # DOS管理员
C:\WINDOWS\system32>mysqld install MySQL --defaults-file="$env:MYSQL_HOME\my.ini" # PowerShell管理员
Service successfully installed.
```

### 7. 启动 MySQL

```
C:\WINDOWS\system32> NET START MySQL
MySQL 服务正在启动 ..
MySQL 服务已经启动成功。

```

### 8. 修改密码

```
C:\WINDOWS\system32>mysql -P 3306 -u root -p
```

~~Enter password:~~

~~mysql> use mysql;~~

~~mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('mysql20170419'); # V5.7.29~~

```
Enter password: ************
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.16

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'mysql20170419';
Query OK, 0 rows affected (0.05 sec)

mysql> EXIT;
Bye
```

***

### 删除 MySQL 服务

```
C:\WINDOWS\system32> NET STOP MySQL
MySQL 服务正在停止.
MySQL 服务已成功停止。

C:\WINDOWS\system32>mysqld --remove MySQL
Service successfully removed.
```