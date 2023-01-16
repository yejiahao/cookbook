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