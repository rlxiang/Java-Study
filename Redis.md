## Redis

## Nosql概述

### 为什么要用Nosql



>1、单机MySQL的年代

![image-20210802152428020](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210802152428020.png)

90年代，一个基本的网站访问量一般不会太大，单个数据库完全足够！这个时候，更多的是去使用静态页面 HTML~，服务器根本没有太大的压力，这样我们的单机mysql没有什么压力

思考一下，这种情况下：整个网站的瓶颈是什么？

1. 数据量如果太大，一个机器放不下了！
2. 数据的索引（Mysql如果单机超过300万条就要使用索引），一个机器内存也放不下
3. 访问量（读写混合），一个服务器承受不了~

只要开始出现以上三种情况之一，那么你就必须就要晋级！

>2、Memcached（缓存）+ MySQL 这样可以做垂直拆分(读写分离)

网站80%的情况都是在读，每次都要去查询数据库的话就十分的麻烦 ！所以说我们希望减轻数据的压力，我们可以使用缓存来保证效率

发展过程：优化数据结构和索引-->文件缓存（IO）-->Memcached（当时最热门的技术！）

![image-20210802153503944](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210802153503944.png)

>3、分库分表 + 水平拆分 + MySQL集群

==数据库本质就是读写==

早些年MyISAM：表锁，十分影响效率！高并发下就会出现严重的锁问题

转战Innodb：行锁

慢慢的就开始使用分库分表解决写的压力！MySQL在这个年代推出了表分区！这个并没有多少公司使用！

之后就是MySQL集群，很好满足那个年代的所有需求！

![image-20210802154112864](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210802154112864.png)

>4、最近的年代

MySQL等关系型数据库就不够用了！数据量很多，变化很快~！

MySQL有的使用它来存储一些比较大的文件，博客，图片!数据库表很大，效率就低了!如果有一种数据库来专门处理这种数据,MySQL压力就变得十分小（研究如何处理这些问题!)，大数据的IO压力下，表几乎没法更大！

>目前一个基本的互联网项目！（架构模型）

![image-20210802155826668](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210802155826668.png)

>为什么要用NoSQL！

用户的个人信息，社交网络，地理位置。用户自己产生的数据，用户日志等等爆发式增长

这个时候我们就需要使用NoSQL 数据库，Nosql可以很好的处理以上的问题

### 什么是NoSQL

NoSQL = Not Only SQL （不仅仅是SQL）

泛指非关系型数据库，随着web2.0互联网的诞生！传统的关系型数据库很难对付web2.0时代！尤其是超大规模的高并发的社区！暴露出来很多难以克服的问题，这些问题关系型数据库解决不了，NoSQL在当前大数据环境下发展的十分迅速，Redis是发展最快的，而且我们必须掌握

很多的数据类型如 用户的个人信息，社交网络，地理位置。这些数据类型的存储不需要一个固定的格式，不需要过多的操作就可以横向扩展的！



>NoSQL 特点

1. 方便扩展（数据之间没有关系，很好扩展！）
2. 大数据量高性能（Redis一秒可以读11万，写8万，NoSQL的缓存记录级，是一种细粒度的缓存，性能会比较高!）
3. 数据类型是多样型的！（不需要事先设计数据库！随取随用！如果是数据量十分大的表，很多人就无法设计了！）
4. 传统的关系型数据库(RDBMS)和NoSQL

```
传统的 RDBMS
- 结构化组织(表和列)
- SQL
- 数据和关系都存在单独的表中
- 操作语言，数据定义语言
- 严格的一致性
- 基础的事务
- ....
```

```
NoSql
- 不仅仅是数据
- 没有固定的查询语言
- 键值对存储，列存储，文档存储，图形数据库（社交关系）
- 最终一致性
- CAP定理 和 BASE 理论
- 高性能，高可用，高可扩
- ....
```

>了解：3V+3高

大数据时代的3V:主要是描述问题的

1. 海量Volume
2. 多样Variety
3. 实时Velocity

大数据时代的3高:主要是对程序的要求

1. 高并发
2. 高可拓
3. 高性能

### NoSQL的四大分类

**KV键值对：**

- 新浪：Redis
- 美团：Redis + Tair
- 阿里、百度：Redis + memecache

**文档型数据库（bson格式 和 json格式一样）**

- MongoDB（一般必须要掌握）
  - MongoDB是一个基于分布式文件存储的数据库，C++编写
  - MongoDB是一个介于关系型数据库和非关系型数据库中间的产品！MongoDB是非关系型数据库中功能最丰富，最想关系型数据库的！
- ConthDB

**列存储数据库**

- HBase
- 分布式文件系统

**图关系数据库**

- 他不是存放图形，存放的是关系，比如：朋友圈社交网络，广告推荐！
- Neo4j，InfoGrid；

>四者对比

![image-20210802171702222](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210802171702222.png)



## Redis入门

### 概述

>Redis是什么？

Redis（==Re==mote ==Di==ctionary ==S==erver），即远程字典服务！

是一个开源的使用ANSIl C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的APl。

![image-20210802173853731](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210802173853731.png)

redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。

免费和开源!是当下最热门的NoSQL技术之一!也被人们称之为结构化数据库!

>Redis 能干嘛？

1. 内存存储、持久化，内存中是断电即失、所以说持久化很重要( rdb、aof )
2. 效率高，可以用于高速缓存
3. 发布订阅系统
4. 地图信息分析
5. 计时器、计数器（浏览量!)
6. ……

>特性

1. 多样的数据类型
2. 持久化
3. 集群
4. 事务
5. ……

>学习中需要用到的东西

1. 官网:https://redis.io/
2. 中文网: http://www.redis.cn/

![image-20210802200513202](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210802200513202.png)

Linux安装redis 网上看教程（略）

### 测试性能

**redis-benchmark**是一个压力测试工具，官方自带的性能测试工具！

redis-benchmark 命令参数

![image-20210803102404405](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210803102404405.png)

我们简单测试下

```bash
#测试：100个并发连接 100000 请求
redis-benchmark -h localhost -p 6379 -c 100 -n 100000
```

### 基础的知识

redis默认有16个数据库

![image-20210803162857922](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210803162857922.png)

默认使用的是第0个

可以使用select 进行数据的切换

```bash
127.0.0.1:6379> select 3 # 切换数据库
OK
127.0.0.1:6379[3]> dbsize # 查看db大小key
(integer) 0
127.0.0.1:6379[3]> 
```

```bash
127.0.0.1:6379[3]> keys * # 查看当前数据库中所有的key
1) "name"

```

清除当前数据库`flushdb`

```bash
127.0.0.1:6379[3]> flushdb # 清除当前数据库
OK
127.0.0.1:6379[3]> keys *
(empty array)
127.0.0.1:6379[3]> 

```

清空所有数据库`FLUSHALL`



>Redis 是单线程的！

明白Redis是很快的，官方表示，Redis是基于内存操作，CPU不是Redis的性能瓶颈，Redis的瓶颈是根据机器的内存和网络贷款，既然可以使用单线程来实现，就是用单线程了

Redis是C语言写的，官方提供的数据为 100000+的QPS，完全不比同样使用key-value的Memecache差！

**Redis为什么单线程还这么快？**

1、误区1：高性能的服务器一定是多线程的？

2、误区2：多线程一定比单线程效率高！

核心：Redis是将所有的数据全部放在内存中的，所以说使用单线程操作就是最高的。如果使用多线程，会产生cpu的上下文切换，会产生耗时的操作，对于内存系统来说，如果没有上下文切换，效率就是最高的！多次读写都是在一个CPU中，在内存情况下，这个方案是最佳的。

## 五大基本数据类型

>官方文档

![image-20210803165036573](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210803165036573.png)

