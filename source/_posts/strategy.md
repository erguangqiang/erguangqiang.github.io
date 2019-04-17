---
title: 深入浅出《设计模式》之策略模式（C++）
date: 2019-03-10 12:31:18
tags: 
    - 设计模式
    - 策略模式
    - C++
categories: 设计模式
---

# 前言

## 模式介绍
策略模式是设计模式中行为模式的一种，定义就是一个类的行为或其算法可以在运行时更改。按照简单的逻辑，都是封装好一系列的类的行为或算法，通过客户端进行调用，不过如果累的行为或算法更改或者扩展，那么就需要随之更新客户端调用的逻辑。这样类的行为或算法与客户端耦合性就太高了。策略模式就是通过增加了程序上下文实例，来调节类的行为或算法。如果看不懂这段，可以跳过，我举个简单的例子，还拿海参炒面举例吧，谁让海参炒面热门呢。
海参炒面面馆大师傅呢就会做两种面：
	- 海参炒面
	- 辣根汤面
这两种面大家都知道使用不同的材料和工序制成。我来吃面其实不需要知道面到底是怎么做的（只需要炒面换汤面就ok了），只要我和服务员说一下，我要海参炒面应该就只等着吃了。
如果将海参炒面和辣根汤面的制作流程算作是两种不同的算法的话，那客户端（我）也就不需要考虑他们的具体流程了，只需要告诉大师傅cooking it。而不是我自己去后厨，拿起菜刀和大勺，开启中华小当家模式了，也没那实力（因为我的名字不叫海参）。

## UML类图
![策略模式类图](https://s2.ax1x.com/2019/03/11/ACH2WV.png)

## 代码实例
代码中包含三种角色：①客户端（我）：负责点菜，负责吃。②上下文类（context）：负责调度算法，封装算法。③菜品类。
下面代码是厨师抽象类，并在内定义出厨师做饭的每步的方法。

``` c++
#ifndef __COOKER__H__
#define __COOKER__H__

class cooker 
{
public:
    cooker() {}
    ~cooker() {}

public:
    virtual void take_oil() = 0;
    virtual void take_vegetable() = 0;
    virtual void take_noodle() = 0;
    virtual void take_salt() = 0;
};

#endif//__COOKER__H__
```
下面是定义和实现了haishen类和辣根类，继承自cooker类，实现了cooker里面所有成员函数。
``` c++
#ifndef __HAISHEN__H__
#define __HAISHEN__H__

#include "cooker.h"

class haishen : public cooker
{
public:
    haishen();
    ~haishen();

public:
    virtual void take_oil();
    virtual void take_vegetable();
    virtual void take_noodle();
    virtual void take_salt();
};

#endif//__HAISHEN__H__
```
``` c++
#include <iostream>
#include "haishen.h"

haishen::haishen()
{
}

haishen::~haishen()
{
}

void haishen::take_oil()
{
    std::cout << "海参炒面，起锅烧油！！！" << std::endl;
}

void haishen::take_vegetable()
{
    std::cout << "海参炒面，没有海参，炝锅放蔬菜！！！" << std::endl;
}

void haishen::take_noodle()
{
    std::cout << "海参炒面，没有海参，放点面条对付对付吧！！！" << std::endl;
}

void haishen::take_salt()
{
    std::cout << "海参炒面，没有海参，放点盐出锅！！！" << std::endl;
}
```
``` c++
#ifndef __LAGEN__H__
#define __LAGEN__H__

#include "cooker.h"

class lagen : public cooker
{
public:
    lagen();
    ~lagen();

public:
    virtual void take_oil();
    virtual void take_vegetable();
    virtual void take_noodle();
    virtual void take_salt();
};

#endif//__LAGEN__H__
```
``` c++
#include <iostream>
#include "lagen.h"

lagen::lagen()
{
}

lagen::~lagen()
{
}

void lagen::take_oil()
{
    std::cout << "辣根汤面不放油，只放辣根！！！" << std::endl;
}

void lagen::take_vegetable()
{
    std::cout << "辣根汤面不放蔬菜，只放辣根！！！" << std::endl;
}

void lagen::take_noodle()
{
    std::cout << "辣根汤面不放面条，只放辣根！！！" << std::endl;
}

void lagen::take_salt()
{
    std::cout << "辣根汤面不放盐，只放辣根！！！" << std::endl;
}
```
策略模式中需要一个调度上下文算法的类，下面这个context类就是调度。可将实例化的菜品类传入context，context会封装菜品的算法过程，这样以达到和客户端解耦的目的。
``` c++
#ifndef __CONTEXT__H__
#define __CONTEXT__H__

class cooker;
class context
{
public:
    context();
    ~context();

public:
	virtual void set_cooker(cooker *c);
    virtual void cooking();

private:
    cooker *m_cooker;
};

#endif//__CONTEXT__H__

```
``` c++
#include <iostream>
#include "context.h"

#include "cooker.h"

context::context() : m_cooker(NULL)
{
}

context::~context()
{
}

void context::set_cooker(cooker *c)
{
    if(c) m_cooker = c;
}

void context::cooking()
{
    if(m_cooker) {
        std::cout << "放油：";
        m_cooker->take_oil();

        std::cout << "放蔬菜：";
        m_cooker->take_vegetable();

        std::cout << "放面条：";
        m_cooker->take_noodle();

        std::cout << "放盐：";
        m_cooker->take_salt();
    }
}
```
客户端代码与之前类似。
``` c++
#include <iostream>
#include <string.h>

#include "context.h"
#include "lagen.h"
#include "haishen.h"

using namespace std;

int main()
{
    context *c = new context();
    cooker *cooker = NULL;

    cout << "老板，来一份海参炒面！！" << endl;
    //实例化海参炒面算法
    cooker = new haishen();

    if(c) {
        c->set_cooker(cooker);
        c->cooking();
    }
    delete (haishen *)cooker; cooker = NULL;

    cout << "------------------------------" << endl;

    cout << "老板，我要用炒面换汤面！！！" << endl;
    //实例化辣根汤面算法
    cooker = new lagen();

    if(c) {
        c->set_cooker(cooker);
        c->cooking();
    }
    delete (lagen *)cooker; cooker = NULL;

    cout << "-----------------------------" << endl;

    if(c) {
        delete c; c = NULL;
    }

    return 0;
}
```
为了编译方便，下面给出CMakeLists.txt代码。
``` shell
cmake_minimum_required(VERSION 2.8)

project(cooker)
set(SRC_LIST main.cpp cooker.h haishen.h haishen.cpp lagen.h lagen.cpp context.h context.cpp)
add_executable(${PROJECT_NAME} ${SRC_LIST})
```

## 编译运行结果
``` shell
erguangqiang@elab:~$ ./cooker
老板，来一份海参炒面！！
放油：海参炒面，起锅烧油！！！
放蔬菜：海参炒面，没有海参，炝锅放蔬菜！！！
放面条：海参炒面，没有海参，放点面条对付对付吧！！！
放盐：海参炒面，没有海参，放点盐出锅！！！
------------------------------
老板，我要用炒面换汤面！！！
放油：辣根汤面不放油，只放辣根！！！
放蔬菜：辣根汤面不放蔬菜，只放辣根！！！
放面条：辣根汤面不放面条，只放辣根！！！
放盐：辣根汤面不放盐，只放辣根！！！
-----------------------------
```

# 结束语
策略模式非常适合那种有多种算法或类行为被选择其中之一或其中之几。示例中还可以在运行过程中对算法和类行为进行重新制定。这个就不在这里提了。
