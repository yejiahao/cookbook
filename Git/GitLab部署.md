### 下载部署包
```
[root@vm2 ~]# wget -c https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el6/gitlab-ce-12.10.6-ce.0.el6.x86_64.rpm
```

### 安装
```
[root@vm2 ~]# rpm -ivh gitlab-ce-12.10.6-ce.0.el6.x86_64.rpm
```

### 配置及运行
***/etc/gitlab/gitlab.rb***
```properties
external_url 'http://vm2.yejh.cn:9999'
gitlab_rails['time_zone'] = 'Asia/Shanghai'
```

```
[root@vm2 ~]# gitlab-ctl reconfigure && gitlab-ctl start
```

### 查看状态
```
[root@vm2 ~]# gitlab-ctl status
run: alertmanager: (pid 3743) 3543s; run: log: (pid 2939) 3870s
run: gitaly: (pid 3753) 3543s; run: log: (pid 2179) 4268s
run: gitlab-exporter: (pid 3760) 3542s; run: log: (pid 2794) 3958s
run: gitlab-workhorse: (pid 3763) 3542s; run: log: (pid 2630) 4063s
run: grafana: (pid 3771) 3542s; run: log: (pid 3385) 3646s
run: logrotate: (pid 3780) 3541s; run: log: (pid 2706) 4015s
run: nginx: (pid 3794) 3541s; run: log: (pid 2675) 4036s
run: node-exporter: (pid 3797) 3541s; run: log: (pid 2758) 3984s
run: postgres-exporter: (pid 3801) 3541s; run: log: (pid 2992) 3833s
run: postgresql: (pid 3812) 3540s; run: log: (pid 2347) 4235s
run: prometheus: (pid 3814) 3540s; run: log: (pid 2868) 3905s
run: redis: (pid 3831) 3540s; run: log: (pid 2127) 4289s
run: redis-exporter: (pid 3837) 3539s; run: log: (pid 2831) 3932s
run: sidekiq: (pid 3844) 3538s; run: log: (pid 2583) 4089s
run: unicorn: (pid 3859) 3537s; run: log: (pid 2551) 4105s
```