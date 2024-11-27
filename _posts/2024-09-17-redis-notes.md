---
layout: post
title: Redis学习笔记
categories: Redis
description: Redis学习笔记
keywords: Redis
excerpt: Redis Notebook
---
# 为什么Redis这么快
1. 基于内存实现
2. 基于不同业务场景的高效数据结构
3. Redis 的网络 IO 以及键值对指令读写是由单个线程来执行的，避免了不必要的context switch和竞选
4. 基于I/O多路复用模型，非阻塞的I/O模型

# 基本命令
* SET
* GET
* DELETE
* TTL：查看键的过期时间
* EXPIRE [key] [second]：设置键的过期时间
* SETEX [key] [second] [value] ：设置带过期时间的键值对
* KEYS [pattern]：查看有哪些键

    e.g. KEYS * 查看所有键

# 五大数据类型
String（字符串）、Hash（哈希）、List（列表）、Set（集合） 和 Sorted Set（有序集合）
# String
使用场景: 适合存储简单的字符串、整数、浮点数等常见的数据，如缓存对象、计数器、限流器等。

## Redis的string类型和常规字符串的区别
1. 可以存储多种数据形式，包括普通字符串、整数和浮点数
2. 是二进制安全的，可以存储任何二进制数据，比如图片、音频文件等，最长可以达到 **512MB**
3. 可以使用 INCR、DECR、INCRBYFLOAT 这类命令直接对存储的整数或浮点数进行加减运算

## 批量设置多个key-value对
```
MSET name "Alice" age "30" city "New York"
```

*注意：MSET 是一种原子操作，这意味着所有的 key-value 对要么同时设置成功，要么同时失败*

# List
## 如何在 Redis 中创建 List？ 
```
# 在左端插入元素
LPUSH mylist "element1"
LPUSH mylist "element2"

# 在右端插入元素
RPUSH mylist "element3"
RPUSH mylist "element4"

# 查看 List 中的所有元素
LRANGE mylist 0 -1
```

## 如何对 Redis List 进行切片操作: LRANGE
```
LRANGE key start stop
```
* key：要操作的 List 的名称。
* start：起始索引，0 表示第一个元素，1 表示第二个元素，依此类推。可以使用负数，-1 表示最后一个元素，-2 表示倒数第二个元素。
* stop：结束索引，包含在结果内。如果 stop 超过 List 的长度，则 Redis 会返回直到 List 末尾的所有元素

e.g.
```
# 获取 List 的最后 3 个元素
LRANGE mylist -3 -1
```

## Redis List 的底层数据结构是什么？ 
早期版本使用 **linkedlist**（双端列表）和 **ziplist**（压缩列表）作为 List 的底层实现，到 Redis 3.2 引入了由 linkedlist + ziplist 组成的 **quicklist**，再到 7.0 版本的时候使用 listpack 取代 ziplist。

首先以ziplist进行存储，在不满足ziplist的存储要求后转换为linkedlist列表
### ziplist
内存紧凑的数据结构，占用一块连续的内存空间，提升内存使用率，**适用于存储元素较少且元素较小的情况**
```
非空 ziplist 示例图

area        |<---- ziplist header ---->|<----------- entries ------------->|<-end->|

size          4 bytes  4 bytes  2 bytes    ?        ?        ?        ?     1 byte
            +---------+--------+-------+--------+--------+--------+--------+-------+
component   | zlbytes | zltail | zllen | entry1 | entry2 |  ...   | entryN | zlend |
            +---------+--------+-------+--------+--------+--------+--------+-------+
```

## 如何使用 Redis List 实现消息队列？ 
生产者用LPUSH将消息插入队列，消费者用RPOP消费

### 消息的可靠性传输问题
使用RPOPLPUSH 指令，当List读取消息的时候，会同步的把该消息复制到另外一个List以作备份
```
# 生产消息 msg1 msg2
> LPUSH list_queue msg1 msg2  
(integer) 2
# 消费消息并同步到备份
> RPOPLPUSH list_queue list_queue_bak
"msg1"
# 当发生故障的时候去消费备份的数据，可以消费到
> RPOP list_queue_bak
"msg1"
```

### 消费及时性问题
使用 BRPOP 阻塞读取。当队列为空时，BLPOP 会阻塞消费者，直到有新的消息被插入到队列中，这样消费者无需主动轮询，能够较为及时地消费消息。

### 消息的重复消费问题
1. List为每一条消息生成一个 Glocal ID，重复的Glocal ID 不进行重复消费
2. 
