# Redis

## Nosql

大数据就是一般的数据库无法进行分析处理的

> 1.单机Mysql年代
>
> ![image-20200728205553535](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200728205553535.png)

那个时候的服务器不会有太大的压力

那个时候的网站的瓶颈:

1. 数据量太大，一个机器放不下
2. 数据的索引，一个机器内存也放不下
3. 访问量（读写混合），一个服务器承受不了

> 2.Memcache(缓存)+Mysql+垂直拆分(读写分离)

网站读操作多；

每次都要查询数据库会很麻烦，为了减轻数据库的压力，可以使用缓存保证效率。

发展过程：优化数据结构和索引->文件缓存（IO）->Memcached

![image-20200728210318504](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200728210318504.png)

> 3.分库分表+水平拆分+Mysql集群

**本质：数据库（读，写）**

早些年MyISM:表锁，十分影响效率！高并发下就睡出现严重的问题

Innodb:行锁

![image-20200728210548148](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200728210548148.png)

> 目前基本互联网

![](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200728211430220.png)

**NoSQL=Not only SQL（不仅仅是SQL ）**

泛指非关系型数据库

> NoSQL特点

解耦！

1. 方便扩展（数据之间没有关系）
2. 大数据量高性能（一秒写8万次）
3. 数据类型多样性（不需要实现设计数据库）
4. 传统的RDBMS和NoSQL

```
传统的RDBMS
-结构化组织
-SQL
-数据和关系都存在单独的表中
-数据操作，数据定义语言
-严格的一致性
-基础的事务
···
```

```
NoSQL
-不仅仅是数据
-没有固定的查询语言
-键值对存储，列存储，文档存储，图形数据库
-最终一致性
-CAP定理和BASE理论
-高性能，高可用，高并发
```

![image-20200728214456204](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200728214456204.png)

## NoSQL四大分类

**KV键值对**

**文档型数据库(bson和json)**

* MongoDB
  * 一个基于分布式文件存储的数据库,主要用来处理大量文档
  * 一个介于关系型数据库和非关系数据库中间产品,是非关系型数据库中功能最丰富,最想关系型数据库的.

**列存储**

**图关系数据库**

![image-20200728215226969](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200728215226969.png)

* 不是存图形的,放的是关系



![image-20200728215331092](C:\Users\yuangong\AppData\Roaming\Typora\typora-user-images\image-20200728215331092.png)



## Redis概述

> redis=remote dictionary server,远程字典服务

1. 内存存储、持久化，内存中是断电就丢失，所以持久化很重要
2. 效率高
3. 发布订阅系统
4. 地图信息分析
5. 计时器，计数器
6. ....

> 特性

1. 多样性的数据类型
2. 持久化
3. 集群
4. 事务

http://redis.cn/

> 测试性能

redis-benchmark是一个压力测试工具

命令可以搜索菜鸟教程

```bash
# 测试：100个并发连接， 100000请求
redis-benchmark -h localhost -p 6379 -c 100 -n 100000
```

## 基础知识

默认16个数据库

默认是0号数据库

**select** 切换数据库

**DBSIZE**查看当前数据库大小

**keys *** 查看数据库所有的key

**flushdb** 清除当前数据库

**flushAll** 清楚所有数据库

**move key** 1从一号库清除某个key

**exists key** 判断某个key是否在当前库存在

**expire key 10** 设置过期时间，单位是秒

**tll key** 查看当前key的剩余时间

**type key** 查看当前key的类型

> redis是单线程的

Redis是基于内存操作，Redis的瓶颈是根据机器的内存和网络带宽。

redis是将所有的数据全部放在内存中，所以说使用单线程去操作效率就是最高的，多线程（CPU上下文会切换：耗时的操作），对于内存系统来说，如果没有上下文切换效率就是最高的！多次读写都是在一个CPU上，在内存情况下，是最佳的。

## 五大数据类型

### String

