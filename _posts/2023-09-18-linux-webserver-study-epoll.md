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

# Socket地址
用于表示socket地址的结构体是`sockaddr`，其定义大致如下：
```
struct sockaddr {
	sa_family_t	sa_family;	/* address family, AF_xxx	*/
	union {
		char sa_data_min[14];		/* Minimum 14 bytes of protocol address	*/
		DECLARE_FLEX_ARRAY(char, sa_data);
	};
};
```

`sa_family`表示地址族类型(`sa_family_t`)，可以是`AF_INET`(IPv4)也可以是`AF_INET6`(IPv6)

*除了`sockaddr`外，还有一个通用地址结构体`sockaddr_storage`*

`sockaddr`这个通用的结构体是不好用的，因为如果要设置IP地址和端口号还得对char数组进行位操作，不方便，于是又有了对于各个协议族的专门结构体

对于IPv4，有如下socket地址结构体：
```
struct sockaddr_in {
  __kernel_sa_family_t	sin_family;	/* Address family		*/
  __be16		sin_port;	/* Port number			*/
  struct in_addr	sin_addr;	/* Internet address		*/

  /* Pad to size of `struct sockaddr'. */
  unsigned char		__pad[__SOCK_SIZE__ - sizeof(short int) -
			sizeof(unsigned short int) - sizeof(struct in_addr)];
};
```

*`be`表示大端，后续的unsigned char部分是为了让整个结构体的大小是8字节的倍数*

*在实际使用的时候，需要将专用的socket地址结构体强制转化为`sockaddr`*