全段翻译：Redis是一个开源(BSD许可)的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件MQ。它支持多种类型的数据结构，如字符串 ( strings )，散列 ( hashes )，列表( lists )，集合( sets )，有序集合( sorted sets ）与范围查询，bitmaps ,hyperloglogs和地理空间( geospatial )索引半径查询。Redis内置了复制( replication )，LUA脚本( Lua scripting )，LRU驱动事件( LRU eviction )，事务( transactions ）和不同级别的磁盘持久化 ( persistence )，并通过Redis哨兵( Sentinel )和自动分区(Cluster )提供高可用性( high availability ) .

### Redis-Key

```bash
##############################################################################################################################
keys * # 查看所有的key
exists key # 判断当前的key是否存在
move key 1 # 移除当前的key
expire key 秒数 # 设置当前key的过期时间
ttl key # 查看被设置了过期时间的key还剩多少秒
type key # 查看当前key的类型
random可以 # 随意取一个key
rename oldkey newkey # 重命名
select num # 按索引查询
dbsize # 数据库有多少个key
```

### String(字符串)

```bash
############################################################################################################################
127.0.0.1:6379> set key1 v1  # 设置值
OK
127.0.0.1:6379> keys *
1) "key1"
127.0.0.1:6379> get key1  # 获得值
"v1"
127.0.0.1:6379> clear
127.0.0.1:6379> EXISTS key1 # 判断某一个key是否存在
(integer) 1
127.0.0.1:6379> append key1 "hello" # 追加字符串，如果当前key不存在，就相当于set key操作
(integer) 7
127.0.0.1:6379> get key1
"v1hello"
127.0.0.1:6379> STRLEN key1 # 获取字符串的长度
(integer) 7
127.0.0.1:6379> append key1 ",xiang"
(integer) 13
127.0.0.1:6379> get key1
"v1hello,xiang"
127.0.0.1:6379> 

############################################################################################################################

127.0.0.1:6379> set views 0
OK
127.0.0.1:6379> get views
"0"
127.0.0.1:6379> incr views # 自增1
(integer) 1
127.0.0.1:6379> incr views
(integer) 2
127.0.0.1:6379> get views
"2"
127.0.0.1:6379> decr views # 自减1
(integer) 1
127.0.0.1:6379> decr views
(integer) 0
127.0.0.1:6379> decr views
(integer) -1
127.0.0.1:6379> get views
"-1"
127.0.0.1:6379> incrby views 10 # 自增10（类似于i+=10）
(integer) 9
127.0.0.1:6379> incrby views 10
(integer) 19
127.0.0.1:6379> decrby views 10 # 自减10（类似于i-=10）
(integer) 9
127.0.0.1:6379> 

############################################################################################################################

# 字符串范围 range
127.0.0.1:6379> set key1 "hello,xiang" # 设置 key1的值
OK
127.0.0.1:6379> get key1
"hello,xiang"
127.0.0.1:6379> GETRANGE key1 0 3   # 截取字符串 [0,3]顾头顾尾
"hell"
127.0.0.1:6379> GETRANGE key1 0 -1 # 获取全部的字符串 和 get key是一样的
"hello,xiang"
127.0.0.1:6379> 

############################################################################################################################
127.0.0.1:6379> set key2 abcdefg
OK
127.0.0.1:6379> get key2
"abcdefg"
127.0.0.1:6379> SETRANGE key 1 xx
(integer) 3
127.0.0.1:6379> get key2
"abcdefg"
127.0.0.1:6379> SETRANGE key2 1 xx # 替换指定位置开始的字符串
(integer) 7
127.0.0.1:6379> get key2
"axxdefg"

############################################################################################################################

# setex (set with expire) # 设置过期时间
# setnx (set if not exist) # 不存在的时候设置 （在分布式锁中常常使用）

127.0.0.1:6379> setex key3 30 "hello" # 设置key3的值为hello，并且30s后过期
OK
127.0.0.1:6379> ttl key3
(integer) 25
127.0.0.1:6379> get key3
"hello"
127.0.0.1:6379> ttl key3
(integer) 8
127.0.0.1:6379> setnx mykey "redis" # 如果mykey不存在就会创建mykey
(integer) 1
127.0.0.1:6379> get mykey
"redis"
127.0.0.1:6379> setnx mykey "MongoDB" # 如果mykey存在，则创建失败
(integer) 0
127.0.0.1:6379> get mykey
"redis"
127.0.0.1:6379> 
############################################################################################################################
 # 批量设置与获取
mset
mget

127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3 # 同时设置多个值
OK
127.0.0.1:6379> keys *
1) "k2"
2) "k3"
3) "k1"
127.0.0.1:6379> get k1
"v1"
127.0.0.1:6379> mget k1 k2 k3 # 同时获取多个值
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> msetnx k1 v1 k4 v4 # msetnx(如果不存在就创建，存在则创建失败) 是原子性的操作，要么同时成功，要么同时失败
(integer) 0
127.0.0.1:6379> get k4
(nil)
127.0.0.1:6379> 

## 对象
set user:1 {name:zhangsan,age:3} # 设置一个user:1 对象 值为 json字符串来保存一个对象！

# 这里的key是一个巧妙的设计 user:{id}:{field}, 如此设计在redis中是完全可以的
127.0.0.1:6379> mset user:1:name zhangsan user:1:age 2 
OK
127.0.0.1:6379> mget user:1:name user:1:age
1) "zhangsan"
2) "2"

############################################################################################################################
getset # 先get然后再set


127.0.0.1:6379> getset db redis # 如果不存在值，则返回nil
(nil)
127.0.0.1:6379> get db
"redis"
127.0.0.1:6379> getset db mongodb #（会将原来的db的值返回，并将其值改为mongodb） 如果存在值，获取原来的值，并设置新的值
"redis"
127.0.0.1:6379> get db
"mongodb"
```

String类似的使用场景:value除了是我们的字符串还可以是我们的数字!

- 计数器
- 统计多单位的数量
- 粉丝数
- 对象缓存存储!

### List（列表）

基本的数据类型，列表，在redis可以将其完成栈，队列，阻塞队列

所有的list命令都是`l`开头的

```bash
############################################################################################################################
127.0.0.1:6379> lpush list one # 将一个值或者多个值，插入到列表头部(左边插入)
(integer) 1
127.0.0.1:6379> lpush list two
(integer) 2
127.0.0.1:6379> lpush list three
(integer) 3
127.0.0.1:6379> lrange list 0 -1 # 获取list中所有的值
1) "three"
2) "two"
3) "one"
127.0.0.1:6379> lpush list four five
(integer) 6
127.0.0.1:6379> lrange list 0 -1
1) "five"
2) "four"
3) "three"
4) "two"
5) "one"
6) "right"
127.0.0.1:6379> lrange list 0 -1
1) "five"
2) "four"
3) "three"
4) "two"
5) "one"
6) "right"
127.0.0.1:6379> lrange list 0 1 # 通过去见获取具体的值！
1) "three"
2) "two"
127.0.0.1:6379> rpush list right # 将一个值或者多个值，插入到列表头部(右边插入)
(integer) 4
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "two"
3) "one"
4) "right"

############################################################################################################################
LPOP
RPOP
127.0.0.1:6379> lrange list 0 -1
1) "five"
2) "four"
3) "three"
4) "two"
5) "one"
6) "right"
127.0.0.1:6379> lpop list # 移除list中左边第一个元素
"five"
127.0.0.1:6379> rpop list # 移除list中右边第一个元素
"right"
127.0.0.1:6379> lrange list 0 -1
1) "four"
2) "three"
3) "two"
4) "one"
############################################################################################################################
Lindex

127.0.0.1:6379> lrange list 0 -1
1) "four"
2) "three"
3) "two"
4) "one"
127.0.0.1:6379> lindex list 1 # 通过下标获得 list 中的某一个值
"three"
127.0.0.1:6379> lindex list 0
"four"

############################################################################################################################

Llen

127.0.0.1:6379> lrange list 0 -1
1) "four"
2) "three"
3) "two"
4) "one"
127.0.0.1:6379> llen list # 返回list的长度
(integer) 4

############################################################################################################################
移除指定的值！

lrem

127.0.0.1:6379> lrange list 0 -1
1) "one"
2) "four"
3) "three"
4) "two"
5) "one"
127.0.0.1:6379> lrem list 1 four # 移除list集合中指定个数的值，精确匹配
(integer) 1
127.0.0.1:6379> lrange list 0 -1
1) "one"
2) "three"
3) "two"
4) "one"
127.0.0.1:6379> lrem list 2 one
(integer) 2
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "two"

############################################################################################################################
ltrim 修剪

127.0.0.1:6379> rpush mylist "hello"
(integer) 1
127.0.0.1:6379> rpush mylist "hello1"
(integer) 2
127.0.0.1:6379> rpush mylist "hello2"
(integer) 3
127.0.0.1:6379> rpush mylist "hello3"
(integer) 4
127.0.0.1:6379> keys *
1) "mylist"
127.0.0.1:6379> ltrim mylist 1 2   # 通过下标截取指定的长度，这个list已经被改变了，被截断后只剩下截取的元素
OK
127.0.0.1:6379> lrange mylist 0 -1
1) "hello1"
2) "hello2"

############################################################################################################################
rpoplpush # 移除列表的最后一个元素，并将该元素移动到新的列表中

127.0.0.1:6379> rpush mylist "hello"
(integer) 1
127.0.0.1:6379> rpush mylist "hello1"
(integer) 2
127.0.0.1:6379> rpush mylist "hello2"
(integer) 3
127.0.0.1:6379> rpoplpush mylist myotherlist # 移除列表的最后一个元素，并将该元素移动到新的列表中
"hello2"
127.0.0.1:6379> lrange mylist 0 -1 # 查看原来的列表，发现少了一个值
1) "hello"
2) "hello1"
127.0.0.1:6379> lrange myotherlist 0 -1 # 查看目标列表中，确实存在该值
1) "hello2"

############################################################################################################################
lset 将列表中指定下标的值替换为另外一个值，更新操作

127.0.0.1:6379> EXISTS list # 判断这个列表是否存在
(integer) 0
127.0.0.1:6379> lset list 0 item # 如果不存在列表，我们去更新就会报错
(error) ERR no such key
127.0.0.1:6379> lpush list value1
(integer) 1
127.0.0.1:6379> lrange list 0 0
1) "value1"
127.0.0.1:6379> lset list 0 item # 如果存在，更新当前下标的值
OK
127.0.0.1:6379> lrange list 0 0
1) "item"
127.0.0.1:6379> lset list 1 other # 如果列表中没有该索引也会报错
(error) ERR index out of range

############################################################################################################################
linsert # 将某个具体的值插入到列表中某个元素的前面或者后面

127.0.0.1:6379> rpush mylist "hello" "world"
(integer) 2
127.0.0.1:6379> linsert mylist before "world" "other" # 将other插入到world前面
(integer) 3
127.0.0.1:6379> lrange mylist 0 -1
1) "hello"
2) "other"
3) "world"
127.0.0.1:6379> linsert mylist after "world" "new" # 将new插入到world后面
(integer) 4
127.0.0.1:6379> lrange mylist 0 -1
1) "hello"
2) "other"
3) "world"
4) "new"
```

>小结

- List实际上是一个链表，before Node after，left，right都可以插入
- 如果key不存在，创建新的链表
- 如果key存在，新增内容
- 如果移除了所有值，空链表，也代表不存在！
- 在两边插入或者改动值，效率最高！操作中间元素，效率会低很多



### Set（集合）

set中的值是不可以重复的

```bash
############################################################################################################################
127.0.0.1:6379> sadd myset "rlxiang1" # 往set集合中添加元素
(integer) 1
127.0.0.1:6379> sadd myset "love me"
(integer) 1
127.0.0.1:6379> SMEMBERS myset # 查看指定的set中所有的值
1) "love me"
2) "rlxiang1"
3) "hello"
127.0.0.1:6379> SISMEMBER myset hello # 判断某个值是不是在set集合中
(integer) 1
127.0.0.1:6379> SISMEMBER myset world
(integer) 0

############################################################################################################################

127.0.0.1:6379> SMEMBERS myset
1) "love me"
2) "rlxiang1"
3) "hello"
127.0.0.1:6379> scard myset # 获取set中元素的个数
(integer) 3

############################################################################################################################
srem

127.0.0.1:6379> SMEMBERS myset
1) "love me"
2) "rlxiang1"
3) "hello"

127.0.0.1:6379> srem myset hello # 移除set集合中指定的元素
(integer) 1
127.0.0.1:6379> SMEMBERS myset
1) "love me"
2) "rlxiang1"

############################################################################################################################
set 是一个无需不重复的集合

127.0.0.1:6379> SRANDMEMBER myset # 随机抽取指定个数的元素，默认为1
"love me"
127.0.0.1:6379> SRANDMEMBER myset 
"hello"
127.0.0.1:6379> SRANDMEMBER myset 2 # 随机抽取指定2个元素
1) "love me"
2) "rlxiang1"

############################################################################################################################
随机删除值

127.0.0.1:6379> SMEMBERS myset
1) "loverlxiang2"
2) "love me"
3) "hello"
4) "rlxiang1"
127.0.0.1:6379> spop myset # 随机删除一个元素
"rlxiang1"
127.0.0.1:6379> spop myset
"hello"
127.0.0.1:6379> SMEMBERS myset
1) "loverlxiang2"
2) "love me"

############################################################################################################################
将一个指定的值移动到另一个set集合中！
127.0.0.1:6379> sadd myset hello world rlxiang1
(integer) 3
127.0.0.1:6379> sadd myset2 set2
(integer) 1
127.0.0.1:6379> smove myset myset2 rlxiang1 # 将一个指定的值，移动到另一个set集合中
(integer) 1
127.0.0.1:6379> SMEMBERS myset
1) "world"
2) "hello"
127.0.0.1:6379> SMEMBERS myset2
1) "rlxiang1"
2) "set2"

############################################################################################################################
差集 sdiff
交集 SINTER
并集 SUNION

127.0.0.1:6379> sadd key1 b
(integer) 1
127.0.0.1:6379> sadd key1 c
(integer) 1
127.0.0.1:6379> sadd key2 c
(integer) 1
127.0.0.1:6379> sadd key2 d
(integer) 1
127.0.0.1:6379> sadd key2 e
(integer) 1
127.0.0.1:6379> sdiff key1 key2 # 差集
1) "b"
2) "a"
127.0.0.1:6379> SINTER key1 key2 # 交集
1) "c"
127.0.0.1:6379> SUNION key1 key2 # 并集
1) "c"
2) "e"
3) "b"
4) "a"
5) "d"
```

### Hash（哈希）

可以将hash想象成一个Map集合，key-Map集合，这时候，值就是map集合！本质和String类型没有太大的区别，还是一个简单的key-value

```bash
############################################################################################################################
127.0.0.1:6379> hset myhash field1 rlxiang1 # 设置一个具体的 key-value值
(integer) 1
127.0.0.1:6379> hget myhash field1
"rlxiang1"
127.0.0.1:6379> hmset mysh field1 hello field2 world # 设置多个 可以-value值
OK
127.0.0.1:6379> hmget mysh field1 field2 # 获取多个字段值
1) "hello"
2) "world"
127.0.0.1:6379> hgetall mysh # 获取全部的数据
1) "field1"
2) "hello"
3) "field2"
4) "world"
127.0.0.1:6379> hdel mysh field1 # 删除hash指定的key字段！对应的value值也就没有了
(integer) 1
127.0.0.1:6379> hgetall mysh
1) "field2"
2) "world"

############################################################################################################################
hlen

127.0.0.1:6379> hmset mysh field1 hello field2 world
OK

127.0.0.1:6379> hgetall mysh # 获取hash表的字段数量！
1) "field2"
2) "world"
3) "field1"
4) "hello"
127.0.0.1:6379> hlen mysh
(integer) 2

############################################################################################################################
127.0.0.1:6379> HEXISTS mysh field1 # 判断hash中指定字段是否存在！
(integer) 1
127.0.0.1:6379> HEXISTS mysh field3
(integer) 0

############################################################################################################################
# 只获得所有的field
# 只获得所有的value
127.0.0.1:6379> hkeys mysh # 只获得所有的field
1) "field2"
2) "field1"

127.0.0.1:6379> hvals mysh # 只获得所有的value
1) "world"
2) "hello"
############################################################################################################################
incr    decr

127.0.0.1:6379> hset mysh field3 5 
(integer) 1
127.0.0.1:6379> HINCRBY mysh field3 1 # 指定增量(field3中的值自增1)
(integer) 6
127.0.0.1:6379> HINCRBY mysh field3 -1
(integer) 5
127.0.0.1:6379> hsetnx mysh field4 hello # 如果存在field4就创建失败，没有则创建成功
(integer) 1
127.0.0.1:6379> hsetnx mysh field4 world
(integer) 0

```

### Zset（有序集合）

在set的基础上，增加了一个值 zset k1 score1 v1

```bash
127.0.0.1:6379> zadd myset 1 one # 添加一个值
(integer) 1
127.0.0.1:6379> zadd myset 2 two 3 three # 添加多个值
(integer) 2
127.0.0.1:6379> zrange myset 0 -1 # 获取集合中所有的值
1) "one"
2) "two"
3) "three"
##################################################################################################
排序如何实现

127.0.0.1:6379> zadd salary 2500 xiaohong # 添加三个用户
(integer) 1
127.0.0.1:6379> zadd salary 5000 zhangsan
(integer) 1
127.0.0.1:6379> zadd salary 500 kuangshen
(integer) 1
# ZRANGEBYSCORE min max
127.0.0.1:6379> ZRANGEBYSCORE salary -inf +inf # 显示全部的用户 从小到大排序
1) "kuangshen"
2) "xiaohong"
3) "zhangsan"

127.0.0.1:6379> ZREVRANGE salary 0 -1 # 从大到小显示全部用户
1) "zhangsan"
2) "kuangshen"

127.0.0.1:6379> ZRANGEBYSCORE salary -inf +inf withscores # 显示全部用户并附带成绩
1) "kuangshen"
2) "500"
3) "xiaohong"
4) "2500"
5) "zhangsan"
6) "5000"

127.0.0.1:6379> ZRANGEBYSCORE salary -inf 2500 withscores # 显示工资小于2500员工的升序排列
1) "kuangshen"
2) "500"
3) "xiaohong"
4) "2500"

##################################################################################################
zrem 

127.0.0.1:6379> zrange salary 0 -1 # 获取所有元素，从小到大
1) "kuangshen"
2) "xiaohong"
3) "zhangsan"
127.0.0.1:6379> zrem salary xiaohong # 移除有序集合中指定的元素
(integer) 1
127.0.0.1:6379> zrange salary 0 -1
1) "kuangshen"
2) "zhangsan"
127.0.0.1:6379> zcard salary # 获取有序集合中的个数
(integer) 2

##################################################################################################
127.0.0.1:6379> zadd myset 1 hello
(integer) 1
127.0.0.1:6379> zadd myset 2 world 3 rlxiang
(integer) 2
127.0.0.1:6379> zcount myset 1 3 # 获取指定区间的成员数量
(integer) 3
127.0.0.1:6379> zcount myset 1 2
(integer) 2

```

## 三种特殊数据类型

### geospatial  地理位置

朋友的定位，附近的人，打车距离计算

可以使用Redis的Geo，在Redis3.2版本就推出了！这个功能可以推算出地址位置的信息，如两地距离，方圆几里的人

可以查询一些测试数据：http://www.jsons.cn/lngcodeinfo/C27D8D38AB157D35/

有八个命令

![image-20210804193540102](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210804193540102.png)

>geoadd

```bash
# geoadd      添加地理位置
# 规则：两级无法直接添加，我们一般会下载城市数据，直接通过java程序一次性导入！
# 有效的经度从-180度到180度。
# 有效的纬度从-85.05112878度到85.05112878度。
# 当坐标位置超出上述指定范围时，该命令将会返回一个错误。
	#127.0.0.1:6379> geoadd china:city 39.90 116.40 beijin
	(error) ERR invalid longitude,1atitude pair 39.900000,116.40000
# 参数 key 值 (维度、经度、名称)
127.0.0.1:6379> geoadd china:city 116.40 39.90 beijing
(integer) 1
127.0.0.1:6379> geoadd china:city 121.47 31.23 shanghai
(integer) 1
127.0.0.1:6379> geoadd china:city 106.50 29.53 chongqin 114.05 22.53 shengzhen
(integer) 2
127.0.0.1:6379> geoadd china:city 120.16 30.24 hangzhou 108.96 34.26 xian
(integer) 2

```

>geopos

获得当前的定位：一定是一个坐标值

```bash
# geopos
127.0.0.1:6379> geopos china:city beijing # 获取制定城市的经度和纬度
1) 1) "116.39999896287918091"
   2) "39.90000009167092543"
127.0.0.1:6379> geopos china:city beijing chongqin
1) 1) "116.39999896287918091"
   2) "39.90000009167092543"
2) 1) "106.49999767541885376"
   2) "29.52999957900659211"
```

>geodist

两人之间的距离

单位

- m表示单位为米
- km 千米
- mi 英里
- ft 英尺

```bash
127.0.0.1:6379> geodist china:city beijing shanghai km # 查看背景到上海的直线距离
"1067.3788"
127.0.0.1:6379> geodist china:city beijing chongqin km # 查看背景到重庆的直线距离
"1464.0708"
```



>georadius 以给定的经纬度为中心，找出某一半径内的元素

我附近人？(获得所有附近的人地址，定位)通过半径来查询！

获得指定数量的人

所有数据应该都录入：china:city, 才会让结果更加准确

```bash
127.0.0.1:6379> GEORADIUS china:city 110 30 1000 km # 以110 30 这个经纬度为中心，寻找方圆1000km以内的城市
1) "chongqin"
2) "xian"
3) "shengzhen"
4) "hangzhou"
127.0.0.1:6379> GEORADIUS china:city 110 30 500 km withdist # 显示到中心距离（直线距离）的位置
1) 1) "chongqin"
   2) "341.9374"
2) 1) "xian"
   2) "483.8340"
127.0.0.1:6379> GEORADIUS china:city 110 30 500 km withcoord # 显示里中心点距离在500km内的城市的经纬度
1) 1) "chongqin"
   2) 1) "106.49999767541885376"
      2) "29.52999957900659211"
2) 1) "xian"
   2) 1) "108.96000176668167114"
      2) "34.25999964418929977"
127.0.0.1:6379> GEORADIUS china:city 110 30 500 km withcoord withdist count 1 # 筛选出指定的结果(只显示一个人)
1) 1) "chongqin"
   2) "341.9374"
   3) 1) "106.49999767541885376"
      2) "29.52999957900659211"
127.0.0.1:6379> GEORADIUS china:city 110 30 500 km withcoord withdist count 2
1) 1) "chongqin"
   2) "341.9374"
   3) 1) "106.49999767541885376"
      2) "29.52999957900659211"
2) 1) "xian"
   2) "483.8340"
   3) 1) "108.96000176668167114"
      2) "34.25999964418929977"

```



># GEORADIUSBYMEMBER

```bash
# 找出位于指定元素周围的其他元素
127.0.0.1:6379> GEORADIUSBYMEMBER china:city beijing 1000 km # 找出距离北京1000km内的城市
1) "beijing"
2) "xian"
127.0.0.1:6379> GEORADIUSBYMEMBER china:city shanghai 400  km
1) "hangzhou"
2) "shanghai"

```



>GEOHASH命令-返回一个或多个位置元素的Geohash表示

该命令返回11个字符的Geohash字符串

```bash
# 将二维的经纬度转换成以为字符串，如果两个字符串越接近，那么距离越近
127.0.0.1:6379> geohash china:city beijing chongqin
1) "wx4fbxxfke0"
2) "wm5xzrybty0"
```



># GEOSEARCH

返回使用GEOADD填充地理空间信息的已排序集合的成员，这些成员位于给定形状指定的区域边界内。该命令扩展了GEORADIUS命令，因此除了在圆形区域内搜索外，它还支持在矩形区域内搜索

```bash
127.0.0.1:6379> geosearch china:city fromlonlat 116 39 byradius 1000 km asc # 以圆形为搜索，找到距离116 39距离在1000km以内的城市，并按照距离升序排列
1) "beijing"
2) "xian"
3) "shanghai"

```



># GEOSEARCHSTORE

这个命令类似于GEOSEARCH，但是将结果存储在目标键中。

```bash
127.0.0.1:6379> GEOSEARCHSTORE key1 china:city fromlonlat 116 39 bybox 400 400 km asc # 将距离116 39 位置 以方形400 400km内的城市添加到key1中
(integer) 1

127.0.0.1:6379> GEOSEARCH key1 fromlonlat 116 39 bybox 400 400 km
1) "beijing"

```

>GEO 底层的实现原理就是Zset！我们可以使用Zset命令来操作geo

```bash
127.0.0.1:6379> zrange china:city 0 -1 # 查看地图中所有的元素
1) "chongqin"
2) "xian"
3) "shengzhen"
4) "hangzhou"
5) "shanghai"
6) "beijing"
127.0.0.1:6379> zrem china:city beijing # 移除指定元素
(integer) 1
127.0.0.1:6379> zrange china:city 0 -1
1) "chongqin"
2) "xian"
3) "shengzhen"
4) "hangzhou"
5) "shanghai"
```



### Hyperloglog

基数：就是一个数据集中不重复的元素的个数

A:{1,2,3,4,3,3,5}  基数为5

Redis Hyperloglog 基数统计算法

有点：占用的内存是固定的，2^64 不同的元素的基数，只需要消耗12KB内存，如果要从内存的角度来比较的话Hyperloglog是首选

**网页的页面访问量(UV)(一个人访问一个网站多次，但还是算作一个人！)**

传统的方式，set保存用户的id，然后就可以统计 set中的元素数量作为标准判断！

这个方式如果保存大量的用户id(id比较长的话，uuid)就会比较麻烦！我们的目的是为了技术，而不是保存用户id

对于统计网页UV的任务，Hyperloglog的错误率是可以接受的，官方错误率为0.81%

>测试使用

```bash
127.0.0.1:6379> pfadd mykey a b c d e f g h i j # 创建第一组元素mykey
(integer) 1
127.0.0.1:6379> PFCOUNT mykey # 统计 mykey 元素的基数数量
(integer) 10
127.0.0.1:6379> pfadd mykey a b c d e f g h i j i
(integer) 0
127.0.0.1:6379> PFCOUNT mykey
(integer) 10
127.0.0.1:6379> pfadd mykey2 i j z x c v b n m # 创建第二元素mykey
(integer) 1
127.0.0.1:6379> PFCOUNT mykey2
(integer) 9
127.0.0.1:6379> PFMERGE mykey3 mykey mykey2 # 合并两组数据到mykey3中
OK
127.0.0.1:6379> PFCOUNT mykey3
(integer) 15

```



### Bitmaps

>位存储(只有0 1)

如果一个对象只有两种状态，就可以使用该数据结构

Bitmaps 位图，数据结构！都是操作二进制位来进行记录，就只有0和1两个状态

>测试

使用bitmaps来记录 周一到周日的打卡！

周一 ：1 周二 ：1  周三 ：0  周四 ：1 周五：1 周六：1 周日：1

![image-20210804212332134](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210804212332134.png)

查看某一天是否有打卡

```bash
127.0.0.1:6379> getbit sign 3 # 获取某一天的值
(integer) 1
127.0.0.1:6379> getbit sign 6
(integer) 1

```

统计操作，统计 打卡的天数

```bash
127.0.0.1:6379> BITCOUNT sign # 统计sign中1的个数
(integer) 6
```



## 事务

原子性：要么同时成功，要么同时失败

Redis单条命令是保证原子性的，但是它的事务是不保证原子性的！

Redis事务本质：一组命令的集合（可以想象成这些命令会入队）！一个事务中的所有命令都会被序列化，在事务执行过程中，会按照顺序执行，它会一次性，顺序性，排他性地执行一系列的命令

Redis事务是没有隔离级别的概念，所有的命令在事务中，并没有直接被执行！只有发起执行命令的时候才会被执行！Exec

Redis的事务：

- 开启事务（multi）
- 命令入队（...）
- 执行事务（exec）

>正常执行事务！

```bash
127.0.0.1:6379> multi   # 开启事务
OK

# 命令入队
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> get k2
QUEUED
127.0.0.1:6379(TX)> set k3 v3
QUEUED

127.0.0.1:6379(TX)> exec # 执行事务，执行完事务后，这个事务就没有了，如果要继续执行事务就必须再开启
1) OK
2) OK
3) "v2"
4) OK

```

>放弃事务

```bash
127.0.0.1:6379> multi   # 开启事务
OK
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> set k4 v4
QUEUED
127.0.0.1:6379(TX)> DISCARD # 取消掉事务
OK
127.0.0.1:6379> get k4 # 取消掉事务后，队列中的命令都不会被执行
(nil)
127.0.0.1:6379> EXEC # 取消掉事务后，再执行就会报错
(error) ERR EXEC without MULTI
```

>编译型异常（代码有问题！，命令有错！）事务中所有的命令都不会被执行

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> set k3 v3
QUEUED
127.0.0.1:6379(TX)> getset k3 # 错误的命令
(error) ERR wrong number of arguments for 'getset' command
127.0.0.1:6379(TX)> set k4 v4
QUEUED
127.0.0.1:6379(TX)> set k5 v5
QUEUED
127.0.0.1:6379(TX)> exec # 执行事务报错
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k5 # 事务中的所有命令都没有执行 
(nil)

```





>运行时异常(1/0),如果事务队列中存在语法性错误，那么执行命令的时候，其他命令是可以正常执行的，错误命令会抛出异常！

```bash
127.0.0.1:6379> set k1 "v1"
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> incr k1 # 字符串是不能自增，运行时会报错
QUEUED
127.0.0.1:6379(TX)> set k2 v3
QUEUED
127.0.0.1:6379(TX)> set k3 v2
QUEUED
127.0.0.1:6379(TX)> get k3
QUEUED
127.0.0.1:6379(TX)> exec # 虽然队列中的命令运行时报错了，但是在队列中的其他命令还是会正常执行
1) (error) ERR value is not an integer or out of range
2) OK
3) OK
4) "v2"
127.0.0.1:6379> get k2
"v3"
127.0.0.1:6379> get k3
"v2"