```bash
###########################################################################
127.0.0.1:6379> set key1 v1
OK
127.0.0.1:6379> get key1
"v1"
127.0.0.1:6379> keys *
1) "key1"
127.0.0.1:6379> append key1 "hello"  #在尾部追加一个value1,如果key不存在，则创建一个key。
(integer) 7
127.0.0.1:6379> strlen key1 # 获取字符串的长度
(integer) 7
127.0.0.1:6379> append key1 "xiang"
(integer) 12
127.0.0.1:6379> strlen key1
(integer) 12
127.0.0.1:6379> get key1
"v1helloxiang"
###############################################################################
127.0.0.1:6379> set k 0
OK
127.0.0.1:6379> get k
"0"
127.0.0.1:6379> incr k # incr key 自动增长1
(integer) 1
127.0.0.1:6379> incr k
(integer) 2
127.0.0.1:6379> incr k
(integer) 3
127.0.0.1:6379> get k
"3"
127.0.0.1:6379> decr k #decr key 自动减小1
(integer) 2
127.0.0.1:6379> decr k
(integer) 1
127.0.0.1:6379> decr k
(integer) 0
127.0.0.1:6379> decr k
(integer) -1
127.0.0.1:6379> incrby k 10 #value增长num
(integer) 9
127.0.0.1:6379> incrby k 10
(integer) 19
127.0.0.1:6379> decrby k 5 #value减小num
(integer) 14
127.0.0.1:6379> decrby k 5
(integer) 9
###############################################################################

127.0.0.1:6379> set k "xiangjunju"
OK
127.0.0.1:6379> getrange k 0 4	#截取字符串[0,4]
"xiang"
127.0.0.1:6379> getrange k 0 -1 #获取全部字符串
"xiangjunju"
127.0.0.1:6379> setrange k 0 zh	#替换指定位置开始的字符串
(integer) 10
127.0.0.1:6379> get k
"zhangjunju"
###############################################################################
#setex(set with  expire) 设置过期时间
#setnx(set if not exist) 不存在的时候才能设置
127.0.0.1:6379> setex k3 30 "hello"
OK
127.0.0.1:6379> ttl key3
(integer) -2	#-2表示过期
127.0.0.1:6379> setnx k4 "redis"
(integer) 1
127.0.0.1:6379> setnx k4 "java"
(integer) 0
127.0.0.1:6379> keys *
1) "k4"
2) "k"
127.0.0.1:6379> get k4
"redis"
###############################################################################
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3	#批量插入
OK
127.0.0.1:6379> keys *
1) "k2"
2) "k1"
3) "k3"
127.0.0.1:6379> mget k1 k2 k3	#批量查看
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> msetnx k1 v1 k4 v4	#是一个原子性的操作，要么一起成功，要么一起失败
(integer) 0
127.0.0.1:6379> get k4
(nil)
#这里的key是一个巧妙地设计：user:{id}:{filesd}
127.0.0.1:6379> mset user:1:name zhangsan user:1:age 2	
OK
127.0.0.1:6379> mget user:1:name user:1:age
1) "zhangsan"
2) "2"
###############################################################################
127.0.0.1:6379> getset k5 redis	#如果不存在值，就返回null
(nil)
127.0.0.1:6379> get k5
"redis"
127.0.0.1:6379> getset k5 mongodb	#如果存在值，就返回原来值，再插入新值
"redis"
127.0.0.1:6379> get k5
"mongodb"
###############################################################################
```

### List

