## Windows 10 安装[ PostgreSQL](https://www.postgresql.org/)

> PostgreSQL is a powerful, open source object-relational database system with over 35 years of active development that
> has earned it a strong reputation for reliability, feature robustness, and performance.

### 1. 下载并解压 postgresql-12.14-1-windows-x64-binaries.zip

### 2. Windows Terminal 下安装（管理员模式）

```bash
PS C:\WINDOWS\system32> D:
PS D:\> cd .\pgsql\bin\
PS D:\pgsql\bin> .\initdb.exe -D "D:\pgsql\data" -E UTF8 -U postgres -A password --locale=C -W
属于此数据库系统的文件宿主为用户 "jiahao.ye".
此用户也必须为服务器进程的宿主.
数据库簇将使用本地化语言 "C"进行初始化.
缺省的文本搜索配置将会被设置到"english"

禁止为数据页生成校验和.

输入新的超级用户口令:postgresql20170419
再输入一遍:postgresql20170419

创建目录 D:/pgsql/data ... 成功
正在创建子目录 ... 成功
选择动态共享内存实现 ......windows
选择默认最大联接数 (max_connections) ... 100
选择默认共享缓冲区大小 (shared_buffers) ... 128MB
selecting default time zone ... Asia/Shanghai
创建配置文件 ... 成功
正在运行自举脚本 ...成功
正在执行自举后初始化 ...成功
同步数据到磁盘...成功

成功。您现在可以用下面的命令开启数据库服务器：

    ^"D^:^\pgsql^\bin^\pg^_ctl^" -D ^"D^:^\pgsql^\data^" -l 日志文件 start

```

### 3. 其它指令

```bash
# 注册服务
PS D:\pgsql\bin> .\pg_ctl.exe register -D "D:\pgsql\data" -PostgreSQL

# 启动服务
PS D:\pgsql\bin> NET START PostgreSQL
PostgreSQL 服务正在启动 .
PostgreSQL 服务已经启动成功。

# 停止服务
PS D:\pgsql\bin> NET STOP PostgreSQL
PostgreSQL 服务正在停止.
PostgreSQL 服务已成功停止。

# 注销服务
PS D:\pgsql\bin> .\pg_ctl.exe unregister -PostgreSQL
```