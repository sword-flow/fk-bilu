## Redis



### 基本数据结构

#### SDS结构

```c
struct sdshdr {
  int len;   // SDS中保存的字符串长度，即buf中已使用的字节数量
  int free;   // 未使用的字节数量，buf中未使用的字节数量
  char buf[];  // 字节数组，实际保存了字符串；字符的结尾是'\0'，也就是比len多一个字符
}
```

使用SDS的好处：

* 常数时间复杂度内获取到字符串长度
* API安全，不会造成缓冲区溢出，而是会进行空间预分配，步骤：
  * 如果修改SDS后，len长度小于1MB，就会分配再次分配len大小的空间
  * 如果大于1MB，那么会再次分配1MB的空间
  * 总而言之，最多会再次分配1MB的空间给SDS
* 惰性空间释放，字符串减少空间时，不会立即释放空间，只是修改free和len变量
* 可以保存文本或者二进制数据



#### 链表

链表底层的节点ListNode是一个双端无环链表

```c
struct listNode {
  struct listNonde *prev;  // 常数时间复杂度内获取前置和后置节点
  struct listNonde *next;
  void * value;
}listNode;
```

链表的list结构提供了更多的参数

```c
typedef struct list {
  listNode *head;  // 可以在常数时间复杂度内或者链表的头尾节点
  listNode *tail;
  unsigned long len;  // 链表的节点数量，也就是长度
  void *(*dup) (void *ptr);  // 复制节点
  void *(*free) (void *ptr); // 释放节点
  void *(*match) (void *ptr, void *key);  // 节点值比对
}
```



#### 字典

字典底层是哈希表实现的

```c
typedef struct dict {
  dictType *type;  // 一些操作函数
  void *privdata;
  // 这里是字典的哈希表底层，有2个哈希表，dictht结构体底层用的是哈希表节点dictEntry，是一个链表结构，除此之外，dictht还存了哈希表大小和已有节点数量等参数
  // dictEntry之所以要用链表结构，是为了解决哈希冲突，有冲突时，会将新值插入到dictEntry的链表头位置，实现了常数时间复杂度
  dictht ht[2];
  int trehashidx;  // 渐进式rehash的索引，默认是-1
}
```

<img src="./普通状态下的字典.png" alt="普通状态下的字典"/>

##### 渐进式rehash

扩张或者收缩哈希表时，使用渐进式rehash的方式，将ht[0]里的键值对迁移到ht[1]，步骤如下：

* 为ht[1]分配空间
* 设置rehashidx=0，表示rehash开始
* 进行增删改查操作时，将rehashidx索引上的所有简直对迁移到ht[1]，之后增加rehashidx
* 完成后设置rehashidx=-1

rehash期间，新增的键值对会放到ht[1]中，删除查找和更新操作会依次在两个ht上进行



#### 跳表

跳表时有序集合zset的底层实现之一，结构是zskiplist -> zskiplistNode，其中zskiplist表村了头尾节点长度，zskiplistNode记录了节点本身，跳表层高是1～32之间的随机数

zskiplistNode里存了后退指针、分值、层数，层数里存有前进指针和跨度，跨度可以用来计算排名

跳表中节点按照分支大小、对象大小的顺序进行排序



#### 其他

整数集合(intset)是集合键的底层实现之一，整数集合底层是数组，只支持升级操作，不能降级

压缩列表(ziplist)是列表键和哈希键的底层实现之一，根据需要升级为其他数据结构


以上的数据结构并没有直接用到Redis中，而是基于这些数据结构创建了对象，包括**字符串对象、列表对象、哈希对象、集合对象、有序集合对象**

```c
typedef struct redisObject {
  unsigned type:4;  // 类型
  unsigned encoding:4;  // 编码
  void *ptr;   // 指向了底层的数据结构
  int refcount; // 引用计数，用于内存回收
  unsigned lru;  // 最后一次被访问的时间，用来计算对象空转时间
}
```

* 列表对象底层是ziplist -> linkedlist

* 哈希对象底层是ziplist -> hashtable

* 集合对象底层是intset -> hashtable

* 有序集合对象底层是ziplist -> skiplist



### 数据库

#### 过期策略

数据库键的过期时间全部保存在redisDb的过期字典中

```c
typedef struct redisDb {
  dict *dict;  // 这里存的键值对本身
  dict *expires; // 这里时过期字典，存的是所有key的过期时间，单位毫秒
}
```

Redis采用**惰性删除**和**定期删除**结合的过期策略

* 惰性删除是指每次查询key时，先检查是否过期，过期了则将键删除
* 定期删除是由服务器启动的单独线程执行`redis.c/serverCron`.`activeExpireCycle`，每次运行时，从过期字典中随机找一定数量的key进行检查并删除。
  * 有一个全局变量current_db记录当前的检查进度，全部检查完后，current_db重置为0，开始新一轮的检查

#### AOF、RDB和复制时过期键的处理

* RDB文件，生成时会忽略过期键，写入时主服务器会忽略过期键，从服务器会载入数据库
* AOF文件，写入时不会因为过期键产生任何影响，因为删除策略删除时会在AOF文件后追加一条删除命令
* AOF重写过程中，过期键不会被重写到AOF文件中
* 复制模式下，过期键的删除操作统一由主服务器发起，主服务器删除一个过期键后，会显示地向所有从服务器发送一个DEL命令；从服务器在读取到过期键时，也不会进行删除处理。

### 内存不足的处理
当执行写入命令时，如果发现内存不够，那么就会按照配置的淘汰策略清理内存，淘汰策略一般有6种，Redis4.0版本后又增加了2种，主要由分为三类

* 不处理，等报错(默认的配置)
noeviction，发现内存不够时，不删除key，执行写入命令时发现内存不够直接返回错误信息。（Redis默认的配置就是noeviction）
* 从所有结果集中的key中挑选，进行淘汰(随机，lru，lfu三种)
  * allkeys-random 就是从所有的key中随机挑选key，进行淘汰
  * allkeys-lru 就是从所有的key中挑选最近使用时间距离现在最远的key，进行淘汰
  * allkeys-lfu 就是从所有的key中挑选使用频率最低的key，进行淘汰。（这是Redis 4.0版本后新增的策略）
* 从设置了过期时间的key中挑选，进行淘汰(随机，lru，ttl，lfu)，这种就是从设置了expires过期时间的结果集中选出一部分key淘汰，挑选的算法有：
  * volatile-random 从设置了过期时间的结果集中随机挑选key删除。
  * volatile-lru 从设置了过期时间的结果集中挑选上次使用时间距离现在最久的key开始删除
  * volatile-ttl 从设置了过期时间的结果集中挑选可存活时间最短的key开始删除(也就是从哪些快要过期的key中先删除)
  * volatile-lfu 从过期时间的结果集中选择使用频率最低的key开始删除（这是Redis 4.0版本后新增的策略）


#### 持久化

#### RDB

RDB文件保存了Redis服务器中所有数据库的所有键值对数据，SAVE(阻塞)和BGSAVE(非阻塞)两个命令可以实现保存操作，服务器状态中会保存用save选项设置的保存条件，任意一个条件满足时会自动执行BGSAVE。

#### AOF

AOF的实现分命令追加、文件写入、文件同步三个步骤。AOF文件记录的是Redis命令。命令会首先保存到aof_buf缓冲区，之后再定期同步到AOF文件。

AOF重写会整理数据库的内容，重新只根据结果生成命令，文件会小很多。在重写期间，服务器新执行的命令会记录到重写缓冲区，重写完成后，再将缓冲区中的命令追加到AOF文件中，然后替换掉原来的AOF文件。
