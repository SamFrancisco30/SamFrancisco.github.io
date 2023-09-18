---
layout: post
title: 《Linux高性能服务器编程》 学习笔记二：Select, Poll and Epoll
categories: Web
description: 《Linux高性能服务器编程》 学习笔记
keywords: Linux Web Epoll
excerpt: 记录对《Linux高性能服务器编程》 的学习笔记
---

# select系统调用
基本思想：在一段指定时间内，监听用户感兴趣的文件描述符上的可读、可写和异常等事件

## select API
```
#include <sys/select.h>
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```
* `nfds`:用于指定被监听的文件描述符的总数
* `readfds`: 指向想要检查可读性的文件描述符的集合
* `writefds`: 指向想要检查可写性的文件描述符的集合
* `exceptfds`: 指向想要检查异常的文件描述符的集合

*这三个集合可以通过宏 FD_SET, FD_CLR, FD_ISSET, 和 FD_ZERO来设置，kernel会修改它们来告知哪些文件描述符已经就绪*
* `timeout`: 一个指向timeval结构体的指针，表示超时时间，设置成NULL的话select会一直阻塞直到否个文件描述符就绪

*kernel会修改它来告知select等待了多久*

select返回值：
* 成功时返回就绪的文件描述符总数
* 超时并且没有文件描述符就绪返回0
* 失败时返回-1并设置errno

## 一个例子
```
#include <iostream>
#include <sys/select.h>
#include <sys/time.h>
#include <unistd.h>
#include <cstdio>

int main() {
    fd_set readfds;
    struct timeval timeout;

    // Initialize the file descriptor set.
    FD_ZERO(&readfds);

    // Add stdin (file descriptor 0) to readfds.
    FD_SET(STDIN_FILENO, &readfds);

    // Set timeout to 5 seconds.
    timeout.tv_sec = 5;
    timeout.tv_usec = 0;

    // Wait for an event to occur on any of the file descriptors in the set.
    int retval = select(STDIN_FILENO + 1, &readfds, nullptr, nullptr, &timeout);

    if (retval == -1) {
        perror("select");
    } else if (retval) {
        std::cout << "Data is ready to read." << std::endl;
		std::cout << "Number of fd ready: " << retval << std::endl;
    } else {
        std::cout << "No data within five seconds." << std::endl;
    }

    return 0;
}
```
在这个例子中我们将stdin的文件描述符添加到了readfds，然后设置5秒的超时时间，5秒内如果stdin有数据可读就会打印“Data is ready to read.”

# poll系统调用
基本思想类似select，poll()定义如下：
```
#include <poll.h>
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

* `fds`: 一个`pollfd`结构体的数组。每一个`pollfd`结构体表示一个文件描述符及用户想监听的事件
```
struct pollfd {
    int   fd;         /* File descriptor to poll */
    short events;     /* Events to poll for */
    short revents;    /* Events that occurred, filled by kernel */
};
```
* `nfds`: 表示fds数组的大小

* `timeout`: poll的超时值，单位为毫秒

*timeout设置为-1则poll会一直阻塞直到某个事件发生;设置为0则立即返回*

poll返回值和select一样：
* 成功时返回就绪的文件描述符总数
* 超时并且没有文件描述符就绪返回0
* 失败时返回-1并设置errno

## 一个例子
```
#include <iostream>
#include <poll.h>
#include <cstdio>

int main() {
    struct pollfd fds[1];
    int timeout = 5000;  // 5 seconds in milliseconds

    // Monitor stdin (fd 0) for input
    fds[0].fd = STDIN_FILENO;
    fds[0].events = POLLIN;

    int ret = poll(fds, 1, timeout);

    if (ret > 0) {
        if (fds[0].revents & POLLIN) {
            std::cout << "Data is ready to read." << std::endl;
        }
    } else if (ret == 0) {
        std::cout << "Timeout." << std::endl;
    } else {
        perror("poll");
    }

    return 0;
}
```