```

>监控！(watch)，面试常问

**悲观锁**

- 很悲观，认为什么时候都会出现问题，无论做什么都会加锁！这很影响性能，比如synchronized

乐观锁

- 很乐观，认为什么时候都不会出问题，所以不会上锁！如果要更改数据，此时会判断一下，在此期间是否会有人修改这个数据
- 获取version，不会加锁
- 更新的时候比较version

>Redis 的监视测试

正常执行成功

```bash
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> watch money  # 监视 money 对象 相当于获取了money 100 ，执行事务的时候会去更新这个值
OK
127.0.0.1:6379> multi # 事务正常结束，数据期间没有发生变动，这个时候被正常执行
OK
127.0.0.1:6379(TX)> DECRBY money 20
QUEUED
127.0.0.1:6379(TX)> incrby out 20
QUEUED
127.0.0.1:6379(TX)> exec
1) (integer) 80
2) (integer) 20
```

测试多线程修改值，使用watch 可以当做redis的乐观锁操作

```bash
127.0.0.1:6379> watch money # 监视 money 对象 相当于获取了money 100 ，执行事务的时候会去更新这个值，更新前会去比较这个值有没有变动，如果变动则事务执行失败
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> DECRBY money 10
QUEUED
127.0.0.1:6379(TX)> INCRBY out 10
QUEUED
127.0.0.1:6379(TX)> exec # 执行之前，另外一个线程修改了数据，这个时候就会导致监视失败
(nil)
```

如果执行失败，获取最新的值就好

![image-20210805204153181](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210805204153181.png)



## Jedis

我们要使用Java来操作Redis

>什么是Jedis ，它是 Redis 官方推荐的java连接开发工具！ 使用java 操作Redis的中间件！如果要使用java操作Redis，那么一定要对Jedis十分熟悉！

>测试

1、导入对应的依赖

```xml
<!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.6.3</version>
</dependency>

