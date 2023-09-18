---
layout: post
title: 《Linux高性能服务器编程》 学习笔记：Select, Poll and Epoll
categories: Web
description: 《Linux高性能服务器编程》 学习笔记
keywords: Linux Web Server
excerpt: 记录对《Linux高性能服务器编程》 的学习笔记
---

# 主机字节序与网络字节序
目前PC大多是小端字节序，而网络字节序又是大端字节序，因此需要进行字节序转换，否则发送端接收端字节序不一样的话就会产生问题

Linux提供了如下函数来完成字节序转换：
`htonl`, `htons`(host to network short), `ntohl`和`ntohs`

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