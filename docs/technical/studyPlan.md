##### 2020/03/09

1. `MySQL` 索引，`LIMIT` 分页优化
2. ~~布隆过滤器~~
3. ~~`synchronized` 改变锁对象，现象验证~~: **线程到达 synchronized 代码块时，检查引用的对象地址**
4. `ReentrantReadWriteLock`
5. ~~[rocketmq-dashboard](https://github.com/apache/rocketmq-dashboard) 部署~~
6. ~~ump v6.0.0 部署~~
7. ~~`Maven` 阿里云仓库更换~~: **已更换为 https://maven.aliyun.com/repository/central**

##### 2020/03/16

1. `JVM` 加载生命周期
2. ~~`synchronized` 偏向锁~~

##### 2020/03/23

1. ~~ump v6.0.0 mtreceiver 模块逻辑~~: **已输出文档**
2. [Redis cluster](https://redis.io/docs/management/scaling/)
3. ~~`String.split()` 验证~~

##### 2020/03/30

1. [redisson](https://redisson.org/) 编写例子运行
2. 《RocketMQ技术内幕：RocketMQ架构设计与实现原理（第2版）.pdf》阅读
3. ~~《MySQL 索引及优化实战.pdf》阅读~~: **（性能高 -> 低）explain 中的 type 是 const, ref, range, index, ALL**
4. `MySQL` 是如何解决幻读的

##### 2020/04/07

1. ~~http code 301 和 302 区别~~: **301 Permanently Moved, 302 Temporarily Moved**
2. ~~[Nginx 配置 https 证书](https://nginx.org/en/docs/http/configuring_https_servers.html)~~:

- 生成 SSL
  证书 `openssl req -x509 -nodes -days 36500 -newkey rsa:2048 -keyout /usr/nginx/nginx.key -out /usr/nginx/nginx.crt`
- 修改 _nginx.conf_ 的 http.server
  配置 `ssl_certificate /usr/nginx/nginx.crt; ssl_certificate_key /usr/nginx/nginx.key;`

##### 2020/04/26

1. `TCP` 结构
2. ~~[Wireshark](https://www.wireshark.org/) 使用~~: **抓包，看协议**
3. ~~ump v6.0.0 centrum 模块逻辑~~: **已输出文档**
4. ~~小（大）顶堆 `PriorityQueue` 使用~~: **offer() 通过 (k - 1) >>> 1 找到 parent, poll() 通过 (k << 1) + 1 + 1 找到左右
   child; {@see #siftUp #siftDown}**

##### 2020/05/06

1. [Netty](https://netty.io/) pineline 分析

##### 2020/05/25

1. ~~`CyclicBarrier, CountDownLatch, Semaphore` 底层研究~~: **CyclicBarrier 使用了 #breakBarrier 和 #nextGeneration，没有使用
   AQS**

##### 2020/06/15

###### JVM 专场

1. 介绍下垃圾收集机制（在什么时候，对什么，做了什么）
2. ~~垃圾收集有哪些算法，各自的特点~~
3. ~~类加载的过程~~: **1.加载(Loading)读二进制字节流 2.验证(Verification)魔数版本号 3.准备(Preparation)静态成员赋零值
   4.解析(Resolution)常量池中符号引用->直接引用 5.初始化(Initialization)执行 `<clinit>()` 5种时机 6.使用(Using) 7.卸载(
   Unloading)**
4. ~~双亲委派模型~~: **`ClassLoader#loadClass`
   如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请求（找不到所需的类）时，子加载器才会尝试自己去加载。像 `java.lang.Object`
   这些存放在 rt.jar
   中的类，无论使用哪个类加载器加载，最终都会委派给最顶端的启动类加载器加载，从而使得不同加载器加载的 `Object`
   类都是同一个。相反，如果没有使用双亲委派模型，由各个类加载器自行去加载的话，如果用户自己编写了一个称为 `java.lang.Object`
   的类，并放在 classpath 下，那么系统将会出现多个不同的 `Object` 类，Java 类型体系中最基础的行为也就无法保证。**
5. ~~有哪些类加载器~~: **启动类加载器 `Bootstrap ClassLoader`，扩展类加载器 `Extension ClassLoader`
   ，应用程序类加载器 `Application ClassLoader`，自定义类加载器**

##### 2020/06/29

1. ~~[tcpdump](https://www.tcpdump.org/) 使用~~:

- 安装前置依赖包 `yum flex bison, make libpcap, make tcpdump`
- 抓包 `tcpdump -i eth0 host vm0.yejh.cn -w a.pcapng`
- 使用 Wireshark 打开文件

##### 2020/09/07

1. ~~Java 8 Lambda 闭包的理解~~: **传入参数与方法参数一致，如 `list.stream().filter(Objects::nonNull)`**

##### 2020/12/16

1. ~~[RocketMQ](https://rocketmq.apache.org/) 双主双从搭建~~:

?> `echo -e 'storePathRootDir=<dir>\nlistenPort=<port>\nenablePropertyFilter=<filter>' >> <file>`

```properties
# $ROCKETMQ_HOME/conf/2m-2s-async/broker-a.properties
storePathRootDir=/root/store/broker-a
listenPort=10911
enablePropertyFilter=true
# $ROCKETMQ_HOME/conf/2m-2s-async/broker-a-s.properties
storePathRootDir=/root/store/broker-a-s
listenPort=10921
enablePropertyFilter=true
# $ROCKETMQ_HOME/conf/2m-2s-async/broker-b.properties
storePathRootDir=/root/store/broker-b
listenPort=20911
enablePropertyFilter=true
# $ROCKETMQ_HOME/conf/2m-2s-async/broker-b-s.properties
storePathRootDir=/root/store/broker-b-s
listenPort=20921
enablePropertyFilter=true
```

##### 2020/12/17

1. ~~BitSet 源码分析~~: **最重要的两行代码**

```java
/**
 * {@see #set}
 */
words[wordIndex] |= (1L << bitIndex); // Restores invariants
/**
 * {@see #clear}
 */
words[wordIndex] &= ~(1L << bitIndex);
```

##### 2021/01/06

1. ~~MySQL 死锁分析~~: **DELETE IN (主键)
   ，会自动排序，但产生间隙锁 `SHOW ENGINE INNODB STATUS;`，`SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX;`**

##### 2021/01/07

1. ~~try-with-resource 语法糖分析~~: **构建三层 try-catch 块，位置分别是 ①创建对象 ②业务逻辑 ③对象.close()；当 ②, ③
   都出现异常时，使用 `Throwable#addSuppressed` 抑制异常**

##### 2021/01/18

1. ~~Redis 批量删除 50W key 使用什么命令~~:

- xargs `redis-cli KEYS '*' | xargs redis-cli DEL`
- Lua
  EVAL `redis-cli EVAL "local keys = redis.call('keys', ARGV[1]) for i=1,#keys,5000 do redis.call('del', unpack(keys, i, math.min(i+4999, #keys))) end return #keys" 0 '*'`

##### 2021/03/14

1. ~~Redis 数据类型 -> 结构对应关系~~: **string -> SDS, list -> 链表 + 压缩列表, set -> 整数集合 + 字典, map ->
   压缩列表 + 字典, zset -> 压缩列表 + 跳跃表**

##### 2021/03/15

1. ~~InnoDB 意向锁~~:

| 锁类型 | X   | IX  | IS  | S   |
|-----|-----|-----|-----|-----|
| X   | ✖   | ✖   | ✖   | ✖   |
| IX  | ✖   | ✔   | ✔   | ✖   |
| IS  | ✖   | ✔   | ✔   | ✔   |
| S   | ✖   | ✖   | ✔   | ✔   |

- InnoDB 支持多粒度锁，特定场景下，行级锁可以与表级锁共存
- 意向锁之间互不排斥，但除了 IS 与 S 兼容外，意向锁会与 共享锁 / 排他锁 互斥
- IX，IS 是表级锁，不会和行级的 X，S 锁发生冲突。只会和表级的 X，S 发生冲突。
- 意向锁在保证并发性的前提下，实现了行锁和表锁共存且满足事务隔离性的要求。

2. ~~MyISAM 和 InnoDB 区别~~:

- InnoDB 支持事务，MyISAM 不支持事务。这是 MySQL 将默认存储引擎从 MyISAM 变成 InnoDB 的重要原因之一
- InnoDB 支持外键，而 MyISAM 不支持。对一个包含外键的 InnoDB 表转为 MYISAM 会失败
- InnoDB 是聚集索引，MyISAM 是非聚集索引。聚簇索引的文件存放在主键索引的叶子节点上，因此 InnoDB
  必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，因为主键太大，其他索引也都会很大。而
  MyISAM 是非聚集索引，数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的
- InnoDB 不保存表的具体行数，执行 `SELECT COUNT(*) FROM table` 时需要全表扫描。而 MyISAM
  用一个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快
- InnoDB 最小的锁粒度是行锁，MyISAM 最小的锁粒度是表锁。一个更新语句会锁住整张表，导致其他查询和更新都会被阻塞，因此并发访问受限。这也是
  MySQL 将默认存储引擎从 MyISAM 变成 InnoDB 的重要原因之一

##### 2021/03/16

1. ~~Redis 队列注意事项~~: **RPOPLPUSH 做 ACK 机制，防止丢消息；BLPOP | BRPOP 带时延阻塞，支持查询多个 list**

2. Spring 为什么用三级缓存

3. ~~海量数据处理解决方案~~: **①Hash 后分治 ②大小顶堆（PriorityQueue） ③位图法（BitSet） ④前缀树**

##### 2021/03/17

1. [ZooKeeper](https://zookeeper.apache.org/) 选举

##### 2023/01/16

1. ~~[docsify](https://docsify.js.org/) 构建在线 GitPage~~

##### 2023/01/17

1. [Protocol Buffers](https://developers.google.com/protocol-buffers) 跨语言联调

##### 2023/01/18

1. 了解 Raft 共识算法
2. [Disruptor](https://github.com/LMAX-Exchange/disruptor) 初探