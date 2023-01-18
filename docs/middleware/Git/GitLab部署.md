### 下载部署包

```sh
[root@vm2 ~]# wget -c https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el6/gitlab-ce-13.2.4-ce.0.el6.x86_64.rpm
```

### 安装

```sh
[root@vm2 ~]# yum install -y curl policycoreutils-python openssh-server cronie
[root@vm2 ~]# service postfix start
[root@vm2 ~]# chkconfig postfix on
[root@vm2 ~]# rpm -ivh gitlab-ce-13.2.4-ce.0.el6.x86_64.rpm
```

### 配置及运行

***/etc/gitlab/gitlab.rb***

```properties
external_url 'http://vm2.yejh.cn:9999'
gitlab_rails['time_zone'] = 'Asia/Shanghai'
```

```sh
[root@vm2 ~]# gitlab-ctl reconfigure && gitlab-ctl start
```

### 查看状态

```sh
[root@vm2 ~]# gitlab-ctl status
run: alertmanager: (pid 3544) 146s; run: log: (pid 3017) 488s
run: gitaly: (pid 3526) 152s; run: log: (pid 2269) 906s
run: gitlab-exporter: (pid 3498) 155s; run: log: (pid 2870) 586s
run: gitlab-workhorse: (pid 3483) 158s; run: log: (pid 2691) 691s
run: grafana: (pid 3558) 145s; run: log: (pid 3356) 254s
run: logrotate: (pid 2772) 646s; run: log: (pid 2782) 643s
run: nginx: (pid 2740) 668s; run: log: (pid 2750) 664s
run: node-exporter: (pid 3490) 157s; run: log: (pid 2837) 612s
run: postgres-exporter: (pid 3551) 146s; run: log: (pid 3083) 452s
run: postgresql: (pid 2413) 878s; run: log: (pid 2436) 873s
run: prometheus: (pid 3506) 154s; run: log: (pid 2948) 528s
run: puma: (pid 2603) 741s; run: log: (pid 2612) 738s
run: redis: (pid 2199) 925s; run: log: (pid 2207) 922s
run: redis-exporter: (pid 3500) 155s; run: log: (pid 2909) 559s
run: sidekiq: (pid 2632) 725s; run: log: (pid 2643) 722s
```

### 卸载 GitLab

```sh
[root@vm2 ~]# gitlab-ctl stop
[root@vm2 ~]# gitlab-ctl cleanse
[root@vm2 ~]# gitlab-ctl uninstall

[root@vm2 ~]# rm -rf /opt/gitlab/ && rm -f /usr/bin/gitlab-* && rm -rf /dev/shm/gitlab/
```

### 移除旧 rpm 包

```sh
[root@vm2 ~]# rpm -qa | grep gitlab | xargs rpm -e
```