```bash
###############################################################################
127.0.0.1:6379> Lpush list one	#将一个值或多个值放在列表的头部（左）
(integer) 1
127.0.0.1:6379> Lpush list two
(integer) 2
127.0.0.1:6379> Lpush list three	
(integer) 3
127.0.0.1:6379> lrange list 0 -1	#获取list中的值
1) "three"
2) "two"
3) "one"
127.0.0.1:6379> lrange list 0 1	#获取区间中的值
1) "three"
2) "two"
127.0.0.1:6379> rpush list right	#将一个值或多个值放在列表的尾部（右）
(integer) 4
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "two"
3) "one"
4) "right"
###############################################################################
127.0.0.1:6379> lpop list	#移除list中的第一个元素
"three"
127.0.0.1:6379> rpop list	#移除list中的最后一个元素
"right"
127.0.0.1:6379> lrange list 0 -1
1) "two"
2) "one"
###############################################################################
127.0.0.1:6379> lindex list 1	#通过下标获取值
"one"
127.0.0.1:6379> lindex list 0
"two"
127.0.0.1:6379> lindex list 2
(nil)
###############################################################################
127.0.0.1:6379> llen list	#获取list的长度
(integer) 2
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "three"
3) "two"
4) "one"
127.0.0.1:6379> lrem list 1 one	#移除list中指定个数的value
(integer) 1
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "three"
3) "two"
127.0.0.1:6379> lrem list 2 three
(integer) 2
127.0.0.1:6379> lrange list 0 -1
1) "two"
###############################################################################
127.0.0.1:6379> rpush list hello
(integer) 1
127.0.0.1:6379> rpush list hello1
(integer) 2
127.0.0.1:6379> rpush list hello12
(integer) 3
127.0.0.1:6379> rpush list hello13
(integer) 4
127.0.0.1:6379> lrange list 0 -1
1) "hello"
2) "hello1"
3) "hello12"
4) "hello13"
127.0.0.1:6379> ltrim list 1 2	#通过下标截取指定长度，只剩下截取的元素
OK
127.0.0.1:6379> lrange list 0 -1
1) "hello1"
2) "hello12"
###############################################################################
127.0.0.1:6379> rpush list 1
(integer) 1
127.0.0.1:6379> rpush list 2
(integer) 2
127.0.0.1:6379> rpush list 3
(integer) 3
127.0.0.1:6379> lrange list 0 -1
1) "1"
2) "2"
3) "3"
127.0.0.1:6379> rpoplpush list mylist	#移除列表的最后一个元素，将他移动到新的列表中
"3"
127.0.0.1:6379> lrange list 0 -1
1) "1"
2) "2"
127.0.0.1:6379> lrange mylist 0 -1
1) "3"
###############################################################################
127.0.0.1:6379> exists list
(integer) 0
127.0.0.1:6379> lset list 0 item	#如果list不存在，报错
(error) ERR no such key
127.0.0.1:6379> lpush list value1
(integer) 1
127.0.0.1:6379> lrange list 0 0
1) "value1"
127.0.0.1:6379> lset list 0 item	#将list中指定下标的值更新为另外一个值
OK
127.0.0.1:6379> lrange list 0 0
1) "item"
###############################################################################
127.0.0.1:6379> rpush list hello
(integer) 1
127.0.0.1:6379> rpush list word
(integer) 2
127.0.0.1:6379> linsert list before word other	#将某个具体的值插入到某个值的前面或者后面
(integer) 3
127.0.0.1:6379> lrange list 0 -1
1) "hello"
2) "other"
3) "word"
127.0.0.1:6379> linsert list after word new
(integer) 4
127.0.0.1:6379> lrange list 0 -1
1) "hello"
2) "other"
3) "word"
4) "new"

```

### Set

set中的值不能重复