<!--fastjson-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.76</version>
</dependency>
```

2、编码测试：

- 连接数据库
- 操作命令
- 断开连接

```java
package com.xiang;

import redis.clients.jedis.Jedis;

public class TestPing {
    public static void main(String[] args) {
        // 1. new Jedis()对象即可
        Jedis jedis = new Jedis("127.0.0.1",6379);
        // 2. jedis的所有方法都是之间学过的命令
        System.out.println(jedis.ping());   
    }
}// 输出为PONG
```

### 常用的API

**key**

```java
package com.xiang;

import redis.clients.jedis.Jedis;

import java.util.Set;

public class TestKey {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1",6379);

        System.out.println("清空数据:"+jedis.flushDB());
        System.out.println("判断某个键是否存在: "+jedis.exists( "username"));
        System.out.println("新增<'username ' , 'kuangshen'>的键值对:"+jedis.set("username", "kuangshen"));
        System.out.println("新增< 'password , ' password'>的键值对。"+jedis.set("password", "password"));
        System.out.print("系统中所有的键如下:");
        Set<String> keys = jedis.keys("*");
        System.out.println(keys);

        System.out.println("删除键password: "+jedis.del("password"));
        System.out.println("判断健password是否存在:"+jedis.exists( "password"));
        System.out.println("查看键username所存储的值的类型。"+jedis.type( "username"));
        System.out.println("随机返回key空间的一个:"+jedis.randomKey());
        System.out.println("重命名key: "+jedis.rename("username", "name"));
        System.out.println("取出改后的name: "+jedis.get( "name"));
        System.out.println("按索引查询:"+jedis.select(0));
        System.out.println("删除当前选择数据库中的所有key: "+jedis.flushDB());
        System.out.println("返回当前数据库中key的数目:"+jedis.dbSize());
        System.out.println("删除所有数据库中的所有key: "+jedis.flushAll());

    }
}

```

**String**

```java
package com.xiang;

import redis.clients.jedis.Jedis;

import java.util.concurrent.TimeUnit;


public class TestString {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        System.out.println("===========增加数据==========");
        System.out.println(jedis.set("key1", "value1"));
        System.out.println(jedis.set("key2", "value2"));
        System.out.println(jedis.set("key3", "value3"));
        System.out.println("删除键key2:" + jedis.del("key2"));
        System.out.println("获取键key2: " + jedis.get("key2"));
        System.out.println("修改key1: " + jedis.set("key1", "value1Changed"));
        System.out.println("获取key1的值:" + jedis.get("key1"));
        System.out.println("在key3后面加入值:" + jedis.append("key3", "End"));
        System.out.println("key3的值:" + jedis.get("key3"));
        System.out.println("增加多个键值对:" + jedis.mset("keye1", "value01", "key02", "valueO2", "keye03", "value93"));
        System.out.println("获取多个键值对。" + jedis.mget("keye1", " key02", " keye3"));
        System.out.println("获取多个键值对。" + jedis.mget("key01", "key02", "key03", "key04"));
        System.out.println("删除多个键值对:" + jedis.del("key01", " key02"));
        System.out.println("获取多个键值对。" + jedis.mget("keye1", " key02", " key03"));
        jedis.flushDB();

        System.out.println("=========新增键值对防止覆盖原先值==========");
        System.out.println(jedis.setnx("key1", "value1"));
        System.out.println(jedis.setnx("key2", "value2"));
        System.out.println(jedis.setnx("key2", "value2-new"));
        System.out.println(jedis.get("key1"));
        System.out.println(jedis.get("key2"));

        System.out.println("==========新增键值对并设置有效时间==去=去去====");
        System.out.println(jedis.setex("key3", 2, "value3"));
        System.out.println(jedis.get("key3"));
        try {
            TimeUnit.SECONDS.sleep(3);
        }catch (InterruptedException e){
            e.printStackTrace();
        }
        System.out.println(jedis.get( "key3" ));
        System.out.println("===========返回原值，并此键对应的值更新为新值==========");
        System. out.println(jedis.getSet( "key2" , "key2GetSet"));
        System. out . println(jedis.get( "key2" ));
        System.out.println("获得key2的值的字串:"+jedis.getrange( "key2",  2,4));
    }
}
```

**List**

```java
package com.xiang;

import redis.clients.jedis.Jedis;

public class TestList {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        System.out.println("================添加一个list=====================");
        jedis.lpush("collections", "ArrayList", "Vector", "Stack", "HashNap", "WeakHashMap", "LinkedHashMap");
        jedis.lpush("collections", "HashSet");
        jedis.lpush("collections", "TreeSet");
        jedis.lpush("collections", "TreeMap");
        System.out.println("collections的内容。" + jedis.lrange("collections", 0, -1));//-1代表倒数第一个元素，-2代表倒敌第二个元素

        // 删除列表指定的值，第二个参数为删除的个数（有重复时)，后add进去的值先被删，类似于出栈
        System.out.println("删除指定元素个数:" + jedis.lrem("collections", 2, "HashMap"));

