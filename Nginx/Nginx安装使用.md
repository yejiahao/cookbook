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
    ./configure --prefix=${instPath}/$1 --with-http_ssl_module --with-http_stub_status_module
    make && make install
    # delete source folder
    rm -rf $HOME/$1/
    # (create|update) symbolic link
    ln -snf $1/ ${instPath}/home

    exit 0
fi

echo "Usage: $0 [version(e.g., nginx-1.16.0)]"
```

### 配置环境变量
***/etc/profile (source /etc/profile)***
```properties
# set nginx profile
NGINX_HOME=/usr/nginx/home
PATH=$PATH:$NGINX_HOME/sbin
export PATH NGINX_HOME
```

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