```bash
###############################################################################
127.0.0.1:6379> sadd myset hello	#向set中添加值
(integer) 1
127.0.0.1:6379> sadd myset xiang
(integer) 1
127.0.0.1:6379> sadd myset lovexiang
(integer) 1
127.0.0.1:6379> smembers myset	#查看set中的值
1) "xiang"
2) "lovexiang"
3) "hello"
127.0.0.1:6379> sismember myset hello	#查看set中是否存在某个值
(integer) 1
127.0.0.1:6379> sismember myset word
(integer) 0
###############################################################################
127.0.0.1:6379> scard myset	#查看set长度
(integer) 3
127.0.0.1:6379> srem myset hello	#移除set中的指定值
(integer) 1
127.0.0.1:6379> smembers myset
1) "xiang"
2) "lovexiang"
###############################################################################
127.0.0.1:6379> smembers myset
1) "hello2"
2) "xiang"
3) "hello1"
4) "lovexiang"
5) "hello4"
127.0.0.1:6379> srandmember myset	#随机取出set中的某个值
"lovexiang"
127.0.0.1:6379> srandmember myset
"lovexiang"
127.0.0.1:6379> srandmember myset
"xiang"
127.0.0.1:6379> srandmember myset
"hello1"
127.0.0.1:6379> srandmember myset 2	#随机取出set中指定个数的值
1) "xiang"
2) "hello2"
127.0.0.1:6379> srandmember myset 2
1) "hello4"
2) "hello2"
###############################################################################
127.0.0.1:6379> smembers myset
1) "hello4"
2) "xiang"
3) "hello2"
4) "lovexiang"
5) "hello1"
127.0.0.1:6379> spop myset	#随机移除set中的一个值
"hello4"
127.0.0.1:6379> spop myset
"hello2"
127.0.0.1:6379> smembers myset
1) "xiang"
2) "lovexiang"
3) "hello1"
###############################################################################
127.0.0.1:6379> smembers myset
1) "xiang"
2) "lovexiang"
3) "hello1"
127.0.0.1:6379> smove myset set2 xiang	#将一个set中的指定元素移动到另一个set中
(integer) 1
127.0.0.1:6379> smembers myset
1) "lovexiang"
2) "hello1"
127.0.0.1:6379> smembers set2
1) "xiang"
###############################################################################
127.0.0.1:6379> sadd k1 a
(integer) 1
127.0.0.1:6379> sadd k1 b
(integer) 1
127.0.0.1:6379> sadd k1 c
(integer) 1
127.0.0.1:6379> sadd k2 c
(integer) 1
127.0.0.1:6379> sadd k2 d
(integer) 1
127.0.0.1:6379> sadd k2 e
(integer) 1
127.0.0.1:6379> sdiff k1 k2	#查看两个集合中差集
1) "a"
2) "b"
127.0.0.1:6379> sinter k1 k2	#交集
1) "c"
127.0.0.1:6379> sunion k1 k2	#并集
1) "c"
2) "b"
3) "e"
4) "d"
5) "a"
```

### Hash（哈希）

Map集合，key-<key,value>

```bash
###############################################################################
127.0.0.1:6379> hset myhash k1 v1	#set一个具体的key-value
(integer) 1
127.0.0.1:6379> hget myhash k1	#获取一个字段值
"v1"
127.0.0.1:6379> hmset myhash  k1 v1 k2 v2 k3 v3	#set多个具体的key-value
OK
127.0.0.1:6379> hmget myhash k1 k2	#获取多个字段值
1) "v1"
2) "v2"
127.0.0.1:6379> hgetall myhash	#获取全部字段值
1) "k1"
2) "v1"
3) "k2"
4) "v2"
5) "k3"
6) "v3"
###############################################################################
127.0.0.1:6379> hdel myhash k1	#删除hash指定key字段，对应的value值也就消失了
(integer) 1
127.0.0.1:6379> hgetall myhash
1) "k2"
2) "v2"
3) "k3"
4) "v3"
###############################################################################
127.0.0.1:6379> hgetall myhash
1) "k2"
2) "v2"
3) "k3"
4) "v3"
127.0.0.1:6379> hlen myhash	#获取hash长度
(integer) 2
###############################################################################
(integer) 2
127.0.0.1:6379> hexists myhash k1	#查看某个元素是否存在
(integer) 0
127.0.0.1:6379> hexists myhash k2
(integer) 1
###############################################################################
127.0.0.1:6379> hkeys myhash	#获取hash中的所有key
1) "k2"
2) "k3"
127.0.0.1:6379> hvals myhash	#获取hash中的所有valu
1) "v2"
2) "v3"
###############################################################################
127.0.0.1:6379> hset myhash k4 5
(integer) 1
127.0.0.1:6379> hincrby myhash k4 1	#指定增量
(integer) 6
127.0.0.1:6379> hincrby myhash k4 -1
(integer) 5
127.0.0.1:6379> hsetnx myhash k5 hello	#如果不存在才能设值
(integer) 1
127.0.0.1:6379> hsetnx myhash k5 word	#如果存在不能设值
(integer) 0
```

hash变更的数据

hash更适合对象的存储

### Zset（有序集合）

在set的基础上增加了一个排序功能