        System.out.println("collections的内容:" + jedis.lrange("collections", 0, -1));
        System.out.println("删除下标0-3区间之外的元素:" + jedis.ltrim("collections", 0, 3));
        System.out.println("collections的内容: " + jedis.lrange("collections", 0, -1));
        System.out.println(" collections列表出栈（左端): " + jedis.lpop("collections"));
        System.out.println("collections的内容:" + jedis.lrange("collections", 0, -1));
        System.out.println("collections添加元素，从列表右端，与1push相对应:" + jedis.rpush("collections", "EnumMap"));
        System.out.println(" collections的内容:" + jedis.lrange("collections", 0, -1));
        System.out.println("collections列表出核（右端）: " + jedis.rpop("collections"));
        System.out.println(" collections的内容: " + jedis.lrange("collections", 0, -1));
        System.out.println("修改collections指定下标1的内容:" + jedis.lset("collections", 1, "LinkedArrayList"));
        System.out.println("collections的内容:" + jedis.lrange("collections", 0, -1));
        System.out.println("=assssssssaesaessaaesssss===s==");
        System.out.println("collections的长度:" + jedis.llen("collections"));
        System.out.println("获取collections下标为2的元素:" + jedis.lindex("collections", 2));
        System.out.println("========================================");
        jedis.lpush("sortedList", "3", "6", "2", "0", "7", "4");
        System.out.println("sortedList排序前:" + jedis.lrange("sortedList", 0, -1));
        System.out.println(jedis.sort("sortedList"));
        System.out.println("sortedList排序后:" + jedis.lrange("sortedList", 0,-1));

    }
}
```

**set**

```java
package com.xiang;

import redis.clients.jedis.Jedis;

public class TestSet {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        System.out.println("=======================向集合中添加元素（不重复）=========================");
        System.out.println(jedis.sadd("eleSet", "e1", "e2", " e4", "e3", " e0", " e8", "e7", "e5"));
        System.out.println(jedis.sadd("eleSet", "e6"));
        System.out.println(jedis.sadd("eleSet", "e6"));
        System.out.println("eleSet的所有元素为: " + jedis.smembers("eleSet"));
        System.out.println("删除一个元素eO:" + jedis.srem("eleSet", "e0"));
        System.out.println("eleSet的所有元素为: " + jedis.smembers("eleSet"));
        System.out.println("删除两个元素e7和e6:" + jedis.srem("eleSet", "e7", "e6"));
        System.out.println("eleSet的所有元素为: " + jedis.smembers("eleSet"));
        System.out.println("随机的移除集合中的一个元素:" + jedis.spop("eleSet"));
        System.out.println("随机的移除集合中的一个元素:" + jedis.spop("eleSet"));
        System.out.println("eleSet的所有元素为:" + jedis.smembers("eleSet"));
        System.out.println("eleSet中包含元素的个数:" + jedis.scard("eleSet"));
        System.out.println("e3是否在eleSet中: " + jedis.sismember("eleSet", "e3"));
        System.out.println("e1是否在eleSet中: " + jedis.sismember("eleSet", "e1"));
        System.out.println("e5是否在eleSet中: " + jedis.sismember("eleSet", "e5"));
        jedis.flushdb();
        System.out.println("======================================================================");
        System.out.println(jedis.sadd("eleSet1", "e1", " e2", " e4", " es3", "e0", " e8", "e7", "e5"));
        System.out.println(jedis.sadd("eleSet2", "e1", " e2", "e4", "e3", " eo", "e8"));
        System.out.println("将eleSet1中删除e1并存入eleSet3中: " + jedis.smove("eleSet1", "eleSet3", "e1"));
        System.out.println("eleSet1中的元素: " + jedis.smembers("eleSet1"));
        System.out.println("eleSet3中的元素: " + jedis.smembers("eleSet3"));
        System.out.println("=========================集合运算==================================");
        System.out.println("e1eSet1中的元素:" + jedis.smembers("eleSet1"));
        System.out.println("eleSet2中的元素: " + jedis.smembers("eleSet2"));
        System.out.println("eleSet1和eleSet2的交集: " + jedis.sinter("eleSet1", "eleSet2"));
        System.out.println("eleSet1和eleSet2的并集: " + jedis.sunion("eleSet1", "eleSet2"));
        System.out.println("eleSet1和eleSet2的差集:" + jedis.sdiff( "eleSet1", " eleSet2"));//eLeSet1中有,eleSet2没有
        jedis.sinterstore( "eleSet4" ,"eleSet1" , "eleSet2");//求交集并将交集保存到dsthey的集合
        System.out.println("eleSet4中的元素:"+jedis.smembers("eleSet4"));

    }
}
```

**Hash**

```java
package com.xiang;

import redis.clients.jedis.Jedis;

import java.util.HashMap;
import java.util.Map;

public class TestHash {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        Map<String, String> map = new HashMap<>();
        map.put("key1", "value1");
        map.put("key2", "value2");
        map.put("key3", "value3");
        map.put("key4", "value4");
        //添加名称为hash (key ) 的hash元素
        jedis.hmset("hash", map);
        //向名称为hash 的hash中添加key为key5,vaLue为vaLue5元素
        jedis.hset("hash", "key5", "value5");
        System.out.println("散列hash的所有键值对为: " + jedis.hgetAll("hash"));//return Map<s
        System.out.println("散列hash的所有键为:" + jedis.hkeys("hash")); //return Set<String>
        System.out.println("散列hash的所有值为:" + jedis.hvals("hash")); //return List<String>
        System.out.println("将k6保存的值加上一个整数，如果k6不存在则添加k6: "+jedis.hincrBy("hash","key6",6));
        System.out.println("散列hash的所有值为:" + jedis.hgetAll("hash"));
        System.out.println("将k6保存的值加上一个整数，如果k6不存在则添加k6: "+jedis.hincrBy("hash","key6",6));
        System.out.println("散列hash的所有键值对为:" + jedis.hgetAll("hash"));
        System.out.println("删除一个或者多个键值对: " + jedis.hdel("hash", "key2"));
        System.out.println("散列hash的所有键值对为:" + jedis.hgetAll("hash"));
        System.out.println("散列hash中键值对的个数: " + jedis.hlen("hash"));
        System.out.println("判断hash中是否存在key2: " + jedis.hexists("hash", "key2"));
        System.out.println("判断hash中是否存在key3: " + jedis.hexists("hash", "key3"));
        System.out.println("获取hash中的值:" + jedis.hmget("hash", "key3"));
        System.out.println("获取hash中的值: " + jedis.hmget("hash",  "key3","key4"));
    }
}
```

>Jedis操作事务

```java
import com.alibaba.fastjson.JSONObject;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.Transaction;

public class TestTx {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("hello","world");
        jsonObject.put("name","rlxiang");
        String relust = jsonObject.toString();
        Transaction multi = jedis.multi();// 开启事务
        try {
            multi.set("user1",relust);
            multi.set("user2",relust);
            int i = 1 / 0; // 代码抛出异常，事务执行失败
            multi.exec();
        } catch (Exception e) {
//            multi.discard(); // 如果事务出错就关闭事务，对应mysql中的回滚等类似的操作
            e.printStackTrace();
        } finally {
            System.out.println(jedis.get("user1"));
            System.out.println(jedis.get("user2"));
            jedis.close();
        }
```

>jedis 操作乐观锁 (watch)

```java
package com.xiang;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.Transaction;

public class TestWatch {
    public static void main(String[] args) throws InterruptedException {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        jedis.set("money","100");
        jedis.set("out","0");
        jedis.watch("money");
        Transaction multi = jedis.multi();
        Thread.sleep(1000);
        multi.incrBy("money",20);
        new Thread(new Runnable() {
            @Override
            public void run() {
                Jedis jedis = new Jedis("127.0.0.1", 6379);
                jedis.set("money","1000");
            }
        }).start();

        multi.exec();
        System.out.println(jedis.get("money")); // 1000, 表明之前的事务提价失败
    }
}

```

## SpringBoot整合

springboot操作数据：spring-data jpa jdbc redis,这些操作数据的操作都在springdata中

说明：在SpringBoot2.x之后，原来使用的jedis被替换为了lettuce

jedis:  jedis底层采用的是直连，多个线程操作的话，是不安全的，如果想要避免不安全，需要使用jedis pool连接池，有点像 BIO(阻塞)模式

lettuce: 采用netty，实例可以再多个线程中进行共享，不存在线程不安全的情况! 可以减少线程数量，不需要连接池了，有点像NIO模式，非阻塞式



源码分析

```java
	@Bean
	@ConditionalOnMissingBean(name = "redisTemplate") // 我们可以自己定义一个redisTemplate来替换这个默认的！
	@ConditionalOnSingleCandidate(RedisConnectionFactory.class)
	public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
		// 默认的 RedisTemplate 没有过多的设置，redis对象都是需要序列化的
        // 两个泛型都是 Object 的类型，我们之后使用需要强制转换 我们希望的类型是<String,Obeject>
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

	@Bean
	@ConditionalOnMissingBean // 由于 String 是 redis中最常使用的类型，所以单独提出来了一个bean
	@ConditionalOnSingleCandidate(RedisConnectionFactory.class)
	public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
		StringRedisTemplate template = new StringRedisTemplate();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}
```



>整合测试

1、导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

2、配置连接

```properties
# 配置redis
spring.redis.host=127.0.0.1
spring.redis.port=6379
```

3、测试

```java
@SpringBootTest
class Redis02SpringbootApplicationTests {

    @Autowired
    RedisTemplate redisTemplate;

    @Test
    void contextLoads() {
        // redisTempLate操作不同的数据类型，api和我们的指令是一样的lopsForVaLue操作字符串类似String
        // opsForList操作List类似List
        // opsForSet
        // opsForHash
        // opsForZSet
        // opsForGeo
        // opsForHyperLogLog
        // 除了进本的操作，我们常用的方法都可以直接通过redisTemplate操作，比如事务，和基本的CRUD


            // 获取数据库的连接对象，一般很少去使用
//        RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
//        connection.flushDb();
//        connection.flushAll();
        redisTemplate.opsForValue().set("mykey","向锐龙");
        System.out.println(redisTemplate.opsForValue().get("mykey"));

    }
```

![image-20210806171635212](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210806171635212.png)

![image-20210806171920969](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210806171920969.png)

关于对象的保存：

<img src="C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210806202109412.png" alt="image-20210806202109412" style="zoom:80%;" />

编写自己的RedisTemplate

```java
package com.xrl.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.jsontype.impl.LaissezFaireSubTypeValidator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig {

    // 自己定义RedisTemplate
    // 固定的模板，可以不用修改，就可以直接使用
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        // 源码中自带的
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);

        // Json序列化配置
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);

        jackson2JsonRedisSerializer.setObjectMapper(om);
        //String的序列化
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        //key采用String的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        // value序列化方式采用jackson
        template.setValueSerializer(jackson2JsonRedisSerializer);
        // hash的value序列化采用jaskon
        template.setHashValueSerializer(jackson2JsonRedisSerializer);

        template.afterPropertiesSet(); // 将我们自定义的设置加入到template中
        return template;
    }
}

```

RedisUtils

```java
package com.xrl.utils;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;
import org.springframework.util.CollectionUtils;

import java.util.Collection;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.TimeUnit;

@Component
public final class RedisUtils {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    // =============================common============================

