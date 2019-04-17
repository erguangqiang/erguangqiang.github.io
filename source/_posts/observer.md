---
title: 深入浅出《设计模式》之观察者模式（C++）
date: 2019-03-10 12:31:18
tags: 
    - 设计模式
    - 观察者模式
    - C++
categories: 设计模式
---

# 前言

## 模式介绍

观察者模式在Design Parttern中定义如下：

> **观察者模式**定义了一种一对多的依赖关系，让多个观察者对象同时监听某一个主题对象。这个主题对象在状态发生变化时会通知所有观察者对象，使他们能够自动更新自己。

以上的定义估计和我一样，能看懂的只有解决一种一对多的依赖关系。其实这个说白了很简单，就是解决一个一对多的依赖关系。多个观察者同时观测一种状态。也就是这种状态变化了，观察者们都会有自己的变化。按照我们正常的逻辑肯定会写一个循环，通过循环来调用观察者们。但是这种循环的写法就会增加调用者和观察者的耦合性，所以使用了观察者模式为这类问题调用者和观察者解耦。

如果还没看明白，没关系，我们还是举例说明。海参面馆以前都是不送外面的，因为单独雇佣几个外面员成本太高了。但是现在好了，有了美团和饿了么这些外面平台。我只需要将面馆里面的面做好等着平台骑手来取并送餐即可。吃饭的客户端只需要在app中下单，海参面馆接受订单并制作，骑手抢单并送货。这里骑手和面馆就是一个一对多的依赖关系。所有骑手等待抢单，当客户下单后，平台通知所有附近的骑手们有新的外卖订单，骑手们开始抢单。抢到订单后反馈给app哪一个骑手抢单订单。这就是一个典型的观察者模式。

## UML类图

