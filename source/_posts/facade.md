---
title: 深入浅出《设计模式》之外观模式（C++）
date: 2019-03-10 12:31:18
tags: 
    - 设计模式
    - 外观器模式
    - C++
categories: 设计模式

---

# 前言

## 模式介绍

外观模式相比较之下比较简单，模式设计中定义是为子系统中的一组接口提供一个一致的界面，此模式定义了一个高层接口，这个接口是的这一子系统更加容易使用。

如果不理解呢，简单些说就是外观模式提供了为内部提供了同意的接口层，解耦了子系统和客户端，这样客户端只需要知道外观类存在即可，不需要知道具体子系统是如何实现的。举一个简单的例子，海参面馆中两个主打面食是：

 - 海参炒面

 - 辣根汤面

两个主打面食都由一个师傅来做，这个师傅会这两种面食的做法。具体的做法呢如下：
    - 海参炒面
        1. 剥蒜
        2. 拉面
        3. 做汤底
        - 辣根汤面
        1. 制作辣根
这两种面食都有复杂的做法，调用者如果直接调用这种步骤，会让调用者与师傅的耦合性很高，如果有一天制作步骤改了，辣根汤面要剥蒜，那就需要修改调用者接口，这样是不可以的。
所以这时候师傅就是一个外观类，调用者只需要和他说，我需要一份海参炒面或者辣根汤面即可，具体如何做，就是师傅的事情了。外部不需要管。UML类图如下：

## UML类图
![外观模式UML](https://s2.ax1x.com/2019/04/16/AxtoyF.png)

## 代码实例
下面是剥蒜类实现，非常简单，只完成剥蒜动作即可。
``` c++
#ifndef GRALIC_H
#define GRALIC_H

class gralic
{
public:
    gralic();
    ~gralic();

public:
    void peelinggralic();
};

#endif // GRALIC_H
```
``` c++
#include <iostream>

#include "gralic.h"

gralic::gralic()
{

}

gralic::~gralic()
{

}

void gralic::peelinggralic()
{
    std::cout << "开始剥蒜啦！！！" << std::endl;
}
```
下面是拉面类实现，只实现拉面动作即可。
``` c++
#ifndef NOODLE_H
#define NOODLE_H


class noodle
{
public:
    noodle();
    ~noodle();

public:
    void makenoodle();
};

#endif // NOODLE_H
```
``` c++
#include <iostream>
#include "noodle.h"

noodle::noodle()
{

}

noodle::~noodle()
{

}

void noodle::makenoodle()
{
    std::cout << "开始拉面啦！！！" << std::endl;
}
```
其余子系统类似，就不在此展示了，包括：做汤底类和制作辣根类。
下面是外观模式，厨师师傅类，包含两个接口：制作海参炒面和辣根汤面。但是实现部分是使用各个子系统中的类和接口完成的，完成了客户端调用的解耦动作。
``` c++
#ifndef COOKERFACADE_H
#define COOKERFACADE_H

class gralic;
class noodle;
class soup;
class lagen;
class cookerfacade
{
public:
    cookerfacade();
    ~cookerfacade();

public:
    void makehaishennoodle();
    void makelagennoodle();

private:
    gralic *m_gra;
    noodle *m_noo;
    soup   *m_so;
    lagen  *m_lg;
};

#endif // COOKERFACADE_H
```
``` c++
#include <iostream>
#include "cookerfacade.h"
#include "gralic.h"
#include "noodle.h"
#include "soup.h"
#include "lagen.h"

cookerfacade::cookerfacade()
{
    m_gra = new gralic();
    m_noo = new noodle();
    m_so = new soup();
    m_lg = new lagen();
}

cookerfacade::~cookerfacade()
{
    delete m_gra; m_gra = NULL;
    delete m_noo; m_noo = NULL;
    delete m_so; m_so = NULL;
    delete m_lg; m_lg = NULL;
}

void cookerfacade::makehaishennoodle()
{
    m_gra->peelinggralic();
    m_noo->makenoodle();
    m_so->makesoup();

    std::cout << "开始做海参炒面啦！！！" << std::endl;
}

void cookerfacade::makelagennoodle()
{
    m_lg->makelagen();

    std::cout << "开始作辣根汤面啦！！！" << std::endl;
}
```
下面是客户端代码，可以很明显的看出他只需要知道外观师傅的类即可，别的细节并不需要知道。
``` c++
#include <iostream>
#include "cookerfacade.h"

using namespace std;

int main()
{
    cookerfacade *cf = new cookerfacade();

    cout << "老板，来一份辣根汤面！！！" << endl;
    cf->makelagennoodle();

    cout << "老板，换一份海参炒面！！！" << endl;
    cf->makehaishennoodle();
    return 0;
}
```
下面是编译需要的CMakeLists.txt文件：
``` shell
cmake_minimum_required(VERSION 2.8)

project(cooker-facade)
set(SRC_LIST main.cpp soup.h soup.cpp gralic.h gralic.cpp cookerfacade.h cookerfacade.cpp noodle.h noodle.cpp
    lagen.h lagen.cpp)
add_executable(${PROJECT_NAME} ${SRC_LIST})
```
## 编译运行结果
外观模式的源代码下载位置是：https://github.com/erguangqiang/freesir_headfirst/blob/master/cooker-facade.tar.gz
代码运行的结果如下：
``` shell
blog@blog-VirtualBox:~/build-cooker-facade-unknown-Default$ ./cooker-facade 
老板，来一份辣根汤面！！！
开始制作辣根！！！
开始作辣根汤面啦！！！
老板，换一份海参炒面！！！
开始剥蒜啦！！！
开始拉面啦！！！
开始做汤底啦！！！
开始做海参炒面啦！！！
```

# 结束语
其实外观模式我们可能一直在用，只是我们不知道自己在用外观模式而已，平时我们开发一个系统的时候，对外API完全封装内部子系统接口，以达到便于扩展可维护的目的，其实用的就是外观模式。这种模式可以充分解耦客户端和子系统。同时也是应用比较广比较简单的设计模式。