    /**
     * 指定缓存失效时间
     *
     * @param key  键
     * @param time 时间(秒)
     */
    public boolean expire(String key, long time) {
        try {
            if (time > 0) {
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据key 获取过期时间
     *
     * @param key 键 不能为null
     * @return 时间(秒) 返回0代表为永久有效
     */
    public long getExpire(String key) {
        return redisTemplate.getExpire(key, TimeUnit.SECONDS);
    }


    /**
     * 判断key是否存在
     *
     * @param key 键
     * @return true 存在 false不存在
     */
    public boolean hasKey(String key) {
        try {
            return redisTemplate.hasKey(key);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 删除缓存
     *
     * @param key 可以传一个值 或多个
     */
    @SuppressWarnings("unchecked")
    public void del(String... key) {
        if (key != null && key.length > 0) {
            if (key.length == 1) {
                redisTemplate.delete(key[0]);
            } else {
                redisTemplate.delete((Collection<String>) CollectionUtils.arrayToList(key));
            }
        }
    }


    // ============================String=============================

    /**
     * 普通缓存获取
     *
     * @param key 键
     * @return 值
     */
    public Object get(String key) {
        return key == null ? null : redisTemplate.opsForValue().get(key);
    }

    /**
     * 普通缓存放入
     *
     * @param key   键
     * @param value 值
     * @return true成功 false失败
     */

    public boolean set(String key, Object value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 普通缓存放入并设置时间
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */

    public boolean set(String key, Object value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
            } else {
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 递增
     *
     * @param key   键
     * @param delta 要增加几(大于0)
     */
    public long incr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递增因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, delta);
    }


    /**
     * 递减
     *
     * @param key   键
     * @param delta 要减少几(小于0)
     */
    public long decr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递减因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, -delta);
    }


    // ================================Map=================================

    /**
     * HashGet
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     */
    public Object hget(String key, String item) {
        return redisTemplate.opsForHash().get(key, item);
    }

    /**
     * 获取hashKey对应的所有键值
     *
     * @param key 键
     * @return 对应的多个键值
     */
    public Map<Object, Object> hmget(String key) {
        return redisTemplate.opsForHash().entries(key);
    }

    /**
     * HashSet
     *
     * @param key 键
     * @param map 对应多个键值
     */
    public boolean hmset(String key, Map<String, Object> map) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * HashSet 并设置时间
     *
     * @param key  键
     * @param map  对应多个键值
     * @param time 时间(秒)
     * @return true成功 false失败
     */
    public boolean hmset(String key, Map<String, Object> map, long time) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @param time  时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value, long time) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 删除hash表中的值
     *
     * @param key  键 不能为null
     * @param item 项 可以使多个 不能为null
     */
    public void hdel(String key, Object... item) {
        redisTemplate.opsForHash().delete(key, item);
    }


    /**
     * 判断hash表中是否有该项的值
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return true 存在 false不存在
     */
    public boolean hHasKey(String key, String item) {
        return redisTemplate.opsForHash().hasKey(key, item);
    }


    /**
     * hash递增 如果不存在,就会创建一个 并把新增后的值返回
     *
     * @param key  键
     * @param item 项
     * @param by   要增加几(大于0)
     */
    public double hincr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, by);
    }


    /**
     * hash递减
     *
     * @param key  键
     * @param item 项
     * @param by   要减少记(小于0)
     */
    public double hdecr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, -by);
    }


    // ============================set=============================

    /**
     * 根据key获取Set中的所有值
     *
     * @param key 键
     */
    public Set<Object> sGet(String key) {
        try {
            return redisTemplate.opsForSet().members(key);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }


    /**
     * 根据value从一个set中查询,是否存在
     *
     * @param key   键
     * @param value 值
     * @return true 存在 false不存在
     */
    public boolean sHasKey(String key, Object value) {
        try {
            return redisTemplate.opsForSet().isMember(key, value);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 将数据放入set缓存
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSet(String key, Object... values) {
        try {
            return redisTemplate.opsForSet().add(key, values);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 将set数据放入缓存
     *
     * @param key    键
     * @param time   时间(秒)
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSetAndTime(String key, long time, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().add(key, values);
            if (time > 0)
                expire(key, time);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 获取set缓存的长度
     *
     * @param key 键
     */
    public long sGetSetSize(String key) {
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 移除值为value的
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 移除的个数
     */

    public long setRemove(String key, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().remove(key, values);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    // ===============================list=================================

    /**
     * 获取list缓存的内容
     *
     * @param key   键
     * @param start 开始
     * @param end   结束 0 到 -1代表所有值
     */
    public List<Object> lGet(String key, long start, long end) {
        try {
            return redisTemplate.opsForList().range(key, start, end);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }


    /**
     * 获取list缓存的长度
     *
     * @param key 键
     */
    public long lGetListSize(String key) {
        try {
            return redisTemplate.opsForList().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }


    /**
     * 通过索引 获取list中的值
     *
     * @param key   键
     * @param index 索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
     */
    public Object lGetIndex(String key, long index) {
        try {
            return redisTemplate.opsForList().index(key, index);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }


    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     */
    public boolean lSet(String key, Object value) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     */
    public boolean lSet(String key, Object value, long time) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }

    }


    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @return
     */
    public boolean lSet(String key, List<Object> value) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }

    }


    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value, long time) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 根据索引修改list中的某条数据
     *
     * @param key   键
     * @param index 索引
     * @param value 值
     * @return
     */

    public boolean lUpdateIndex(String key, long index, Object value) {
        try {
            redisTemplate.opsForList().set(key, index, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }


    /**
     * 移除N个值为value
     *
     * @param key   键
     * @param count 移除多少个
     * @param value 值
     * @return 移除的个数
     */

    public long lRemove(String key, long count, Object value) {
        try {
            Long remove = redisTemplate.opsForList().remove(key, count, value);
            return remove;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }

    }

}
```



## Redis.conf详解

启动的时候，就是通过配置文件启动的

>单位

![image-20210806213330251](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210806213330251.png)

1、配置文件 unit单位对大小写不敏感

>可以包含其他redis的配置文件

![image-20210806213540988](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210806213540988.png)

就好比我们学习Spring中的Import

>网络

```bash
bind 127.0.0.1 -::1 # 绑定的ip
protected-mode yes # 保护模式
port 6379  # 端口
```

>通用配置 GENERAL

```bash
daemonize yes # 以守护进程的方式启动(就是在后台运行的意思)，默认是no，我们需要手动改为yes
pidfile /var/run/redis_6379.pid # 如果以后台的方式运行，我们就需要指定以个pid文件

#日志
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing) 开发或者测试环境下使用
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably) 生产环境下使用
# warning (only very important / critical messages are logged)
loglevel notice
logfile "/home/rlxiang/log/redis_log.log" # 日志文件位置名，手动配置需要我们
databases 16 # 数据库数量
always-show-logo no # 是否总是显示logo





```

>快照

表示，在规定时间内，执行多少次操作，就会持久化到文件，文件有两种类型 一种是.rdb, 一种是.aof

redis是内存数据库，如果没有持久化，那么数据就会断电即失

```bash
# 在最新版的redis中，以下配置都被注释了
save 3600 1 # 如果在3600s内，如果至少有1个key进行了修改，就会被持久化
save 300 100 # 如果在300s内，如果至少有100个key进行了修改，就会被持久化
save 60 10000 # 如果在300s内，如果至少有10000个key进行了修改，就会被持久化

stop-writes-on-bgsave-error yes # 持久化如果失败，redis是否基序工作

rdbcompression yes # 是否压缩rdb文件，这时候会消耗一些cpu资源

rdbchecksum yes # 保存rdb文件的时候，是否进行错误的检查校验

dir /usr/local/bin/redis_dbfile/ # rdb文件保存的目录，手动配置
```

>REPLICATION复制，在主从复制的时候详见





>SECURITY 安全

可以在这里设置redis的密码，默认是没有密码的！

```bash
127.0.0.1:6379> config get requirepass # 获取redis密码
1) "requirepass"
2) ""
127.0.0.1:6379> config set requirepass "123456" # 设置redis密码，但是新版设置成功后要重启redis服务也不生效
OK
127.0.0.1:6379> auth 123456 # 如果设置密码成功，需要使用密码登录
```



>限制 CLIENTS (基本上这些东西我们都不会去配置)

```bash
maxclients 10000 # 设置能连接的最大客户端数
maxmemory <bytes> # redis 配置最大内存
maxmemory-policy noeviction # 内存达到上限后的处理策略
    1、volatile-lru：只对设置了过期时间的key进行LRU（默认值） 
    2、allkeys-lru ： 删除lru算法的key   
    3、volatile-random：随机删除即将过期key   
    4、allkeys-random：随机删除   
    5、volatile-ttl ： 删除即将过期的   
    6、noeviction ： 永不过期，返回错误
```



>APPEND ONLY MODE aof 配置

```bash
appendonly no # 默认是不开启的，默认是使用rdb方式持久化的，在大部分的情况下，rdb完全够用
appendfilename "appendonly.aof" # 持久化文件的名字

# appendfsync always # 每次修改都会 sync，消耗性能
appendfsync everysec  # 每秒执行一次同步，可能会丢失这1s的数据
# appendfsync no  # 不执行 syc，这个时候操作系统会自己同步数据，速度最快
```

持久化的配置，在Redis持久化中详见



## Redis持久化

Redis是内存数据库，如果不将内存中的数据库状态保存到磁盘，那么一旦服务器进程退出，服务器中的数据库状态也会消失。所以Redis提供了持久化功能!

### RDB（Redis DataBase）

>什么是RDB

![image-20210807092910548](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807092910548.png)

在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里。

Redis会单独创建 （ fork )一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的。这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。我们默认的就是RDB，一般情况下不需要修改这个配置

有时候在生产环境我们会备份dump.rdb

==rdb保存的文件就是 dump.rdb== 都是在我们的配置文件中的快照中进行配置的！

![image-20210807100036507](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807100036507.png)

![image-20210807100504771](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807100504771.png)

>触发机制

1、save的规则满足的情况下，会自动触发rdb规则，生成rdb文件

2、执行flushall命令，也会触发rdb规则，生成rdb文件

3、退出redis，也会产生rdb文件

备份就会自动生成一个dump.rdb文件

>如何恢复rdb文件！

1、只需要将rdb文件放在我们redis启动目录就可以，redis启动的时候会自动检查dump.rdb文件，恢复其中的数据

```bash
127.0.0.1:6379> config get dir
1) "dir"
2) "/usr/local/bin/redis_dbfile"  # 如果在这个目录下存在 dump.rdb文件，启动的时候就会自动恢复其中的数据
```

>redis中的对rdb的默认配置几乎就够用了，但是还是要学习

**优点：**

1、适合大规模的数据恢复！

2、如果对数据的完整性要求不高也可以使用rdb，比如60s内修改至少5次key的场景，如果在第59只修改了4次key，并且此时宕机，那么这一次修改的备份就没有了

**缺点：**

1、需要一定的时间间隔进行操作，如果redis意外宕机，那么redis最后一次修改的数据就没有了！

2、fork(创建)另一条进程的时候，会占用一定的内存空间





### AOF (Append Only File)

将我们的所有命令都记录下来，就像是history一样，恢复的时候，就把这个文件执行一遍

>是什么

![image-20210807110157341](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807110157341.png)

以日志的形式来记录每个写操作，将Redis执行过的所有指令记录下来(读操作不记录），只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

==Aof保存的是 appendonly.aof文件==



>append

![image-20210807110903278](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807110903278.png)

默认是不开启的，我们需要手动配置！我们只需要将appendonly改为yes就可以支持aof了，而且系统自带的配置一般也不需要改动。

只要重启redis服务，该aof文件就会生成

如果这个aof文件有错误，这个时候 redis 是启动不起来的，我们需要修复这个aof文件

redis 给我提供了解决这样的问题的操作工具 `redis-check-aof`, 命令为 `redis-check-aof --fix`

```bash
rlxiang@xpjiang-System-Product-Name:/usr/local/bin$ redis-check-aof --fix redis_dbfile/appendonly.aof 
0x              a4: Expected \r\n, got: 6461
AOF analyzed: size=189, ok_up_to=139, ok_up_to_line=40, diff=50
This will shrink the AOF from 189 bytes, with 50 bytes, to 139 bytes
Continue? [y/N]: y
Successfully truncated AOF # 修复成功

```

如果文件正常，重启redis就可以直接恢复了！

![image-20210807123743661](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807123743661.png)

>重写规则说明(了解下)

aof默认就是无限追加文件，这会导致文件越来越大

![image-20210807124624330](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807124624330.png)

如果aof的文件大于 64m，这样的话这个aof文件太大了！此时就在再次fork一个新的进程来将我们的文件进行重写。aof下，redis每次都会记录上一次aof文件的大小，如果超过了大小就会触发重写。



>优点和缺点！

```bash
appendonly no # 默认是不开启的，默认是使用rdb方式持久化的，在大部分的情况下，rdb完全够用
appendfilename "appendonly.aof" # 持久化文件的名字

# appendfsync always # 每次修改都会 sync，消耗性能
appendfsync everysec  # 每秒执行一次同步，可能会丢失这1s的数据
# appendfsync no  # 不执行 syc，这个时候操作系统会自己同步数据，速度最快
```



优点：

1、每一次修改都同步，文件的完整性会更好！

2、默认是每秒同步，可能会丢失一秒的数据

3、从不同步，效率最高

缺点：

1、相对于数据文件来说，aof远远大于rdb，修复的速度也比rdb慢

2、Aof运行效率也比rdb慢，所以我们的redis默认的配置就是rdb持久化



**扩展：**

1、RDB持久化方式能够在指定的时间间隔内对你的数据进行快照存储

2、AOF持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以Redis 协议追加保存每次写的操作到文件末尾，Redis还能对AOF文件进行后台重写，使得AOF文件的体积不至于过大。

3、只做缓存，如果你只希望你的数据在服务器运行的时候存在，你也可以不使用任何持久化

4、同时开启两种持久化方式：

- 在这种情况下，当redis重启的时候会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。
- RDB的数据不实时，同时使用两者时服务器重启也只会找AOF文件，那要不要只使用AOF呢?作者建议不要，因为RDB更适合用于备份数据库（AOF在不断变化不好备份），快速重启，而且不会有AOF可能潜在的Bug，留着作为一个万一的手段。

5、性能建议

- 因为RDB文件只用作后备用途，建议只在Slave(从机)上持久化RDB文件，而且只要15分钟备份一次就够了，只保留save 900 1这条规则。
- 如果开启AOF，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了，代价一是带来了持续的IO，二是AOF rewrite 的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上，默认超过原大小100%大小重写可以改到适当的数值。
- 如果不开启AOF，仅靠Master-Slave Replcation(主从复制)实现高可用性也可以，能省掉一大笔lO，也减少了rewrite时带来的系统波动。代价是如果Master/Slave同时宕掉，会丢失十几分钟的数据，启动脚本也要比较两个Master/Slave 中的RDB文件，载入较新的那个，微博就是这种架构



## Redis发布订阅

Redis 发布订阅(pub/sub)是一种==消息通信模式==︰发送者(pub)发送消息，订阅者(sub)接收消息。比如微信公众号、微博，有关注系统的都可以

Redis客户端可以订阅任意数量的频道。

订阅/发布消息图：

第一个：消息发送者，第二个：频道，但三个：消息订阅者

![image-20210807130405858](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807130405858.png)

下图展示了频道channel1，以及订阅这个频道的三个客户端--client2、client5和client1之间的关系：

![image-20210807150603656](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807150603656.png)

当有新消息通过PUBLISH命令发送给频道channel1时，这个消息就会被分发给订阅它的三个客户端：

![image-20210807150758298](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807150758298.png)



>命令

这些命令被广泛用于构建即时通信应用，比如网络聊天室(chatroom)和实时广播、实时提醒等。

![image-20210807150852358](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807150852358.png)

>测试

订阅端

```bash
127.0.0.1:6379> subscribe xiangruilong  # 订阅一个频道 xiangruilong
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "xiangruilong"
3) (integer) 1
# 等待频道发送消息
1) "message" # 消息
2) "xiangruilong" # 来自那个频道
3) "hello,xiangruilong" # 消息的具体内容

