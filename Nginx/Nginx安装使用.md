### 查看系统版本
```
[root@vm2 ~]# uname -a
Linux localhost.yejh 2.6.32-754.el6.x86_64 #1 SMP Tue Jun 19 21:26:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
[root@vm2 ~]# cat /etc/issue
CentOS release 6.10 (Final)
Kernel \r on an \m

```

### CentOS 安装 Nginx
***ngx.inst.sh***
```sh
#!/bin/sh

package=$1.tar.gz
instPath=/usr/nginx

if [[ "$1" != "" ]]
then
    yum -y install pcre-devel zlib-devel openssl-devel
    # download
    echo "downloading ${package}"
    wget -c -P $HOME/upload/ http://nginx.org/download/${package}
    echo "downloaded ${package}"
    # extract and install
    tar -zxvf $HOME/upload/${package} -C $HOME/ && cd $HOME/$1/
    # refer to http://nginx.org/en/docs/configure.html
    ./configure --prefix=${instPath}/$1 --with-http_ssl_module --with-http_stub_status_module --with-stream
    make && make install
    # delete source folder
    rm -rf $HOME/$1/
    # (create|update) symbolic link
    ln -snf $1/ ${instPath}/home

    exit 0
fi

echo "Usage: $0 [version(e.g., nginx-1.18.0)]"
```

### 配置环境变量
***/etc/profile (source /etc/profile)***
```properties
# set nginx profile
NGINX_HOME=/usr/nginx/home
PATH=$PATH:$NGINX_HOME/sbin
export PATH NGINX_HOME
```

### Nginx 操作文档
[官方文档](http://nginx.org/en/docs/)

### Nginx 基本指令
```sh
nginx -V # 查看版本
nginx -t # 验证配置
nginx [-c $NGINX_HOME/conf/nginx.conf] # 启动服务
nginx -s reopen # 打开日志
nginx -s reload # 重载配置
nginx -s quit # 正常关闭
nginx -s stop # 快速关闭
```

***

### 开机自启动

#### 添加启动文件

***/etc/init.d/nginxd***
```sh
#!/bin/sh

# chkconfig: 3 10 90
# description: nginx is an HTTP and reverse proxy server, a mail proxy server, and a generic TCP/UDP proxy server.

nginx=/usr/nginx/home/sbin/nginx

case "$1" in
start)
  # shellcheck disable=SC2039
  echo -n "Starting Nginx"
  $nginx
  echo " done"
  ;;
stop)
  # shellcheck disable=SC2039
  echo -n "Stopping Nginx"
  $nginx -s quit
  echo " done"
  ;;
forceStop)
  # shellcheck disable=SC2039
  echo -n "Stopping(force) Nginx"
  $nginx -s stop
  echo " done"
  ;;
restart)
  ${0} stop
  ${0} start
  ;;
reload)
  # shellcheck disable=SC2039
  echo -n "Reloading Nginx"
  $nginx -s reload
  echo " done"
  ;;
test)
  $nginx -t
  ;;
version)
  $nginx -V
  ;;
show)
  # shellcheck disable=SC2009
  ps -ef | grep nginx | grep -v grep
  ;;
*)
  echo "Usage: ${0} {start|stop|forceStop|restart|reload|test|version|show}" >&2
  exit 1
  ;;
esac
```

#### 配置启动
```
[root@vmx ~]# chmod +x /etc/init.d/nginxd
[root@vmx ~]# chkconfig --add nginxd
[root@vmx ~]# chkconfig --list nginxd
nginxd          0:off   1:off   2:off   3:on    4:off   5:off   6:off
[root@vmx ~]# service nginxd
Usage: /etc/init.d/nginxd {start|stop|forceStop|restart|reload|test|version|show}
```