```bash
127.0.0.1:6379> zadd myset 1 one	#插入一个值
(integer) 1
127.0.0.1:6379> zadd myset 2 two 3 three	#插入多个值
(integer) 2
127.0.0.1:6379> zrange myset 0 -1
1) "one"
2) "two"
3) "three"
###############################################################################
127.0.0.1:6379> zadd salary 2500 xiaohong	#添加3个用户
(integer) 1
127.0.0.1:6379> zadd salary 5000 zhangsan
(integer) 1
127.0.0.1:6379> zadd salary 500 lisi
(integer) 1
127.0.0.1:6379> zrangebyscore salary -inf +inf	#按照从小到大排序显示所有用户
1) "lisi"
2) "xiaohong"
3) "zhangsan"
127.0.0.1:6379> zrangebyscore salary  -inf +inf  withscores	#按照从小到大显示用户带上数字
1) "lisi"
2) "500"
3) "xiaohong"
4) "2500"
5) "zhangsan"
6) "5000"
127.0.0.1:6379> zrangebyscore salary  -inf 2500  withscores	#按照从小到大排序显示小于2500的用户
1) "lisi"
2) "500"
3) "xiaohong"
4) "2500"
###############################################################################
127.0.0.1:6379> zrange salary 0 -1
1) "lisi"
2) "xiaohong"
3) "zhangsan"
127.0.0.1:6379> zrem salary xiaohong
(integer) 1
127.0.0.1:6379> zrange salary 0 -1
1) "lisi"
2) "zhangsan"
###############################################################################
127.0.0.1:6379> zcard salary	#查看长度
(integer) 2
```

## 三中的特殊数据类型

### geospatial地理位置

> getadd

```bash
#geoadd 添加地理位置
#规则：两极无法添加
#有效的经度-180到180
#有效的维度-85.05112878到85.05112878
#当坐标位置超出上述限定的范围时，该命令会返回一个错误
127.0.0.1:6379> geoadd china:city 116.40 39.90 beijing
(integer) 1
127.0.0.1:6379> geoadd china:city 121.47 31.23 shanghai
(integer) 1
127.0.0.1:6379> geoadd china:city 106.50 29.53 chongqing
(integer) 1
127.0.0.1:6379> geoadd china:city 114.05 22.52 shengzhen
(integer) 1
127.0.0.1:6379> geoadd china:city 120.16 30.24 xian
(integer) 1
```

> geopos

```bash
#获得当前定位
127.0.0.1:6379> geopos china:city beijing
1) 1) "116.39999896287918091"
   2) "39.90000009167092543"
127.0.0.1:6379> geopos china:city beijing chongqing
1) 1) "116.39999896287918091"
   2) "39.90000009167092543"
2) 1) "106.49999767541885376"
   2) "29.52999957900659211"
```

> geodist

指定单位的参数 unit 必须是以下单位的其中一个：

- **m** 表示单位为米。
- **km** 表示单位为千米。
- **mi** 表示单位为英里。
- **ft** 表示单位为英尺。

```bash
127.0.0.1:6379> geodist china:city beijing shanghai
"1067378.7564"
127.0.0.1:6379> geodist china:city beijing chongqing
"1464070.8051"
127.0.0.1:6379> geodist china:city beijing chongqing km
"1464.0708"
```

> georadius

以给定的经纬度为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。

```bash
127.0.0.1:6379> georadius china:city 110 13 3000 km
1) "chongqing"
2) "shengzhen"
3) "xian"
4) "shanghai"
```

> georadiusbymember