![观察者模式UML类图](https://s2.ax1x.com/2019/03/13/AksCtg.png)

- 美团类就是一个平台抽象来，定义了观察者保存容器的内容。

- 海参面馆类，就是一个平台具体类，继承自美团类。这时海参面馆相当于没有雇人但是多出了好几个骑手为自己服务。
- 骑手类就是所有骑手的抽象类。
- zhangsan、sili、wangwu是具体的骑手类。所有骑手都会添加到海参面馆的容器中去。此时所有的骑手都会侦听面馆。有订单的时候面馆会调用notify通知所有骑手开始抢单。

## 代码实例

下面是平台抽象类，具有增加删除通知骑手新订单行为并且具备新订单地址信息：

```C++
#ifndef PORTAL_H
#define PORTAL_H
#include <list>
#include <string>

class qishou;

typedef std::list<qishou *>  qslist;
class portal
{
public:
    portal() {}
    virtual ~portal() {}

    virtual int attach(qishou *q) = 0;
    virtual int detach(qishou *q) = 0;

    virtual int notify() = 0;

    std::string get_address() { return m_address; }
    void set_address(std::string &s) { m_address = s; }

protected:
    qslist   m_list;
    std::string  m_address;
};

#endif // PORTAL_H
```



下面是海参面馆类，继承了平台的抽象类，实现了骑手添加删除和通知动作，相当于海参面馆自己雇佣了大批的骑手帮自己送外卖：

``` C++
#ifndef HAISHENNOODLE_H
#define HAISHENNOODLE_H

#include "portal.h"

class qishou;
class haishennoodle : public portal
{
public:
    haishennoodle();
    ~haishennoodle();

    virtual int attach(qishou *q);
    virtual int detach(qishou *q);

    virtual int notify();
};

#endif // HAISHENNOODLE_H
```

```  c++
#include "haishennoodle.h"
#include "qishou.h"

haishennoodle::haishennoodle()
{
    m_address = "";
}

haishennoodle::~haishennoodle()
{
    qslist::iterator it = m_list.begin();
    for(; it != m_list.end(); it++) {
        if(*it) {
            delete *it;
        }
    }

    m_list.clear();
}

int haishennoodle::attach(qishou *q)
{
    qslist::iterator it = m_list.begin();
    for(; it != m_list.end(); it++) {
        if(q == *it) {
            return 0;
        }
    }

    m_list.push_back(q);
    return 0;
}

int haishennoodle::detach(qishou *q)
{
    qslist::iterator it = m_list.begin();
    for(; it != m_list.end(); it++) {
        if(q == *it) {
            delete q; q = NULL;
            it = m_list.erase(it);
        }
    }

    return 0;
}

int haishennoodle::notify()
{
    qslist::iterator it = m_list.begin();
    for(; it != m_list.end(); it++) {
        (*it)->new_order();
    }
    return 0;
}
```

下面是骑手抽象类，具备平台通知动作，具体骑手抢单功能呢暂时是不是我们讨论的范围：

``` C++
#ifndef QISHOU_H
#define QISHOU_H

class portal;
class qishou
{
public:
    qishou() {}
    virtual ~qishou() {}

    virtual int new_order() = 0;

protected:
    portal  *m_portal;
};

#endif // QISHOU_H
```

张三和李四分别是平台的骑手，可以添加到海参面馆的骑手队列中去，这样只要有新的订单，就一定会通知到这两位骑手：

``` c++
#ifndef ZHANGSAN_H
#define ZHANGSAN_H

#include "qishou.h"

class portal;
class zhangsan : public qishou
{
public:
    zhangsan(portal *p);
    ~zhangsan();

    int new_order();
};

#endif // ZHANGSAN_H
```

``` c++
#include <iostream>

#include "zhangsan.h"
#include "portal.h"

zhangsan::zhangsan(portal *p)
{
    m_portal = p;
}

zhangsan::~zhangsan()
{

}

int zhangsan::new_order()
{
    std::cout << "zhangsan has one new order. adress is " << m_portal->get_address() << std::endl;
}
```

``` c++
#ifndef LISI_H
#define LISI_H

#include "qishou.h"

class portal;
class lisi : public qishou
{
public:
    lisi(portal *p);
    ~lisi();

    int new_order();
};

#endif // LISI_H
```

``` c++
#include <iostream>

#include "portal.h"
#include "lisi.h"

lisi::lisi(portal *p)
{
    m_portal = p;
}

lisi::~lisi()
{

}

int lisi::new_order()
{
    std::cout << "lisi has one new order. adress is " << m_portal->get_address() << std::endl;
}
```

客户端代码如下：

``` c++
#include <iostream>
#include "haishennoodle.h"
#include "zhangsan.h"
#include "lisi.h"

using namespace std;

int main()
{
    haishennoodle *hsn = new haishennoodle();
    qishou *q1 = new zhangsan(hsn);
    qishou *q2 = new lisi(hsn);

    hsn->attach(q1);
    hsn->attach(q2);

    cout << "has new order!!!" << endl;
    string add1 = "shuianhuayuan 12#904";
    hsn->set_address(add1);
    hsn->notify();

    hsn->detach(q2);
    string add2 = "shuianhuayuan 12#903";
    hsn->set_address(add2);
    cout << "has new order !!!"<< endl;
    hsn->notify();

    delete hsn;
    hsn = NULL;
    return 0;
}
```

下面是CMakeList.txt文件，帮助生成Makefile：

``` shell
cmake_minimum_required(VERSION 2.8)

project(cook-observer)
set(SRC_LIST main.cpp portal.h qishou.h haishennoodle.h haishennoodle.cpp zhangsan.h zhangsan.cpp lisi.h lisi.cpp)
add_executable(${PROJECT_NAME} ${SRC_LIST})
```



## 编译运行结果

代码下载链接是：https://github.com/erguangqiang/freesir_headfirst/blob/master/cook-observer.tar.gz
使用cmake生成Makefile，并编译出可执行程序cook-observer。运行结果如下：

``` she
blog@blog-VirtualBox:~/build-cook-observer-unknown-Default$ ./cook-observer 
has new order!!!
zhangsan has one new order. adress is shuianhuayuan 12#904
lisi has one new order. adress is shuianhuayuan 12#904
has new order !!!
zhangsan has one new order. adress is shuianhuayuan 12#903
```



# 结束语

观察者模式非常适合一对多场景的描述。如果是事件驱动场景，发生事件A的同时需要通知许多模块A事件，可用观察者模式。开放-封闭原则可以得到很好的保留。A模块不必要修改，只需要将观察者添加或者删除出观察列表就ok了。非常的方便。