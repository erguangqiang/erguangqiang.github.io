---
title: 深入浅出《设计模式》之装饰器模式（C++）
date: 2019-03-10 12:31:18
tags: 
    - 设计模式
    - 装饰器模式
    - C++
categories: 设计模式
---

# 前言
## 模式介绍
装饰器模式允许向一个现有的对象添加新的功能，同时又不改变其结构。在遵循开放-封闭原则的前提下通过包装的方式动态为原有的对象添加新的功能。
如果上面看不懂，可以还是以海参面馆举个例子。海参面馆经营海参炒面和辣根汤面两种面条系列，但是老板为了方便顾客，增加了卤蛋、海带和丸子等配菜。每个进门来吃饭的客人，在点完面之后都会有服务员问一下需要配菜不，需要的话都会在后厨配备好之后端到大堂食用。整个过程都是连续的，也就是说整个过程面条和配菜都是从一个碗里端到客人面前的，不会是先端来一碗面，然后在拿个卤蛋，再拿个丸子。
从上面的例子中可以看出向海带、卤蛋和丸子都只是面条的配菜（装饰）。现在假设每样菜品都有自己的价格，如何在计算面条钱的基础之上同时计算出配菜（装饰）的价钱，并自动求和。
	- 海参炒面    18.00元/份
	- 辣根汤面    12.00元/份
	- 卤蛋            1.50元/个
	- 海带            2.00元/个
	- 丸子 	    1.80元/个
## UML类图
这个设计模式中涉及到几种类：
	- 食物类，是抽象类。所有类都会继承与这个类，这个类主要是为了多态时使用的
	- 海参炒面和辣根汤面类，是继承食物类的实体类
	- 装饰类，是继承于食物类的又一基类
	- 海带、卤蛋和丸子类，是继承于装饰类的实体类