可以找出位于指定范围内的元素， 但是 `GEORADIUSBYMEMBER` 的中心点是由给定的位置元素决定的， 而不是像 [GEORADIUS](http://redis.cn/commands/georadius.html) 那样， 使用输入的经度和纬度来决定中心点指定成员的位置被用作查询的中心。

```bash
127.0.0.1:6379> georadiusbymember china:city shanghai 300 km
1) "xian"
2) "shanghai"
```

> geohash 

-该命令将返回11个字符的Geohash字符串

```bash
#将二维的经纬度转换为一维的字符串，字符串越接近两地越近
127.0.0.1:6379> geohash china:city beijing
1) "wx4fbxxfke0"
127.0.0.1:6379> geohash china:city beijing chongqing
1) "wx4fbxxfke0"
2) "wm5xzrybty0"
```

```bash
127.0.0.1:6379> zrange china:city 0 -1
1) "chongqing"
2) "shengzhen"
3) "xian"
4) "shanghai"
5) "beijing"
127.0.0.1:6379> zrem china:city beijing
(integer) 1
127.0.0.1:6379> zrange china:city 0 -1
1) "chongqing"
2) "shengzhen"
3) "xian"
4) "shanghai"
```

geo底层是zset实现的。

### Hyperloglogs

目的是计数的时候就可使用Hyperloglogs

```bash
127.0.0.1:6379> pfadd k 1 2 1 3 4 5
(integer) 1
127.0.0.1:6379> pfcount k
(integer) 5
127.0.0.1:6379> pfadd k2 6 7 8 9 10 11
(integer) 1
127.0.0.1:6379> pfcount k2
(integer) 6
127.0.0.1:6379> pfmerge k3 k k2
OK
127.0.0.1:6379> pfcount k3
(integer) 11
```

如果允许容错可使用此种方式。

### Bitmaps

位操作，只有0，1

```bash
127.0.0.1:6379> setbit sign 0 0
(integer) 0
127.0.0.1:6379> setbit sign 1 0
(integer) 0
127.0.0.1:6379> setbit sign 2 1
(integer) 0
127.0.0.1:6379> setbit sign 3 1
(integer) 0
127.0.0.1:6379> setbit sign 4 0
(integer) 0
127.0.0.1:6379> setbit sign 5 1
(integer) 0
127.0.0.1:6379> setbit sign 6 0
(integer) 0
127.0.0.1:6379> getbit sign 5
(integer) 1
127.0.0.1:6379> getbit sign 2
(integer) 1
```

## 事务

redis单条命令是保证原子性的，但是事务是不保证原子性的 。

Redis事务本质：一组命令的集合，一个事务中的所有命令会被顺序化，在事务执行的过程中，会按照顺序执行

一次性、顺序性、排他性、执行一系列命令

redis事务：

* 开启事务（multi）
* 命令队列(.....)
* 执行事务(exec)

```bash
127.0.0.1:6379> multi	#开启事务
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> exec	#执行事务
1) OK
2) OK
3) OK
4) "v2"
5) OK


127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> discard	#取消事务
OK
127.0.0.1:6379> get k4
nil
```

> 编译型异常（命令错误），事务中的所有命令都不执行

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> getset k3 v3
QUEUED
127.0.0.1:6379> setget k4 v4	#错误的命令
(error) ERR unknown command `setget`, with args beginning with: `k4`, `v4`,
127.0.0.1:6379> set k5 v5
QUEUED
127.0.0.1:6379> exec	#执行事务会报错
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k5	#所有的命令都没有执行
(nil)
127.0.0.1:6379> get k1
(nil)
```

> 运行时异常

```bash
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> incr k1		#语法没问题，但是不可以向字符串加1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> exec
1) (error) ERR value is not an integer or out of range	#虽然命令错了，但是其他依旧正常执行
2) OK
3) OK
127.0.0.1:6379> get k2
"v2"
```

**悲观锁**

* 无论做什么都会加锁

**乐观锁**

* 更新数据的时候会判断一下，在此期间是否有人修改过

```bash
27.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> decrby money 20
QUEUED
127.0.0.1:6379> incrby out 20
QUEUED
127.0.0.1:6379> exec	
1) (integer) 80
2) (integer) 20
```

测试多线程环境，使用watch可以当作redis的乐观锁操作

```bash
127.0.0.1:6379> watch money
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> incrby money 10
QUEUED
127.0.0.1:6379> decrby out 10
QUEUED
127.0.0.1:6379> exec#在此动作之前执行了127.0.0.1:6379> incrby money 1000
									  #(integer) 1080
(nil)

127.0.0.1:6379> unwatch	#当事务执行失败就先解锁
OK
127.0.0.1:6379> watch money	#再获取锁
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> decrby money 10
QUEUED
127.0.0.1:6379> incrby out 10
QUEUED
127.0.0.1:6379> exec
1) (integer) 1070
2) (integer) 30
```

## Jedis

* 添加依赖
* 连接（new Jedis(ip,port)）
* 操作命令
* 断开连接

springboot 整合redis

* 导入依赖
* 配置连接

关于对象的保存必须将对象序列化

# GUI



