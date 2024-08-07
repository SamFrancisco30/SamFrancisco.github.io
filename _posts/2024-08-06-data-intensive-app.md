---
layout: post
title: 《Designing Data-Intensive Application》读书笔记
categories: Distributed system
description: 《Designing Data-Intensive Application》读书笔记
keywords: Data-Intensive
excerpt: Data-Intensive Application
---

# 数据系统的基石
## 可靠性，可扩展性，可维护性
### 可靠性
fault和failure不同，fault意思是系统的一部分状态偏离其标准，而failure则是系统作为一个整体停止向用户提供服务。fault的概率不可能降到零，因此需要一定的容错机制来防止fault导致failure

有时在一些容错系统中需要通过故意触发来提高故障率，来确保容错机制不断运行并接受考验，从而提高故障自然发生时系统能正确处理的信心

分布式系统的故障大致可分为：硬件故障、软件错误和人为错误（如配置错误）

### 可扩展性
这一部分需要关注负载参数和性能指标

负载参数可以是是每秒向Web服务器发出的请求、数据库中的读写比率、聊天室中同时活跃的用户数量、缓存命中率等

对于在线系统，通常考虑的性能指标是response time，即客户端发送请求到接收响应之间的时间。需要注意的是response time不同于latency（包含了latency）

Tail latencies（high latencies that clients see fairly infrequently）非常重要，这是因为请求响应最慢的客户往往也是数据最多的客户

排队延迟（queueing delay）占了Tail latencies的很大一部分，只要有少量缓慢的请求就能阻碍后续请求的处理

应对负载需要将vertical scaling（换更好的机器）和horizontal scaling（将负载分布到多台小机器上）灵活地结合

### 无状态服务 (Stateless Service)
无状态服务指的是每个请求都是独立的，服务器不保留任何关于客户端的会话信息。每个请求都包含了服务器处理所需的所有信息。

例如：RESTful API

# 数据模型与查询语言
## NoSQL
NoSQL数据库是一类非关系型数据库，专门设计用于处理大量的数据和高并发的应用程序需求

特点：不使用固定的表格结构，而是采用灵活的数据模型，如键-值对、文档、列族、图等。

### NoSQL数据库的主要类型
1. 键-值存储（Key-Value Store）

    特点：通过键-值对来存储数据，适用于简单的数据存取。
    示例：Redis, Riak

2. 文档存储（Document Store）

    特点：数据以文档的形式存储，通常使用JSON、BSON等格式，每个文档可以有不同的结构。
    示例：MongoDB, CouchDB

3. 列族存储（Column Family Store）

    特点：数据以列簇的形式存储，适用于存储大量的半结构化数据。
    示例：Apache Cassandra, HBase

4. 图数据库（Graph Database）

    特点：使用节点、边和属性来存储数据，特别适合处理复杂的关系和连接。
    示例：Neo4j, ArangoDB

### 优缺点
优点：

* 扩展性强：NoSQL数据库通常支持水平扩展，即通过增加更多的服务器节点来处理更多的数据和请求。

* 高性能：针对特定的查询和存储模式进行了优化，可以提供高并发的读写性能。

* 灵活的数据模型：支持灵活的、动态的模式，无需预定义表结构，适用于快速变化的应用需求。

* 大数据处理：适合处理海量数据，能够存储和处理比关系型数据库更多的数据。

* 高可用性和容错性：许多NoSQL数据库内置了自动分片、复制和容错机制，能够在硬件故障时提供高可用性。

缺点：可能导致数据不一致、查询复杂、事务处理能力有限