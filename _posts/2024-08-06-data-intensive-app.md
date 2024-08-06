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

### 可扩展性
这一部分需要关注负载参数和性能指标

负载参数可以是是每秒向Web服务器发出的请求、数据库中的读写比率、聊天室中同时活跃的用户数量、缓存命中率等

对于在线系统，通常考虑的性能指标是response time，即客户端发送请求到接收响应之间的时间。需要注意的是response time不同于latency（包含了latency）

Tail latencies（high latencies that clients see fairly infrequently）非常重要，这是因为请求响应最慢的客户往往也是数据最多的客户

排队延迟（queueing delay）占了Tail latencies的很大一部分，只要有少量缓慢的请求就能阻碍后续请求的处理

应对负载需要将vertical scaling（换更好的机器）和horizontal scaling（将负载分布到多台小机器上）灵活地结合