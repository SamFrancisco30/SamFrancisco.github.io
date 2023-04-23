---
layout: post
title: 《Linux高性能服务器编程》 学习笔记一：Socket
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