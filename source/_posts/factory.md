---
title: 深入浅出《设计模式》之工厂模式（C++）
date: 2019-03-10 12:41:06
tags: 
    - 设计模式
    - 工厂模式
    - C++
categories: 设计模式
---

# 前言

## 模式介绍

在之前简单工厂模式中，我们介绍了简单工厂模式的缺陷是违背了开放-封闭原则。如果在面馆中添加了烤海参，那将会修改waiter工厂类。违背了类内封闭原则。

还以面馆为例，现在两种面，用一个服务员来卖就可以，如果这个服务员不干了，后面卖面的厨师需要兼职顶替服务员，但是厨师又不能离开灶台，就将模式改成了窗口排队式，一队海参炒面，另一队辣根汤面。每个窗口分别有一个厨师放饭，一个会做海参炒面，另一个会做辣根汤面。老板觉得这种模式挺好，万一来了一个会做烤海参的，就只需要开一个烤海参的窗口就好了，不需要重新要服务员学习，因为烤海参的就会卖。这就变成了工厂模式。

## UML类图

这里涉及到2种类①我，客户端，排队买饭。②做饭厨师类，工厂类，为我生成饭。③菜品类，生成菜品类。具体关系如下UML类图：

![](https://s2.ax1x.com/2019/03/10/A9wFLd.png)



## 代码实例

下面是noodle类，是为工厂类使用的，继承他就可以扩展noodle类别：

``` c++
#ifndef NOODLE_H
#define NOODLE_H

class noodle
{
public:
    noodle() {}
    ~noodle() {}

   virtual void eating() = 0;
};

#endif // NOODLE_H
```

下面是海参炒面类，继承了noodle，实现eating方法，吃海参炒面：
``` c++
#ifndef HAISHENNOODLE_H
#define HAISHENNOODLE_H

#include "noodle.h"

class haishennoodle : public noodle
{
public:
    haishennoodle();
    ~haishennoodle();

    virtual void eating();
};

#endif // HAISHENNOODLE_H
```
``` c++
#include <iostream>
#include "haishennoodle.h"

haishennoodle::haishennoodle()
{

}

haishennoodle::~haishennoodle()
{

}

void haishennoodle::eating()
{
    std::cout << "我是海参炒面，里面没有海参哦！！吃的时候注意!" << std::endl;
}
```
下面是辣根汤面，继承了noodle，实现eating方法，吃辣根汤面：
``` c++
#ifndef LAGENNOODLE_H
#define LAGENNOODLE_H

#include "noodle.h"

class lagennoodle : public noodle
{
public:
    lagennoodle();
    ~lagennoodle();

    virtual void eating();
};

#endif // LAGENNOODLE_H
```
``` c++
#include <iostream>
#include "lagennoodle.h"

lagennoodle::lagennoodle()
{

}

lagennoodle::~lagennoodle()
{

}

void lagennoodle::eating()
{
    std::cout << "我是辣根汤面，吃完呛的哼啊！！！" << std::endl;
}
```
下面是waiter工厂的基类。所有工厂都继承这个类：
``` c++
#ifndef WAITER_H
#define WAITER_H

class noodle;
class waiter
{
public:
    waiter() {}
    ~waiter() {}

    virtual noodle *createnoodle() = 0;
};

#endif // WAITER_H
```
下面是海参厨师（工厂1），海参厨师只管做海参炒面，重写了createnoodle方法：
``` c++
#ifndef HAISHEN_H
#define HAISHEN_H

#include "waiter.h"

class noodle;
class haishen : public waiter
{
public:
    haishen();
    ~haishen();

    virtual noodle *createnoodle();
};

#endif // HAISHEN_H
```
``` c++
#include <iostream>
#include "haishen.h"
#include "haishennoodle.h"

haishen::haishen()
{

}

haishen::~haishen()
{

}

noodle *haishen::createnoodle()
{
    std::cout << "面是我炒得，我的名字叫海参！！！" << std::endl;
    return new haishennoodle();
}
```
下面是辣根厨师（工厂1），辣根厨师只管做辣根汤面，重写了createnoodle方法：
``` c++
#ifndef LAGEN_H
#define LAGEN_H

#include "waiter.h"

class lagen : public waiter
{
public:
    lagen();
    ~lagen();

    virtual noodle *createnoodle();
};

#endif // LAGEN_H
```
``` c++
#include <iostream>
#include "lagen.h"
#include "lagennoodle.h"

lagen::lagen()
{

}

lagen::~lagen()
{

}

noodle *lagen::createnoodle()
{
    std::cout << "吃辣根汤面，你不觉得呛得哼吗？?" << std::endl;
    return new lagennoodle();
}
```
下面是客户端，客户端通过类别，使用相应的工厂类建立相应的实例：
``` c++
#include <iostream>
#include <string.h>

#include "haishen.h"
#include "lagen.h"
#include "noodle.h"

using namespace std;

char *product_list[] = {
    "haishen-noodle",
    "lagen-noodle",
    NULL
};

int main()
{
    char *p = NULL;
    char *pd = "haishen-noodle";
    int i = 0;

    waiter *w = NULL;
    noodle *n = NULL;

    for(p = product_list[i]; p != NULL; i++, p = product_list[i]) {
        if(strncmp(pd, p, strlen(pd)) == 0) {
           if(i == 0) {
               w = new haishen();
           } else if(i == 1) {
               w = new lagen();
           } else {
               cout << "对不起，请您排在队内！！！" << std::endl;
               break;
           }

        }
    }

    if(w) n = w->createnoodle();
    if(n) n->eating();

    if(w) {
        delete w; w = NULL;
    }

    if(n) {
        delete n; n = NULL;
    }
    return 0;
}
```
下面是CMakeList.txt文件，帮助生成Makefile：
``` shell
cmake_minimum_required(VERSION 2.8)

project(noodle-factory)
set(SRC_LIST main.cpp noodle.h waiter.h haishen.h haishen.cpp haishennoodle.h haishennoodle.cpp
    lagennoodle.h lagennoodle.cpp lagen.h lagen.cpp)
add_executable(${PROJECT_NAME} ${SRC_LIST})
```

## 编译运行结果

代码下载链接是：https://github.com/erguangqiang/freesir_headfirst/blob/master/noodle-factory.tar.gz
使用cmake生成Makefile，并编译出可执行程序noodle。运行结果如下：

``` shell
erguangqiang@elab$./noodle-factory
面是我炒得，我的名字叫海参！！！
我是海参炒面，里面没有海参哦！！吃的时候注意！
```

# 结束
工厂模式解决了简单工厂违背了的开放-封闭原则。虽然累的结构变的复杂了，但是对于扩展性得到了很大的提高。
