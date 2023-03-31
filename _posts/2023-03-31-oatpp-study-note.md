---
layout: post
title: Oatpp web框架学习笔记
categories: Oatpp
description: Oatpp web框架学习笔记
keywords: Oatpp, Web Framework
excerpt: 记录Oatpp，一个轻量级C++网络框架
---

## 项目布局
```
|- CMakeLists.txt                        // projects CMakeLists.txt
|- src/
|    |
|    |- dto/                             // Folder containing DTOs definitions
|    |    |
|    |    |- DTOs.hpp                    // DTOs are declared here
|    |     
|    |- controller/                      // Folder containing API Controllers where all endpoints are declared
|    |    |
|    |    |- MyController.hpp            // Sample - MyController is declared here
|    |     
|    |- AppComponent.hpp                 // Application Components configuration 
|    |- App.cpp                          // main() is here
|
|- test/                                 // test folder
     |
     |- app/
     |    |
     |    |- MyApiTestClient.hpp         // Api client for test API calls is declared here
     |    |- TestComponent.hpp           // Test application components configuration
     |                                   
     |- MyControllerTest.cpp             // MyController test logic is here
     |- MyControllerTest.hpp             // MyController test header
     |- Tests.cpp                        // tests main() is here
```

## 最简单的web服务器例子
```
void run() {

  /* Register Components in scope of run() method */
  AppComponent components;

  /* Get router component */
  OATPP_COMPONENT(std::shared_ptr<oatpp::web::server::HttpRouter>, router);

  /* Create MyController and add all of its endpoints to router */
  router->addController(std::make_shared<MyController>());

  /* Get connection handler component */
  OATPP_COMPONENT(std::shared_ptr<oatpp::network::ConnectionHandler>, connectionHandler);

  /* Get connection provider component */
  OATPP_COMPONENT(std::shared_ptr<oatpp::network::ServerConnectionProvider>, connectionProvider);

  /* Create server which takes provided TCP connections and passes them to HTTP connection handler */
  oatpp::network::Server server(connectionProvider, connectionHandler);

  /* Print info about server port */
  OATPP_LOGI("MyApp", "Server running on port %s", connectionProvider->getProperty("port").getData());

  /* Run server */
  server.run();
  
}

/**
 *  main
 */
int main(int argc, const char * argv[]) {

  oatpp::base::Environment::init();

  run();
  
  oatpp::base::Environment::destroy();
  
  return 0;
}
```

有以下几个部分需要关注：

1. `HttpRouter`类：Router of HTTP requests. It maps URLs to endpoint handlers. 

`endpoint`指的是一个可以用来访问某种资源或功能的具体的URL，比如在`http://example.com/products/1234`中，endpoint就是`/products/1234`，又比如`/login`可能就是一个用于通过接受用户名和密码来验证用户的endpoint

所以`HttpRouter`类实际上就是用来定义一些规则，把不同的HTTP requests交给不同的endpoint handlers来处理

2. `HttpConnectionHandler`类：It handles incoming connections in a multi threaded manner, creating one thread per each connection.

`HttpConnectionHandler`类为每一个HTTP连接创建一个线程来处理

3. `ConnectionProvider`类：Provider of TCP connections. It binds to a specified port.

`ConnectionProvider`类为web服务器提供TCP连接，它会监听一个具体的端口看是否有用户端传来的连接请求

4. `Server`类：Server runs a loop which takes connections from `ConnectionProvider` and passes them to `ConnectionHandler`

`Server`类持续不断地将`ConnectionProvider`提供的连接传给`ConnectionHandler`

## 添加自定义的Handler
以上代码