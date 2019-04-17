---
title: 深入浅出《设计模式》之简单工厂模式（C++）
date: 2019-03-10 12:31:18
tags: 
    - 设计模式
    - 工厂模式
    - C++
categories: 设计模式
---

# 前言

## 模式介绍
简单工厂模式其实并不属于GoF23（23种设计模式），更类似工厂模式的一种变型。其定义是可以根据参数的不同返回不同类的实例。简单工厂模式专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类。
这就有点像去饭馆吃饭，进了门和服务员说一句：“waiter！来一份海参炒面！” 一般情况下他只会给你海参炒面（不管里面有没有海参）。这会第二个人进来了，也喊了一句：“服务员，来一碗辣根汤面！”此时他就会给你一份辣根汤面。你们根本不会考虑海参炒面和辣根汤面咋炒的，只需要告诉店家，他就给你了。
上面实例中的“服务员”就是一个简单工厂。饭馆里的菜单就是简单工厂中的类列表。可以通过告知服务员点哪个类，服务员将类的实例也就是菜品端出来。这样对于我这个这个客户端来讲，没必要知道后厨做了什么，只需要拿到实例菜品吃就好了。如果终于有一天海参炒面变成了烧烤店，但是我也没必要知道后厨咋串的串儿，只需要进去还找服务员，要腰子就OK了。

## UML类图
这里涉及到2种类，①我：客户端，负责调用服务员生成后端菜品实例。②服务员：工厂类，负责产生后端菜品并返回给客户端。③菜品：后端菜品类，生成的菜品。具体关系如下UML类图：

![](https://s2.ax1x.com/2019/03/10/A9ZYwR.png)
## 代码实例
下面是noodle类，是一个抽象类，里面具备一个eating函数，是为客户端准备的，食用。

``` c++
#ifndef NOODLE_H
#define NOODLE_H

class noodle {
public:
    noodle() {}
    ~noodle() {}

public:
    virtual void eating() = 0;
};

#endif // NOODLE_H
```

下面是海参炒面类，集成自noodle类，实现了eating方法：

``` c++
#ifndef HAISHENNOODLE_H
#define HAISHENNOODLE_H

#include "noodle.h"

class haishennoodle : public noodle
{
public:
    haishennoodle();
    ~haishennoodle();

public:
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

下面是辣根汤面类，继承了noodle类，重写了eating方法：

``` c++
#ifndef LAGENNOODLE_H
#define LAGENNOODLE_H

#include "noodle.h"

class lagennoodle : public noodle
{
public:
    lagennoodle();
    ~lagennoodle();

public:
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

下面是服务员类，负责生成后端noodle。通过createnoodle生成noodle实例化，是工厂。

``` c++
#ifndef WAITER_H
#define WAITER_H

class noodle;
class waiter
{
public:
    waiter();
    ~waiter();

public:
    noodle *createnoodle(int type);
};

#endif // WAITER_H
```

``` c++
#include <iostream>
#include "waiter.h"
#include "haishennoodle.h"
#include "lagennoodle.h"

waiter::waiter()
{

}

waiter::~waiter()
{

}

noodle *waiter::createnoodle(int type)
{
    noodle *n = NULL;
    switch(type) {
    case 0:
        n = new haishennoodle();
        break;
    case 1:
        n = new lagennoodle();
        break;
     default:
        std::cout << "对不起，我们这没有这个菜，请您换一个！" << std::endl;
    }

    return n;
}
```

客户端代码如下，是我进店来点餐的步骤：

```c++
#include <iostream>
#include <string.h>
#include "haishennoodle.h"
#include "lagennoodle.h"
#include "waiter.h"

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

    waiter *w = new waiter();
    noodle *n = NULL;

    for(p = product_list[i]; p != NULL; i++, p = product_list[i]) {
        if(strncmp(pd, p, strlen(pd)) == 0) {
            n = w->createnoodle(i);
            if(n) {
                cout << "开吃！！！" << endl;
                n->eating();
            }
        }
    }

    if(n) {
        delete n; n = NULL;
    }

    if(w) {
        delete w; w = NULL;
    }

    return 0;
}
```
附赠CMakeList.txt代码：
``` shell
cmake_minimum_required(VERSION 2.8)

project(noodle)
set(SRC_LIST main.cpp noodle.h lagennoodle.h lagennoodle.cpp haishennoodle.h haishennoodle.cpp waiter.h waiter.cpp)
add_executable(${PROJECT_NAME} ${SRC_LIST})

```
## 编译运行结果
代码下载链接是：https://github.com/erguangqiang/freesir_headfirst/blob/master/noodle.tar.gz
使用cmake生成Makefile，并编译出可执行程序noodle。运行结果如下：

```shell
erguangqiang@elab$./noodle
开吃！！！
我是海参炒面，里面没有海参哦！！吃的时候注意!
```

# 结束
使用简单工厂模式优点就是可以隔离客户端和实例化，这样客户端可以不理会类实例化的具体流程，达到了给客户端和逻辑解耦的目的。
但是简单工厂模式缺点也很严重，根据设计模式的开放-封闭原则，对于程序的扩展，不应改变类具体代码，可以对类集成扩展开放。一旦面馆出了新菜品，就需要修改工厂类。规避的方法是使用工厂模式，通过扩展的方式来实现。