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

## 实现Reactor的工作流程（以epoll_wait）为例
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

# epoll系统调用
与select和poll的不同在于，epoll需要使用多个函数来完成操作

## 内核事件表
储存用户关心的fd及其上的事件，这个表本质上就是一个文件，因此表本身需要一个额外的fd来标识。

### 创建内核事件表：`epoll_create` 和 `epoll_create1`

```
int epoll_create(int size);
```
* `size`: 给内核提示这个表有多大，现在这个参数基本没用了

```
int epoll_create1(int flags);
```

* `flags`: 用于控制创建出的表：设置为0时和`epoll_create`相同;设置为`EPOLL_CLOEXEC`时会设置这个fd的close-on-exec (FD_CLOEXEC) flag，表示这个epoll的fd会在新的程序通过exec系syscall（execv, execp, execle等）执行时自动关闭

*设置EPOLL_CLOEXEC的好处是可以避免出现fd资源被浪费的情况以及可能的安全问题，例如，当一个程序调用了epoll_create，然后fork了一个新进程，这时如果没有设置EPOLL_CLOEXEC那么父进程和子进程都能访问到这个epoll instance，即使子进程用了exec来替换了自己的image（这时候按道理子进程不应该可以访问到这个fd了）*

### 操作内核事件表：`epoll_ctl`
`epoll_ctl`函数定义如下：
```
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

* `epfd`: 要操作的这个内核事件表的fd

* `op`: 操作类型，有以下几种：

    1. `EPOLL_CTL_ADD`：注册fd上的事件
    2. `EPOLL_CTL_MOD`：修改fd上注册了的事件
    3. `EPOLL_CTL_DEL`：删除fd上注册了的事件

* `fd`: 要操作的fd

* `event`: 指向`epoll_event`结构体的一个指针。表示想要在fd上监听的事件

```
struct epoll_event {
    uint32_t     events;      /* Epoll events */
    epoll_data_t data;        /* User data variable */
};

typedef union epoll_data {
    void        *ptr;
    int          fd;
    uint32_t     u32;
    uint64_t     u64;
} epoll_data_t;
```

`epoll_ctl`成功时返回0;失败时返回-1并设置errno

#### 几种events
events可以是以下几个宏的集合：
* `EPOLLIN` ：表示对应的文件描述符可以读（包括对端SOCKET正常关闭）；
* `EPOLLOUT`：表示对应的文件描述符可以写；
* `EPOLLPRI`：表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）；
* `EPOLLERR`：表示对应的文件描述符发生错误；
* `EPOLLHUP`：表示对应的文件描述符被挂断；
* `EPOLLET`： 将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。
* `EPOLLONESHOT`：对于设置了该flag的fd，当一次事件被触发后，这一fd不会再被触发，除非使用`epoll_ctl`重置这一fd上注册了这一flag的事件

*若没有`EPOLLONESHOT`这一flag，会出现一种情况，例如epoll_wait检测到某个fd可读，通知应用程序用一个线程去处理这一fd的数据，在处理过程中这一fd又可能再次可读，于是另一个线程被唤醒去处理同一个fd，造成竞争现象*

*同样要注意，注册了`EPOLLONESHOT`的事件在被处理完毕后需要重置`EPOLLONESHOT`事件，以确保该事件下一次可以被触发*

## 主要函数：`epoll_wait`
epoll_wait()会在一段超时时间内等待一组fd上的事件，其函数定义如下：
```
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

* `epfd`: 要操作的这个内核事件表的fd

* `events`: 指向一个`epoll_event`结构体的数组。由kernel来填充，当`epoll_wait`检测到事件时就将就绪的事件从内核事件表（epfd）中复制到这个数组中，所以这个参数其实只是用于输出结果（就绪的事件）的

* `maxevents`: 指定最多监听多少事件，也就是`events`数组的大小

* `timeout`: 超时值

`epoll_wait`成功时返回就绪的fd的个数;失败时返回-1并设置errno

## LT和ET
LT和ET(Level-Triggered vs Edge-Triggered)是两种事件通知机制，这两种模式决定了`epoll_wait`在什么时候通知fd就绪

### Level-Triggered (LT)
LT模式是epoll的默认工作模式，这种模式下epoll相当于一个高效率的poll，`epoll_wait`会在注册的条件满足时持续通知应用程序这一fd就绪了，此时应用程序可以不立即处理该事件，这样一来，下一次调用`epoll_wait`时该事件还会被再次通知，直到这一事件被处理，因此这可能会造成busy loop和高CPU使用，效率较低

### Edge-Triggered (ET)
通过注册一个EPOLLET事件来使用ET模式来操作某个fd，例如：
```
struct epoll_event event;
event.events = EPOLLIN | EPOLLET;  // Edge-Triggered for read events
event.data.fd = some_fd;
epoll_ctl(epoll_fd, EPOLL_CTL_ADD, some_fd, &event);
```

ET模式是epoll的高效工作模式，ET模式下，`epoll_wait`仅仅在检测到fd上的事件发生（注册的条件由false变为true）的时候会通知应用程序，然后应用程序必须尽快处理该事件，因为之后这一事件就不会再次被通知了

## 一个例子
```
#include <iostream>
#include <sys/epoll.h>
#include <unistd.h>
#include <cstring>

int main() {
    // Create an epoll instance
    int epoll_fd = epoll_create1(0);
    if (epoll_fd == -1) {
        std::cerr << "Failed to create epoll instance: " << strerror(errno) << std::endl;
        return 1;
    }

    // Register stdin (file descriptor 0) with the epoll instance
    epoll_event event;
    event.events = EPOLLIN;  // Monitor for data to read
    event.data.fd = STDIN_FILENO;  // Monitoring stdin

    if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, STDIN_FILENO, &event) == -1) {
        std::cerr << "Failed to add file descriptor to epoll: " << strerror(errno) << std::endl;
        return 1;
    }

    // Create an array to hold returned events
    epoll_event events[10];

    // Wait for events with a 5-second timeout
    int num_fds = epoll_wait(epoll_fd, events, 10, 5000);

    if (num_fds == -1) {
        std::cerr << "epoll_wait failed: " << strerror(errno) << std::endl;
        return 1;
    } else if (num_fds == 0) {
        std::cout << "No data within 5 seconds." << std::endl;
    } else {
        for (int i = 0; i < num_fds; ++i) {
            if (events[i].data.fd == STDIN_FILENO) {
                std::cout << "Data is ready to read from stdin." << std::endl;
            }
        }
    }

    // Close the epoll instance
    close(epoll_fd);

    return 0;
}
```