![装饰器模式UML类图](https://s2.ax1x.com/2019/03/12/AirZdg.png)
## 代码实例
下面是食物类，此类事所有类的基类。
``` c++
#ifndef __FOOD__H__
#define __FOOD__H__

class food
{
protected:
	food() {}
	~food() {};

public:
	virtual double get_price() = 0;
	virtual char *get_food_name() = 0;

protected:
	char *m_food;
	double m_price;
};

#endif//__FOOD__H__
```
下面是海参炒面实体类，继承自食物类，实现了食物类的成员函数。
``` c++
#ifndef __HAISHENNOODLE__H__
#define __HAISHENNOODLE__H__

#include "food.h"
class haishennoodle : public food
{
public:
	haishennoodle();
	~haishennoodle();

	double get_price();
	char *get_food_name();
};

#endif//__HAISHENNOODLE__H__
```
``` c++
#include <iostream>
#include "haishennoodle.h"

haishennoodle::haishennoodle()
{
	m_food = "海参炒面";
	m_price = 18.00;
}

haishennoodle::~haishennoodle()
{
}

double haishennoodle::get_price()
{
	std::cout << "海参炒面     1份     18.00元/份" << std::endl;
	return m_price;
}

char *haishennoodle::get_food_name()
{
	return m_food;
}
```
下面是辣根汤面类，继承自食物类，重写了食物类的成员函数
``` c++
#ifndef __LAGENNOODLE__H__
#define __LAGENNOODLE__H__

#include "food.h"

class lagennoodle : public food
{
public:
	lagennoodle();
	~lagennoodle();

	double get_price();
	char *get_food_name();
};

#endif//__LAGENNOODLE__H__
```
``` c++
#include <iostream>
#include "lagennoodle.h"

lagennoodle::lagennoodle()
{
	m_food = "辣根汤面";
	m_price = 12.00;
}

lagennoodle::~lagennoodle()
{
	
}

double lagennoodle::get_price()
{
	std::cout << "辣根汤面     1份     12.00元/份" << std::endl;
	return m_price;
}

char *lagennoodle::get_food_name()
{
	return m_food;
}
```
下面是装饰类，继承自食物类，这样可以充分利用多态特性实现装饰效果
``` c++
#ifndef __DECORATE__H__
#define __DECORATE__H__

#include "food.h"

class decorate : public food
{
protected:
	decorate() {};
	~decorate() {};

protected:
	food  *m_dec;
};

#endif//__DECORATE__H__
```
下面是海带类，继承自装饰类，重写了食物的成员函数，并做出了装饰行为（装饰类以此为例）。
``` c++
#ifndef __HAIDAI__H__
#define __HAIDAI__H__

#include "decorate.h"

class haidai : public decorate
{
public:
	haidai(food *d);
	~haidai();

	double get_price();
	char * get_food_name() { return m_food; }
};

#endif//__HAIDAI__H__
```
```  c++
#include <iostream>
#include "haidai.h"

haidai::haidai(food *d)
{
	m_food = "海带";
	m_price = 2.00;
	m_dec = d;
}

haidai::~haidai()
{

}

double haidai::get_price()
{
	if(m_dec) {
		std::cout << "海带    1份    2.00元/份" << std::endl;
		return m_dec->get_price() + m_price;
	}
	return m_price;
}
```
下面是客户端调用的过程，分别是点了一个海参炒面，点了一个辣根汤面，还有以此没有点面，直接点的装饰品。。。
``` c++
#include <iostream>

#include "haishennoodle.h"
#include "lagennoodle.h"

#include "wanzi.h"
#include "haidai.h"
#include "ludan.h"

using namespace std;

int main(int argc, char **argv)
{
	food *f = NULL;

	cout << "老板，来一份海参炒面！！" << endl;
	f = new haishennoodle();

	cout << "老板，海参炒面里面加一个卤蛋和海带！！" << endl;
	decorate *hd = new haidai(f);
	decorate *ld = new ludan(hd);
	
	//开始给装饰海参炒面
	cout << "您的面钱一共是 ：" << endl;
	cout << ld->get_price() << endl;
	delete (haishennoodle *)f; f = NULL;
	delete (ludan *)ld; ld = NULL;
	delete (haidai *)hd; hd = NULL;

	cout << "--------------------------------------------" << endl;
	cout << "老板，来一份辣根汤面，加两个卤蛋和一个丸子！！！" << endl;
	f = new lagennoodle();
	decorate *ld1 = new ludan(f);
	decorate *ld2 = new ludan(ld1);
	decorate *wz = new wanzi(ld2);

	//开始装饰辣根汤面
	cout << "您的面钱一共是 ： " << endl;
	cout << wz->get_price() << endl;
	delete (lagennoodle *)f; f = NULL;
	delete (ludan *)ld1; ld1 = NULL;
	delete (ludan *)ld2; ld2 = NULL;
	delete (wanzi *)wz; wz = NULL;

	cout << "--------------------------------------------" << endl;
	cout << "老板，不要面，来3个卤蛋，两个海带，5个丸子！！！" << endl;
	decorate *ld3 = new ludan(NULL);
	decorate *ld4 = new ludan(ld3);
	decorate *ld5 = new ludan(ld4);

	decorate *hd1 = new haidai(ld5);
	decorate *hd2 = new haidai(hd1);

	decorate *wz1 = new wanzi(hd2);
	decorate *wz2 = new wanzi(wz1);
	decorate *wz3 = new wanzi(wz2);
	decorate *wz4 = new wanzi(wz3);
	decorate *wz5 = new wanzi(wz4);

	cout << "您的面钱一共是 ： " << endl;
	cout << wz5->get_price() << endl;
	delete (ludan *)ld3; ld3 = NULL;
	delete (ludan *)ld4; ld4 = NULL;
	delete (ludan *)ld5; ld5 = NULL;
	delete (haidai *)hd1; hd1 = NULL;
	delete (haidai *)hd2; hd2 = NULL;
	delete (wanzi *)wz1; wz1 = NULL;
	delete (wanzi *)wz2; wz2 = NULL;
	delete (wanzi *)wz3; wz3 = NULL;
	delete (wanzi *)wz4; wz4 = NULL;
	delete (wanzi *)wz5; wz5 = NULL;
	cout << "--------------------------------------------" << endl;

	return 0;
}
```
下面是CMakeLists.txt文件代码，可以自动生成Makefile文件使用
``` shell
cmake_minimum_required(VERSION 2.8)

project(cooker_decorate)
set(SRC_LIST main.cpp food.h haishennoodle.h haishennoodle.cpp lagennoodle.h lagennoodle.cpp
	wanzi.h wanzi.cpp haidai.h haidai.cpp ludan.h ludan.cpp)
add_executable(${PROJECT_NAME} ${SRC_LIST})
```

## 编译运行结果
``` shell
erguangqiang@elab:~$ ./cooker_decorate 
老板，来一份海参炒面！！
老板，海参炒面里面加一个卤蛋和海带！！
您的面钱一共是 ：
卤蛋    1份    1.50元/份
海带    1份    2.00元/份
海参炒面     1份     18.00元/份
21.5
--------------------------------------------
老板，来一份辣根汤面，加两个卤蛋和一个丸子！！！
您的面钱一共是 ： 
丸子    1份    1.80元/份
卤蛋    1份    1.50元/份
卤蛋    1份    1.50元/份
辣根汤面     1份     12.00元/份
16.8
--------------------------------------------
老板，不要面，来3个卤蛋，两个海带，5个丸子！！！
您的面钱一共是 ： 
丸子    1份    1.80元/份
丸子    1份    1.80元/份
丸子    1份    1.80元/份
丸子    1份    1.80元/份
丸子    1份    1.80元/份
海带    1份    2.00元/份
海带    1份    2.00元/份
卤蛋    1份    1.50元/份
卤蛋    1份    1.50元/份
17.5
--------------------------------------------
```

# 结束语
装饰模式的优势是符合了开放-封闭原则，使类动作更容易扩展。便于了维护。但是这个设计模式也是有缺陷了，会让类的层次和结构编的更加复杂，掌握起来更困难。