1) "message"
2) "xiangruilong"
3) "hello,redis"
```



发送端

```bash
127.0.0.1:6379> PUBLISH xiangruilong "hello,xiangruilong" # 发布者往频道发送消息
(integer) 1
127.0.0.1:6379> PUBLISH xiangruilong "hello,redis"
(integer) 1

```



>实现原理

Redis是使用C实现的，通过分析Redis源码里的pubgub.c文件，了解发布和订阅机制的底层实现，籍此加深对Redis的理解。Redis通过PUBLISH 、SUBSCRIBE和PSUBSCRIBE等命令实现发布和订阅功能。

通过SUBSCRIBE 命令订阅某频道后，redis-server里维护了一个字典，字典的键就是一个个channel(频道)，而字典的值则是一个链表，链表中保存了所有订阅这个channel的客户端。SUBSCRIBE 命令的关键，就是将客户端添加到给定channel的订阅链表中。

通过PUBLISH 命令向订阅者发送消息，redis-server会使用给定的频道作为键，在它所维护的channel 字典中查找记录了订阅这个频道的所有客户端的链表，遍历这个链表，将消息发布给所有订阅者。

Pub/Sub从字面上理解就是发布( Publish )与订阅(Subscribe )，在Redis中，你可以设定对某一个key值进行消息发布及消息订阅，当一个key值上进行了消息发布后，所有订阅它的客户端都会收到相应的消息。这一功能最明显的用法就是用作实时消息系统，比如普通的即时聊天，群聊等功能。



使用场景：

1、实现消息系统！

2、事实聊天！（频道当做聊天室，将信息回显给所有人）

3、订阅，关注系统都是可以的！





## Redis主从复制

**概念**

主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(masterleader)，后者称为从节点(slave/follower);数据的复制是单向的，只能由主节点到从节点。Master以写为主，Slave以读为主。

==默认情况下，每台Redis服务器都是主节点;且一个主节点可以有多个从节点(或没有从节点)，但一个从节点只能有一个主节点。==

主从复制的作用主要包括：

1、数据冗余︰主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。

2、故障恢复∶当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复;实际上是一种服务的冗余。

3、负载均衡∶在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载;尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。

4、高可用(集群)基石∶除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。

一般来说，要将Redis运用于工程项目中，只使用一台Redis是万万不能（可能会宕机）的，原因如下∶

1、从结构上，单个Redis服务器会发生单点故障，并且一台服务器需要处理所有的请求负载，压力较大﹔

2、从容量上，单个Redis服务器内存容量有限，就算一台Redis服务器内存容量为256G，也不能将所有内存用作Redis存储内存，一般来说，==单台Redis最大使用内存不应该超过20G。==

电商网站上的商品，一般都是一次上传，无数次浏览的，说专业点也就是"多读少写"。

对于这种场景，我们可以使如下这种架构︰	 

![image-20210807154710950](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807154710950.png)

主从复制，读写分离！一般80%的情况下都是在进行读操作！减缓服务器的压力！架构中经常使用，最低要求是一主二仆

只要在公司中，主从复制就是必须要使用的，在真实的项目中不可能使用单机使用Redis



### 环境配置

只配置从库，不用不配置主库，因为Redis默认自己就是一个主库

```bash
127.0.0.1:6379> info replication # 查看当前库的信息
# Replication
role:master # 角色 master
connected_slaves:0 # 连接从机的个数， 这里是0
master_failover_state:no-failover
master_replid:f2dc0c1c794fc19979fed53e3aace9c8fc2ce15a
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

复制2个配置文件，然后修改对应的信息

1、端口

2、pid 名字

3、log文件名字

4、dump.rdb名字

修改完毕后启动三个redis服务，可以通过进程查看

![image-20210807164541435](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807164541435.png)



### 一主二从

==默认情况下，每台Redis服务器都是主节点==，我们一般情况只配置从机

一主(6379)二从(80、81)

```bash
127.0.0.1:6380> SLAVEOF 127.0.01 6379 # SLAVEOF 127.0.01 6379 认6379为主机
OK
127.0.0.1:6380> info replication
# Replication
role:slave   # 身份变成了从机
master_host:127.0.01 # 主机的信息
master_port:6379
master_link_status:up
master_last_io_seconds_ago:6
master_sync_in_progress:0
slave_repl_offset:14
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:2c8f5f6474f37dcb65195228276c496b4c74511d
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:14
second_repl_offset:-1

# 在主机中查看
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:1 # 多了从机的配置
slave0:ip=127.0.0.1,port=6380,state=online,offset=28,lag=1 # 从机的配置
master_failover_state:no-failover
master_replid:2c8f5f6474f37dcb65195228276c496b4c74511d
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:28
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:28
```

从机配置完毕后，可以在主机中看到连接了几个从机

![image-20210807170542732](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807170542732.png)

上面的设置都是使用的命令设置的，是一次行的，真实的场景都是在配置文件中配置的，就是在从机的配置文件中配置主机的ip和端口，如果主机有密码认证，也可以配置一下即可

![image-20210807171009366](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807171009366.png)



>细节

主机主要负责写，从机不能写！主机中的所有信息和数据，都会被从机自动保存

主机写：

![image-20210807171333140](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807171333140.png)

从机只能读取内容，如果在从机中操作了写，从机就会报错

![image-20210807171500516](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807171500516.png)

测试： 主机断开连接，从机依旧与主机相连接，但是此时没有写的操作了，这个时候，如果主机上线了，从机依旧可以直接取到主机上线后存的数据！

如果是使用命令行来配置的主从，如果这个时候重启主机，这个时候从机会变成主机！如果此时再将该机变成从机，那么主机中的数据从机也是全有的

>复制原理

Slave启动成功连接到 master后会发送一个sync同步命令

Master接到命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，==master将传送整个数据文件到slave，并完成一次完全同步==。

==全量复制==:而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。

==增量复制==:Master继续将新的所有收集到的修改命令依次传给slave，完成同步

但是只要是重新连接master，一次完全同步(全量复制)将被自动执行！我们的数据一定可以在从机中看到



>层层链路

![image-20210807201652521](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807201652521.png)

这种模型也可以完成主从复制，80服务器依然不能写入。

>如果没有了主机，这个时候能不能选择一个从机作为主机？ 手动配置即可

==谋朝篡位==

如果主机断开了连接，我们可以使用`slaveof no one` 让当前机器变成主机！其他的节点就可以手动连接到最新的这个主节点(这一切操作都是手动完成的)，如果此时主机 回来了，也只能重新连接才行

