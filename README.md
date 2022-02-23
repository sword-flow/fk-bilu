# fk-bilu
* Programmer Interview Questions

## 目录
* 计算机系统
* 计算机网络
* 数据库
* 语言
* 算法

---

# 八股文索引

---

# 编程语言
## Go
### goroutine 并发模型
G-M-P 模型
G: goroutine 逻辑线程， 保存栈，状态，id, 函数等信息。G只有绑定了P才能够被调度
M: OS 系统级别线程,绑定有效的P之后，进入调度
P: 逻辑处理器，保留各种队列G， 用来调度P和协调M的关系，对于G，P相当于cpu核心，而对于M，则相当于上下文
+ 如果P中的G用完了怎么办？
  - 线程窃取
### context
+ ConcurrentMap
  - Map为什么线程不安全？
  - Concurrent Map 如何实现？
  - 什么是自旋锁？
  - Map + mutex 读写锁实现有何坏处？
+ 互斥锁/读写锁
  - mutex的实现原理
  - 如何为锁加超时？
+ 垃圾回收
  - Go语言如何进行的垃圾回收？
+ Grpc 实现
## Python
+ async/await

# 数据库
## Mysql
+ B+树索引
+ 悲观锁/乐观锁
+ 事务隔离级别
+ 分表分库
  - 分表分库的区别
  - 跨数据库分表注意什么？
  - 跨库分表分布式事务的常用实现
    - 直接答TCC
+ 索引
+ 事务
+ 分布式事务
+ 死锁

# 消息队列
+ RabbitMQ
+ amqp 协议

# 缓存
## Redis
+ 数据结构
  - [sds](./数据库/Redis.md)
+ 缓存穿透
+ 哨兵模式
+ 分布式
+ 一致性hash
+ hash槽

# 操作系统
+ 进程与线程
+ 线程同步
+ 进程通信
+ 系统信号
+ 进程调度策略


# 网络
+ select/epoll/poll
  - select与epoll的区别
  - 与传统线程的区别
+ TCP/UDP
+ 流量控制
+ TCP拥塞控制
  - [拥塞控制](./计算机网络/TCP拥塞控制.md)

# 数据结构与算法
## 数据结构
+ HashMap
  - 扩容
+ Array
  - 扩容
+ LinkedList
+ 图
+ 树
+ 算法

