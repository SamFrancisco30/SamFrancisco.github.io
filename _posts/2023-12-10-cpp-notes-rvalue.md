---
layout: post
title: C++ 学习笔记： rvalue, std::move, std::forward
categories: C++
description: C++ 学习笔记： rvalue, std::move, std::forward
keywords: C++ rvalue std::move std::forward
excerpt: C++ 学习笔记： rvalue, std::move, std::forward
---

# 左值和右值

## 左值(lvalue)

lvalue是实际占据内存空间（有自己的地址）的对象，是表达式（不一定是赋
值表达式）后依然存在的持久对象

## 右值(rvalue)

与lvalue相反，不为一个占据内存的物体，表达式结束后就不再存在的临时对象，可以分为纯右值和将亡值

* 纯右值：例如10,true，1+2这种临时对象，还有**Lambda表达式**

* 将亡值：即将被销毁、却能够被移动的值

考虑如下代码：
```
std::vector<int> foo() {
    std::vector<int> temp = {1, 2, 3, 4};
    return temp;
}
std::vector<int> v = foo();
```

在这个例子中foo函数返回的`std::vector<int>`对象就是个将亡值，因为其马上就要被销毁，但其内容可以被移动给`v`

*注意：形参永远是左值，即使它的类型是一个右值引用，例如`void f(Widget&& w);`中的`w`*

# 右值引用
语法：`T &&`

## 移动语义(move semantics)
在C++11前要实现资源的移动必须先复制一份析构，造成大量浪费的数据拷贝

以下代码例子展示了通过std::move减少拷贝出现的开销：
```
std::string str = "Hello world.";
std::vector<std::string> v;

// 将使用 push_back(const T&), 即产生拷贝行为
v.push_back(str);
// 输出 "str: Hello world."
std::cout << "str: " << str << std::endl;


// 将使用 push_back(const T&&), 不会出现拷贝行为
// 而整个字符串会被移动到 vector 中，所以这步操作后, str 中的值会变为空
v.push_back(std::move(str));
// 输出 "str: "
std::cout << "str: " << str << std::endl;
```

我们再来分析一下执行这段代码时到底发生了什么：
```
std::string str = "Hello world.";
std::vector<std::string> v;
v.push_back(std::move(str));
```

1. `std::move`：将`str`变成一个右值引用(`std::string&&`)，告知这个物体可以被move了

*注意在这里`std::move`没有move任何东西，实际的move操作还未发生*

2. `v.push_back()`：调用了`std::string`的移动构造函数

3. `std::string`的移动构造函数：将内部的数据指针转移到vector里的新物体并将`str`的设置为空

所以在这里如果没有使用`std::move`，`str`会被当作一个左值，从而`v.push_back(str)`会调用`std::string`的拷贝构造函数而不是移动构造函数

*注意：1. 不要在你希望能移动对象的时候，声明他们为const，否则还是会调用拷贝构造; 2. `std::move`不移动任何东西，也不保证它执行转换的对象可以被移动*

# 完美转发(perfect forwarding)
语法：`std::forward<T>(arg)`

完美转发让我们在传递参数时可以保持原来的参数类型（左引用保持左引用，右引用保持右引用）

## 完美转发的典型用法
最常见的情景是一个模板函数，接收一个通用引用(`T&&`)形参，并将它传递给另外的函数：
```
void process(const Widget& lvalArg);        //处理左值
void process(Widget&& rvalArg);             //处理右值

template<typename T>                        //用以转发param到process的模板
void logAndProcess(T&& param)
{
    auto now =                              //获取现在时间
        std::chrono::system_clock::now();
    
    makeLogEntry("Calling 'process'", now);
    process(std::forward<T>(param));
}
```
本来我们设想的是既然`process`做了对左值和右值的重载，那么应该分别对左值和右值调用各种版本的函数，但是由于`param`作为一个形参是一个左值，所以如果没有`std::forward`的话每次都只会调用左值重载版本的`process`

*注意：`std::forward`是一个有条件的转换：只有在它的实参是用右值初始化时，才会转换为一个右值。*