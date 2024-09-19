---
layout: post
title: Redis学习笔记
categories: Redis
description: Redis学习笔记
keywords: Redis
excerpt: Redis Notebook
---

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
## String
使用场景: 适合存储简单的字符串、整数、浮点数等常见的数据，如缓存对象、计数器、限流器等。

### Redis的string类型和常规字符串的区别
1. 可以存储多种数据形式，包括普通字符串、整数和浮点数
2. 是二进制安全的，可以存储任何二进制数据，比如图片、音频文件等，最长可以达到 **512MB**
3. 可以使用 INCR、DECR、INCRBYFLOAT 这类命令直接对存储的整数或浮点数进行加减运算

