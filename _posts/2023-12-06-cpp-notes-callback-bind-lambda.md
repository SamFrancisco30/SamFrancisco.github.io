---
layout: post
title: C++ 学习笔记： cross-class callbacks, std::bind and Lambda expression
categories: C++
description: C++ 学习笔记： cross-class callbacks, std::bind and Lambda expression
keywords: C++ std::bind Lambda
excerpt: C++ 学习笔记： cross-class callbacks, std::bind and Lambda expression
---

# 使用`std::bind`绑定类的成员函数

考虑以下代码：
```
void Server::newConnection(Socket *serv_sock){
    InetAddress *clnt_addr = new InetAddress();      //会发生内存泄露！没有delete
    Socket *clnt_sock = new Socket(serv_sock->accept(*clnt_addr));       //会发生内存泄露！没有delete
    printf("new client fd %d! IP: %s Port: %d\n", clnt_sock->getFd(), inet_ntoa(clnt_addr->GetAddr().sin_addr), ntohs(clnt_addr->GetAddr().sin_port));
    clnt_sock->setNonBlocking();
    Channel *clntChannel = new Channel(loop, clnt_sock->getFd());
    std::function<void()> cb = std::bind(&Server::handleReadEvent, this, clnt_sock->getFd());
    clntChannel->setCallback(cb);
    clntChannel->enableReading();
}
```
下面的语句使用`std::bind`创建了一个function object
```
std::function<void()> cb = std::bind(&Server::handleReadEvent, this, clnt_sock->getFd());
```
如下是`handleReadEvent`的函数定义：
```
void handleReadEvent(int);
```
可以看到该函数仅有一个int类型参数，但在bind里却多了一个this参数，这是因为当使用`std::bind`来绑定一个类的成员函数时，需要提前指定好调用函数的类的对象，这样在某个对象调用`cb`的时候才能正常调用`Server::handleReadEvent`

值得注意的是此时`cb`这一function object就可以被任意的类的对象调用了，比如在这个例子中就会被一个`Channel`类的对象调用。需要注意要保证这时候绑定的那个`Server`类的对象还存在。

# Lambda表达式

在新的C++标准中`std::bind` 和 `std::function`均被弃用，应由Lambda表达式来替代

Lambda表达式提供了一个类
似匿名函数的特性，而匿名函数则是在需要一个函数，但是又不想费力去命名一个函数的情况下去使用的。

Lambda表达式的基本语法如下：
```
[捕获列表](参数列表) mutable(可选) 异常属性 -> 返回类型 {
// 函数体
}
```

例如上文使用`std::bind`的语句可以用Lambda表达式这么写：
```
std::function<void()> cb = [this, fd = clnt_sock->getFd()]() { this->handleReadEvent(fd); };
```

## 捕获列表的类型
1. 值捕获：在这种情况下被捕获的变量在 Lambda表达式被创建时拷贝，而非调用时才拷贝

2. 引用捕获：比较好理解，类似于引用传参

3. 隐式捕获

*  [] 空捕获列表
*  [name1, name2, . . . ] 捕获一系列变量
*  [&] 引用捕获, 让编译器自行推导引用列表
*  [=] 值捕获, 让编译器自行推导值捕获列表

4. 表达式捕获