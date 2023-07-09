2021-02-26 新心数科

* **一面**
    - 可重复读跟读提交区别
    - gap 锁，update 带不带唯一索引有什么不同
    - 聚簇索引、非聚簇索引区别
    - `EXPLAIN` 后看什么字段，`file sort` 的原因
    - 事务有哪些传播属性，`require`, `require new` 区别，`@Transaction` 默认用哪个
    - `ThreadLocalMap` 源码
    - `String` 为什么不可变
    - Spring 动态代理哪些，区别
    - Spring `init-method` 对应哪个方法 **（afterPropertiesSet，问得有问题）**
    - JDK 线程池的 `coreSize`, `maxSize`, `queueLength` 解释，IO密集型跟CPU密集型时如何配置这些 size
    - Netty 的 Boss 跟 worker 分别做什么
    - 短信网关怎么做负载均衡

* **总监**
    - 表字段 `stud_id, course_id, score`，找出总分前三的学生；进阶：找出总分前三的学生中，每一科都要 60 分或以上
  ```sql
  SELECT stud_name
  FROM course
  WHERE stud_name NOT IN (SELECT stud_name FROM course WHERE score < 60)
  GROUP BY stud_name
  ORDER BY SUM(score) DESC LIMIT 3;
  ```
    - 做一个微信朋友圈，设计思路

***
2021-03-20 名创优品

* **一面**
    - 面向对象三大特性解读
    - 索引的好处坏处
    - Spring IOC 怎么理解
    - 了解什么排序，冒泡排序的实现
    - 如何理解 `ElasticSearch`
    - `ElasticSearch` 怎么做分页
    - 对数据建仓的理解

***
2021-03-22 亿阳信通

* **一面**
    - `Redis` 特性及 `Redis` 过期逻辑
    - 项目中用 `Redis` 做什么
    - 项目中并发量，遇到过什么问题，如何解决
    - 事务死锁、事务超时解释
    - 进程中获取一条几百个字段的数据，如何处理

***
2022-12-06 字节跳动

* **一面**
    - push 通道延时发送怎么实现，有没有用过时间轮实现
    - push 通道基于峰值，是怎么设计出来的
    - push 通道 http 连接泄漏如何处理
    - 客户端连 `Redis` 哨兵怎么往下找到主从节点
    - `Redis` 的数据类型有哪些，ZSET 底层的数据结构是什么，为什么用 skiplist 不用 B+ tree
    - `RocketMQ` 高性能的原因，`RokcetMQ` 有哪些组件
    - `RocketMQ` 的事务，有没有了解
    - `RocketMQ` 怎么做到不丢消息
    - `JDK7` `JDK8` 的区别 **（jjs那块，map链表红黑树，永久代，metaspace）**
    - 一个 Java 对象如何让它释放句柄这些系统资源
    - 编程题：实现一个 LRU 类

***
2022-12-16 CVTE

* **一面**
    - 生成 yytoken 同步返回给客户端是怎么实现的
        - **yy的自研框架，内部异步处理后带上上下文的连接 id，sdk 视觉上是同步**
    - push 通道日常的工作内容是哪些
    - `RocketMQ` 延时队列怎么实现的，删除消息的策略有哪些
    - `RocketMQ` 消费者消费轮训的方式和原理是怎样的
    - `RocketMQ` commitLog 为什么固定大小，能否是 2g | 3g | 4g
    - `Clickhouse OLAP` 和列式存储怎么理解

***
2023-02-15 滴普科技

* **一面**
    - CP, AP 什么场景使用
    - 解释 BASE 理论
    - JMM Java 内存模型
    - push 通道 `Redis` 哨兵部署方式
    - `Zookeeper` 如何保证一致性
    - ZSET 底层数据结构，跳表的结构

***
2023-02-20 虎牙科技

* **一面**
    - 阻塞非阻塞队列区别，举例
    - `@Around` 实现短路超时返回
    - 普通方法 A 调内部事务方法 B，事务不生效，为什么
    - `Spring` 能否解决构造器循环依赖，原因
    - `MySQL redolog，undolog，binlog` 流程解析
    - 先 `MySQL`，后 `MQ`，分布式事务一致性如何保证
    - `MySQL` 主主有没有部署过
    - ```UPDATE t SET 唯一索引 = x WHERE id = y```，与其它事务 sql 并行产生死锁的原因
    - 如何实现轮训算法，进阶：带权重的轮训算法
    - 黑客缓存穿透，如何解决
    - 直播弹幕笛卡尔积，高并发如何处理