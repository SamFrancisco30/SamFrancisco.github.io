---
layout: post
title: 《Linux高性能服务器编程》 学习笔记三：两种高效的事件处理模式： Reactor and Proactor
categories: Web
description: 《Linux高性能服务器编程》 学习笔记
keywords: Linux Reactor Proactor
excerpt: 记录对《Linux高性能服务器编程》 的学习笔记
---

# I/O 模型
* 阻塞I/O： 程序阻塞于读写函数
* I/O复用： 程序阻塞于I/O复用的syscall，I/O本身的读写操作是非阻塞的
* SIGIO： 信号触发读写就绪事件，然后用户执行读写操作，程序无阻塞
* 异步I/O： kernel执行读写操作并触发读写完成事件，程序无阻塞

# Reactor模式
同步I/O模式用于实现Reactor模式，Reactor要求主线程（I/O处理单元）只负责监听fd上是否有事件发生，有则立即将事件通知给工作线程，此外主线程不做任何其他实质性的事情。

Reactor模式含义：I/O 多路复用监听事件，收到事件后，根据事件类型分配（Dispatch）给某个进程 / 线程

Reactor模式的两个核心部分：
* Reactor：负责监听和分发事件
* 处理资源池：负责处理事件

## 实现Reactor的工作流程（以epoll_wait）为例
1. 主线程向epoll事件表中注册socket上的read就绪事件并调用epoll_wait，等待socket有数据可读;
2. socket有数据可读时，epoll_wait通知主线程，主线程将这一可读事件放入请求队列;
3. 某个在请求队列上sleep的工作线程被唤醒，它从socket中读数据，处理客户请求，然后向epoll注册write就绪事件;
4. 主线程调用epoll_wait等待socket可写;
5. socket可写时，epoll_wait通知主线程，主线程将这一可写事件放入请求队列;
6. 某个在请求队列上sleep的工作线程被唤醒，它向socket上写入数据。

