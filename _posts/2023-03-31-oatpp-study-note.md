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

  /* Create Router for HTTP requests routing */
  auto router = oatpp::web::server::HttpRouter::createShared();

  /* Create HTTP connection handler with router */
  auto connectionHandler = oatpp::web::server::HttpConnectionHandler::createShared(router);

  /* Create TCP connection provider */
  auto connectionProvider = oatpp::network::tcp::server::ConnectionProvider::createShared({"localhost", 8000, oatpp::network::Address::IP_4});

  /* Create server which takes provided TCP connections and passes them to HTTP connection handler */
  oatpp::network::Server server(connectionProvider, connectionHandler);

  /* Print info about server port */
  OATPP_LOGI("MyApp", "Server running on port %s", connectionProvider->getProperty("port").getData());

  /* Run server */
  server.run();
}

int main() {

  /* Init oatpp Environment */
  oatpp::base::Environment::init();

  /* Run App */
  run();

  /* Destroy oatpp Environment */
  oatpp::base::Environment::destroy();

  return 0;

}
```

有以下几个部分需要关注：

`HttpRouter`类：Router of HTTP requests. It maps URLs to endpoint handlers. 

`endpoint`指的是一个可以用来访问某种资源或功能的具体的URL，比如在`http://example.com/products/1234`中，endpoint就是`/products/1234`，又比如`/login`可能就是一个用于通过接受用户名和密码来验证用户的endpoint

所以`HttpRouter`类实际上就是用来定义一些规则，把不同的HTTP requests交给不同的endpoint handlers来处理

thread per each connection.

`HttpConnectionHandler`类为每一个HTTP连接创建一个线程来处理

   `ConnectionProvider`类：Provider of TCP connections. It binds to a specified port.

`ConnectionProvider`类为web服务器提供TCP连接，它会监听一个具体的端口看是否有用户端传来的连接请求

   `Server`类：Server runs a loop which takes connections from `ConnectionProvider` and passes them to `ConnectionHandler`

`Server`类持续不断地将`ConnectionProvider`提供的连接传给`ConnectionHandler`

## 添加自定义的Handler
以上代码运行时只会显示404，这是因为还没有添加任何handler

要想添加自定义的handler，我们需要自己写handler类来继承`HttpConnectionHandler`类，并重写其中的handle函数

```
/** 
 * Custom Request Handler
 */
class Handler : public oatpp::web::server::HttpRequestHandler {
public:

  /**
   * Handle incoming request and return outgoing response.
   */
  std::shared_ptr<OutgoingResponse> handle(const std::shared_ptr<IncomingRequest>& request) override {
    return ResponseFactory::createResponse(Status::CODE_200, "Hello World!");
  }

};
```
参数是一个`IncomingRequest`的智能指针的引用，返回值是`OutgoingResponse`的智能指针

有了handler还不够，还需要一个router来将requests交给我们定义的handler，我们之前的代码已经创建了router了，现在我们通过`route`方法将URL给router到handler

```
/* Route GET - "/hello" requests to Handler */
router->route("GET", "/hello", std::make_shared<Handler>());
```

`route`方法定义如下：
```
void route(const oatpp::String& method, const oatpp::String& pathPattern, const RouterEndpoint& endpoint)
```

`method`代表http method，比如`GET` `POST`等。比如上述例子中的router仅匹配传入的 `GET` 请求

`pathPattern`代表URL的path pattern，意思是表示特定 URL 或一组相关 URL 的字符串，可以包含固定部分和变量部分的组合。URL path pattern的固定部分是常量且不会改变，而变量部分可以匹配一系列值，并使用特殊语法指定。例如`/products/{id}`就是一个URL pattern

最后一个参数`endpoint`就代表要route到的handler了

## 通过JSON对象来回复
前文中，我们添加了一个handler进行HTTP request的回复，但我们回复的内容仅仅只有一个status和一个字符串。

此外，我们还可以回复JSON对象，这一对象首先要进行序列化和反序列化（Serialize/Deserialize，需要我们用到两个类，
`Data Transfer Object (DTO)`和`ObjectMapper`

### DTO
以下是定义DTO object的示例：
```
/* Begin DTO code-generation */
#include OATPP_CODEGEN_BEGIN(DTO)

/**
 * Message Data-Transfer-Object
 */
class MessageDto : public oatpp::DTO {

  DTO_INIT(MessageDto, DTO /* Extends */)

  DTO_FIELD(Int32, statusCode);   // Status code field
  DTO_FIELD(String, message);     // Message field

};

/* End DTO code-generation */
#include OATPP_CODEGEN_END(DTO)
```

可以看到，我们需要自定义自己的DTO类，继承`oatpp:DTO`类，一定要有个`DTO_INIT`语句，随后是这个DTO object包含的field，在这里就是一个`Int32`类型的statusCode和字符串类型的message

关于其他类型的`DTO_FIELD`定义，请参考：https://oatpp.io/docs/components/dto/#field-name-qualifier

*注意，DTO object的定义部分一定要包括在两个include中*

### ObjectMapper
首先我们需要在自定义的`Handler`类中添加一个`ObjectMapper`类的对象作为私有成员变量：
```
class Handler : public oatpp::web::server::HttpRequestHandler {
private:
  std::shared_ptr<oatpp::data::mapping::ObjectMapper> m_objectMapper;
  // ...
};
```
当然还有使用object mapper为参数的构造函数：
```
Handler(const std::shared_ptr<oatpp::data::mapping::ObjectMapper>& objectMapper)
    : m_objectMapper(objectMapper)
  {}
```

然后就可以更改`Handler`类中的`handle`方法，使用另一个重载版本的`ResponseFactory::createResponse`方法来产生回复了

```
static std::shared_ptr<Response> createResponse(const Status& status,
                                                const oatpp::Void& dto,
                                                const std::shared_ptr<data::mapping::ObjectMapper>& objectMapper)
```
修改后的`handle`方法：
```
std::shared_ptr<OutgoingResponse> handle(const std::shared_ptr<IncomingRequest>& request) override {
    auto message = MessageDto::createShared();
    message->statusCode = 1024;
    message->message = "Hello DTO!";
    return ResponseFactory::createResponse(Status::CODE_200, message, m_objectMapper);
  }
```
先使用自定义的`DTO object`类`MessageDto`的`createShared`方法创建这个object的智能指针，然后设置它的两个fields，然后再调用`createResponse`方法

注意，我们现在只更改了自定义的`Handler`类，让类中的`handle`方法可以回复JSON object，此外我们还需要改外面的`run`函数，因为之前调用`route`时：
```
/* Route GET - "/hello" requests to Handler */
router->route("GET", "/hello", std::make_shared<Handler>());
```
Handler类还没有`m_objectMapper`这一成员变量，所以现在我们用新的构造函数来初始化我们的Handler：
```
void run() {

  /* Create json object mapper */
  auto objectMapper = oatpp::parser::json::mapping::ObjectMapper::createShared();

  /* Create Router for HTTP requests routing */
  auto router = oatpp::web::server::HttpRouter::createShared();

  /* Route GET - "/hello" requests to Handler */
  router->route("GET", "/hello", std::make_shared<Handler>(objectMapper /* json object mapper */ ));

  // ...
}
```