![image-20210807202129594](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807202129594.png)





### 哨兵模式

（自动选举主机）

>概述

主从切换技术的方法是∶当主服务器宕机后，需要手动把一台从服务器切换为主服务器，这就需要人工干预，费事费力，还会造成一段时间内服务不可用。这不是一种推荐的方式，更多时候，我们优先考虑哨兵模式。Redis从2.8开始正式提供了Sentinel (哨兵）架构来解决这个问题。

谋朝篡位的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将**从库转换为主库**。

哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。其原理是**哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。**

![image-20210807203307410](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807203307410.png)

这里的哨兵有两个作用

- 通过发送命令，让Redis服务器返回监控其运行状态，包括主服务器和从服务器。
- 当哨兵监测到master宕机，会自动将slave切换成master，然后通过发布订阅模式通知其他的从服务器，修改配置文件，让它们切换主机。

然而一个哨兵进程对Redis服务器进行监控，可能会出现问题，为此，我们可以使用多个哨兵进行监控。各个哨兵之间还会进行互相监控，这样就形成了多哨兵模式。

![image-20210807203716361](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807203716361.png)

假设主服务器宕机，哨兵1先检测到这个结果，系统并不会马上进行failover过程，仅仅是哨兵1主观的认为主服务器不可用，这个现象成为**主观下线**。当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票的结果由随机一个哨兵发起，进行failover[故障转移]操作。切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，这个过程称为**客观下线**。

>测试

我们当前的状态是一主二从

1、配置哨兵配置文件 sentinel.conf，名字不能写错

```bash
# 语法格式： sentinel monitor(监视) 被监视的名字(这个可以随便写) host port 1
sentinel monitor myredis 127.0.0.1 6379 1
```

后面的这个数字1，代表当主机挂了，哨兵就会投票让那个从机来接替主机，谁的票数多谁就是主机

2、启动哨兵

```bash
rlxiang@xpjiang-System-Product-Name:/usr/local/bin$ sudo redis-sentinel xconfig/sentinel.conf # 启动哨兵
468268:X 07 Aug 2021 20:56:00.558 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
468268:X 07 Aug 2021 20:56:00.558 # Redis version=6.2.5, bits=64, commit=00000000, modified=0, pid=468268, just started
468268:X 07 Aug 2021 20:56:00.558 # Configuration loaded
468268:X 07 Aug 2021 20:56:00.559 * Increased maximum number of open files to 10032 (it was originally set to 1024).
468268:X 07 Aug 2021 20:56:00.559 * monotonic clock: POSIX clock_gettime
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 6.2.5 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                  
 (    '      ,       .-`  | `,    )     Running in sentinel mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 26379
 |    `-._   `._    /     _.-'    |     PID: 468268
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           https://redis.io       
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

468268:X 07 Aug 2021 20:56:00.575 # Sentinel ID is 13dabb2e4aabb0c9f603b35a6d1f7edbb54d5fa7
468268:X 07 Aug 2021 20:56:00.575 # +monitor master myredis 127.0.0.1 6379 quorum 1
468268:X 07 Aug 2021 20:56:00.576 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6379
468268:X 07 Aug 2021 20:56:00.583 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6379
```

如果Master 节点断开了，这个时候就会从从机中随机选择一个服务！（这里有个投票算法）

![image-20210807210345863](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807210345863.png)



哨兵日志

![image-20210807210601821](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807210601821.png)

如果原来的主机回来了，也只能归并到新的主机下，当做从机，这就是哨兵规则！



>哨兵模式

优点：

1、哨兵集群，是基于主从复制模式，所有的主从复制的优点，他全有

2、 主从可以切换，故障可以转移(这里的意思是，切换主机)，系统可用性就会更好

3、哨兵模式就是主从模式的升级，从原来的手动配置变成自动配置，更加健壮！

缺点：

1、Redis是不好在线扩容的，集群容量一旦达到上限，在线扩容就十分麻烦

2、实现哨兵模式的配置其实是十分麻烦的，里面有很多选择！



>哨兵模式的全部配置！

```bash
#Example sentinel.conf

#哨兵sentine1实例运行的端口默认26379,如果有多个哨兵就需要配置多个端口，然后像启动多个redis服务一样去启动哨兵
port 26379

#哨兵sentine1的工作目录
dir /tmp

# 哨兵sentine1监控的redis主节点的 ip port
# master-name可以自己命名的主节点名字只能由字母A-z、数字0-9 、这三个字符".-_"组成。
# quorum配置多少个sentine1哨兵统一认为master主节点失联那么这时客观上认为主节点失联了
# sentine7 monitor <master-name> <ip> <redis-port> <quorum>
sentinel monitor mymaster 127.0.0.1 6379 2

#当在Redis实例中开启了requirepass foobared授权密码这样所有连接Redis实例的客户端都要提供密码
#设置哨兵sentine1连接主从的密码注意必须为主从设置一样的验证密码
# sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster MySUPER--secret-0123passwOrd

#指定多少毫秒之后主节点没有应答哨兵sentine1此时哨兵主观上认为主节点下线默认30秒
#- sentinel down-after-mi1liseconds <master-name> <milliseconds>
sentinel down-after-mi1liseconds mymaster 30000

#这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行同步，这个数字越小，完成failover所需的时
#间就越长，但是如果这个数字越大，就意味着越多的s1ave因为replication而不可用。
#可以通过将这个值设为1来保证每次只有一个slave 处于不能处理命令请求的状态
# sentinel para11e1-syncs <master-name> <nums1aves>
sentinel para1le1-syncs mymaster 1

#故障转移的超时时间failover-timeout可以用在以下这些方面:
#1.同一个sentine1对同一个master两次failover之间的间隔时间。
#2．当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
#3.当想要取消一个正在进行的failover所需要的时间。
#4.当进行failover时，配置所有s1aves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按para7le1-syncs所配置的规则来了
#默认三分钟
#sentinel failover-timeout <master-name> <mi17iseconds>
sentinel failover-timeout mymaster 180000

# SCRIPTS EXECUTION

#配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时发邮件通知相关人员。
#对于脚本的运行结果有以下规则:
#若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10
#若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。
#如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。
#一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个STGKILL信号终止，之后重新执行。

#通知型脚本:当sentine1有任何警告级别的事件发生时(比如说redis实例的主观失效和客观失效等等)，将会去调用这个脚本，这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信息。调用该脚本时，将传给脚本两个参数，一个是事件的类型，一个是事件的描述。如果sentine1.conf配置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则sentine1无法正常启动成功。
#通知脚本
# sentinel notification-script <master-name> <script-path>
sentinel notification-script mymaster /var/redis/notify.sh

# 客户端重新配置主节点参数脚本
#当一个master由于fai1over而发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已经发生改变的信息。
#以下参数将会在调用脚本时传给脚本:
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
#目前<state>总是“failover"，
#<ro1e>是“leader”或者"observer”中的一个。
#参数 from-ip，from-port，to-ip，to-port是用来和旧的master和新的master(即旧的slave)通信的
#这个脚本应该是通用的，能被多次调用，不是针对性的。
# sentinel client-reconfig-script <master-name> <script-path>
sentinel client-reconfig-scripf mymaster /var/redis/reconfig.sh
```



## Redis缓存穿透与缓存雪崩

Redis缓存的使用，极大的提升了应用程序的性能和效率，特别是数据查询方面。但同时，它也带来了一些问题。其中，最要害的问题，就是数据的一致性问题，从严格意义上讲，这个问题无解。如果对数据的一致性要求很高，那么就不能使用缓存。



另外的一些典型问题就是，缓存穿透、缓存雪崩和缓存击穿。目前，业界也都有比较流行的解决方案。

![image-20210807213531892](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807213531892.png)

### 缓存穿透（没有待查找数据导致）

>概念

缓存穿透的概念很简单，用户想要查询一个数据，发现redis内存数据库没有，也就是缓存没有命中，于是向持久层数据库查询。发现也没有，于是本次查询失败。当用户很多的时候，缓存都没有命中，于是都去请求了持久层数据库。这会给持久层数据库造成很大的压力，这时候就相当于出现了缓存穿透。

>解决方案

**布隆过滤器**

布隆过滤器是一种数据结构，对所有可能查询的参数以hash形式存储，在控制层先进行校验，不符合则丢弃，从而避免了对底层存储系统的查询压力﹔

![image-20210807213847960](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807213847960.png)

**缓存空对象**

当存储层不命中后，即使返回的空对象也将其缓存起来，同时会设置一个过期时间，之后再访问这个数据将会从缓存中获取，保护了后端数据源;

![image-20210807213938436](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807213938436.png)

但是这种方法会存在两个问题：

1、如果空值能够被缓存起来，这就意味着缓存需要更多的空间存储更多的键，因为这当中可能会有很多的空值的键;

2、即使对空值设置了过期时间，还是会存在缓存层和存储层的数据会有一段时间窗口的不一致，这对于需要保持一致性的业务会有影响。



### 缓存击穿（量太大，缓存过期导致）

>概述

这里需要注意和缓存击穿的区别，缓存击穿，是指一个key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在一个屏障上凿开了一个洞。



当某个key在过期的瞬间，有大量的请求并发访问，这类数据一般是热点数据，由于缓存过期，会同时访问数据库来查询最新数据，并且回写缓存，会导使数据库瞬间压力过大。



>解决方案

**设置热点数据永不过期**

从缓存层面来看，没有设置过期时间，所以不会出现热点 key过期后产生的问题。



**加互斥锁**

分布式锁∶使用分布式锁，保证对于每个key同时只有一个线程去查询后端服务，其他线程没有获得分布式锁的权限，因此只需要等待即可。这种方式将高并发的压力转移到了分布式锁，因此对分布式锁的考验很大。

![image-20210807214740064](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807214740064.png)

### 缓存雪崩

>概念

缓存雪崩，是指在某一个时间段，缓存集中过期失效。比如Redis集群宕机！

产生雪崩的原因之一，比如在写本文的时候，马上就要到双十二零点，很快就会迎来一波抢购，这波商品时间比较集中的放入了缓存，假设缓存一个小时。那么到了凌晨一点钟的时候，这批商品的缓存就都过期了。而对这批商品的访问查询，都落到了数据库上，对于数据库而言，就会产生周期性的压力波峰。于是所有的请求都会达到存储层，存储层的调用量会暴增，造成存储层也会挂掉的情况。

![image-20210807215027462](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210807215027462.png)

其实集中过期，倒不是非常致命，比较致命的缓存雪崩，是缓存服务器某个节点宕机或断网。因为自然形成的缓存雪崩，一定是在某个时间段集中创建缓存，这个时候，数据库也是可以顶住压力的。无非就是对数据库产生周期性的压力而已。而缓存服务节点的宕机，对数据库服务器造成的压力是不可预知的，很有可能瞬间就把数据库压垮。

>解决方案

**Redis高可用**

这个思想的含义是，既然redis有可能挂掉，那我多增设几台redis，这样一台挂掉之后其他的还可以继续工作，其实就是搭建的集群。（异地多活）

**限流降级**

这个解决方案的思想是，在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。

**数据预热**

数据加热的含义就是在正式部署之前，我先把可能的数据先预先访问一遍，这样部分可能大量访问的数据就会加载到缓存中。在即将发生大并发访问前手动触发加载缓存不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀。