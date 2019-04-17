---
title: 深入浅出《设计模式》之代理模式（C++）
date: 2019-03-10 12:31:18
tags: 
    - 设计模式
    - 代理模式
    - C++
categories: 设计模式

---

# 前言

## 模式介绍
***代理模式***其实非常好理解，就是在某些情况下一个对象不适合或者不能直接引用另一个对象，而使用代理对象可以解耦客户端和目标对象。起到中介的作用。
这个模式让我想起了近期曝光的外卖商家使用第三方食品厂生产的菜肴包再加热，通过外卖的方式售卖给客人，外卖商家几乎不需要有任何技术，只需要有一台微波炉，加热食物即可。每份成本才几块钱，但是客人购买的时候往往需要十几甚至几十块。客人也不知道食品厂的存在。这就是典型的代理模式，[资讯传送门](http://www.sohu.com/a/246764063_673185)。
我们还是以海参面馆为例，海参面馆主打两种面：海参炒面、辣根汤面。但是老板为了追逐利润，通过关系找到第三方食品厂购买了大量的菜肴包，低价格购入。通过门店，再加热高价出售。
针对上面的例子，可以发现其实海参面馆只是代理了第三方食品厂。这种情况下食品厂不适宜暴露给客户，客户只知道海参面馆的存在，并不知道食品厂的存在，还是以为是海参大厨做的美味。其中包含几个类：

  - 第三方食品厂：这个类负责产出菜品：海参炒面和辣根汤面。
  - 海参面馆：这个类负责与客户交互，反馈给客户海参炒面和辣根汤面。
  - 客户端：这个并非是类，只是调用海参面馆这个代理的客户。

## UML类图
![海参面馆代理模式UML类图](https://s2.ax1x.com/2019/03/16/AZNK1I.png)
## 代码实例

首先是一个食品生产抽象类，抽象具备制作海参炒面和辣根汤面的能力。需要在子类中实现：

``` c++
#ifndef FOODPRODUCER_H
#define FOODPRODUCER_H


class foodproducer
{
public:
    foodproducer() {}
    ~foodproducer() {}

    virtual void create_haishennoodle() = 0;
    virtual void create_lagennoodle() = 0;
};

#endif // FOODPRODUCER_H
```

然后是食品工厂类，继承自`foodproducer`，自己生产海参炒面和辣根汤面。但是并不对私人销售这些产品，且售价低廉：

``` c++
#ifndef FOODFACTORY_H
#define FOODFACTORY_H

#include "foodproducer.h"

class foodfactory : public foodproducer
{
public:
    foodfactory();
    ~foodfactory();

    virtual void create_haishennoodle();
    virtual void create_lagennoodle();
};

#endif // FOODFACTORY_H
```

``` c++
#include <iostream>
#include "foodfactory.h"

foodfactory::foodfactory()
{

}

foodfactory::~foodfactory()
{

}

void foodfactory::create_haishennoodle()
{
    std::cout << "我工厂出货一份5元海参炒面！！！" << std::endl;
}

void foodfactory::create_lagennoodle()
{
    std::cout << "我工厂出货一份3元辣根汤面！！！" << std::endl;
}
```

最后是我们平时吃的海参面馆类，里面对外宣称自己做的海参炒面和辣根汤面，其实都是来自于食品工厂的，代理了食品工厂类。

``` c++
#ifndef HAISHENSHOP_H
#define HAISHENSHOP_H

#include "foodproducer.h"

class foodfactory;
class haishenshop : public foodproducer
{
public:
    haishenshop();
    ~haishenshop();

    virtual void create_haishennoodle();
    virtual void create_lagennoodle();

private:
    foodfactory  *m_fac;
};

#endif // HAISHENSHOP_H
```

``` c++
#include <iostream>
#include "haishenshop.h"
#include "foodfactory.h"

haishenshop::haishenshop()
{
    m_fac = new foodfactory();
}

haishenshop::~haishenshop()
{
    delete m_fac;
    m_fac = NULL;
}

void haishenshop::create_haishennoodle()
{
    std::cout << "卖出18.00元海参炒面,我是从工厂进的货！！" << std::endl;
    m_fac->create_haishennoodle();
}

void haishenshop::create_lagennoodle()
{
    std::cout << "卖出12元辣根汤面,我是从工厂进的货！！" << std::endl;
    m_fac->create_lagennoodle();
}
```

接着是客户端调用海参面馆模拟点餐环境：

``` c++
#include <iostream>
#include "haishenshop.h"

using namespace std;

int main()
{
    haishenshop *hs = new haishenshop();

    cout << "我在美团点了一份海参炒面！！！" << endl;
    hs->create_haishennoodle();

    cout << "我在美团点了一份辣根汤面！！！" << endl;
    hs->create_lagennoodle();
    return 0;
}
```



下面是`CMakeList.txt`文件，帮助生成`Makefile`：

``` she
cmake_minimum_required(VERSION 2.8)

project(food-proxy)
set(SRC_LIST main.cpp foodproducer.h foodfactory.h foodfactory.cpp haishenshop.h haishenshop.cpp)
add_executable(${PROJECT_NAME} ${SRC_LIST})
```



## 编译运行结果

代码下载链接是：https://github.com/erguangqiang/freesir_headfirst/blob/master/food-proxy.tar.gz
使用`cmake`生成`Makefile`，并编译出可执行程序`food-proxy`。运行结果如下：

``` shell
blog@blog-VirtualBox:~/build-food-proxy-unknown-Default$ ./food-proxy 
我在美团点了一份海参炒面！！！
卖出18.00元海参炒面,我是从工厂进的货！！
我工厂出货一份5元海参炒面！！！
我在美团点了一份辣根汤面！！！
卖出12元辣根汤面,我是从工厂进的货！！
我工厂出货一份3元辣根汤面！！！
```



# 结束语

代理模式能够起到的作用就是调用这和和真实实现之间的耦合性。达到解耦的目的。比较适用于IO请求或者耗时时间长的场景。通过代理也可以做出很多其他的动作。比如在上面的例子中，可以针对不同的食品添加订餐份数参数，这些在食品工厂中是没办法统计的。

