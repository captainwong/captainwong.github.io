---
layout: post
title:  "Mutable Lambda与带有状态的（Stateful）Function Object"
subtitle: "天干物燥，小心火烛"
date:   "2017-07-19" 
author: "cj"
tags:
    c++
    c++11
    lambda
    function object
---

众所周知lanmbda值捕获有2种方式：by reference, by value，前者可以对原对象进行修改，后者不能。还有一种方式，声明lambda为mutable，以by value方式捕获对象，但在这个lambda定义的函数对象内，有权利涂写传入的值。例如：

```cpp
int id = 0;
auto f = [id]() mutable { std::cout << "id: " << id << std::endl; ++id; };
id = 42;
f();
f();
f();
std::cout << id << std::endl;
f();
std::cout << id << std::endl;
```

会产生以下输出：

```bash
id: 0
id: 1
id: 2
42
id: 3
42
```

是不是很奇怪 :P
其实是`mutable lambda`函数对象定义时即拷贝了一份id，修改的不是外部id，而是本地拷贝。
拷贝发生在lambda函数对象被定义的时刻，如果将`auto f=...;`一行与下一行`id=42;`交换，则lambda内部拷贝的id值为42。
可以把上述lambda的行为视同下面这个`function object`。

```cpp
class f {
private:
    int id; // copy of outside id
public:
    void operator()() {
        std::cout << "id: " << id << std::endl; ++id;
    }
};
```

>由于mutable的缘故，operator()被定义为一个non-const成员函数，意味着对id的涂写是可行的。所以，有了mutable，lambda变得stateful，即使state是以by value方式传递的。如果没有指明mutable（一般都不用mutable）operator()就成为一个const成员函数，那么对于对象就只能读取。

注：大部分内容摘抄自[C++标准库（第二版）](https://item.jd.com/11706352.html)
