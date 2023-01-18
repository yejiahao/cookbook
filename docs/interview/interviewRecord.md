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