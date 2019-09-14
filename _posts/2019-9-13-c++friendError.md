---
layout:     post
title:      友元、异常和其它
subtitle:   c++
date:       2019-9-13
author:     Jow
header-img: img/about-bg-walle.jpg
catalog: 	 true 
tags:
    - C++

---

### 目录
1. 友元
2. 移动语义和右值引用
3. 委托构造函数
4. 继承构造函数
5. 管理虚方法：override和final
6. 嵌套类
7. 异常


> Study hard, work hard.


## 友元
友元类的所有方法都可以访问原始类的私有成员和保护成员。另外可以做更严格的限制，只将特定过得成员函数指定为另一个类的友元。那些函数、成员函数或类为友元是由类定义的，而不能从外部强加友情。因此，尽管友元被授予从外部访问类的私有部分权限，但他们并不与面向对象的编程思想相悖：相反，它们提高了公有接口的灵活性。

```c++
//tv.h
#pragma once
class Tv
{
private:
	int state;
	int volume;
	int maxchannel;
	int channel;
	int mode;
	int input;
public:
	friend class Remote; // Remote can access Tv private parts
	//friend void Remote::set_chan(Tv& t, int c);
	enum {
		Off, On
	};
	enum {
		MinVal, MaxVal = 20
	};
	enum {
		Antenna, Cable
	};
	enum {
		TV, DVD
	};

	Tv(int s = Off, int mc = 125) :state(s), volume(5), maxchannel(mc), channel(2),mode(Cable), input(TV) {}
	void onoff() { state = (state == On) ? Off : On; }
	bool ison() const { return state == On; }
	bool valup();
	bool valdown();
	void chanup();
	void chandown();
	void set_mode() { mode = (mode == Antenna) ? Cable : Antenna; }
	void set_input() { input = (input == TV) ? DVD : TV; }
	void settings() const;//display all settings
	void buzz(Remote & r);
};

class  Remote
{
public:
	enum State {
		Off, On
	};
	enum {
		MinVal, MaxVal = 20
	};
	enum {
		Antenna, Cable
	};
	enum {
		TV, DVD
	};
	Remote(int m = Tv::TV) :mode(m) {}
	bool volup(Tv& t) {
		return t.valup();
	}
	bool voldown(Tv& t) {
		return t.valdown();
	}
	void onoff(Tv& t) {
		t.onoff();
	}
	void chanup(Tv& t) {
		t.chanup();
	}
	void chandown(Tv& t) {
		t.chandown();
	}
	void set_chan(Tv& t, int c);
	void set_mode(Tv& t) { t.set_mode(); }
	void set_input(Tv& t) { t.set_input(); }

private:
	int mode;
};


//Tv.cpp
#include<iostream>
#include "Tv.h"

bool Tv::valup()
{
	if (volume < MaxVal)
	{
		volume++;
		return true;
	}
	else
		return false;
}

bool Tv::valdown()
{
	if (volume > MinVal)
	{
		volume--;
		return true;
	}
	else
		return false;
}

void Tv::chanup()
{
	if (channel < maxchannel)
	{
		channel++;
	}
	else
		channel = 1;
}

void Tv::chandown()
{
	if (channel > 1)
	{
		channel--;
	}
	else
		channel = maxchannel;
}

void Tv::settings() const
{
	using std::cout;
	using std::endl;
	cout << "TV is " << (state == Off ? "Off" : "On") << endl;
	if (state == On)
	{
		cout << "Volume serring = " << volume << endl;
		cout << "Channel setting = " << channel << endl;
		cout << "Mode = " << (mode == Antenna ? "antenna" : "cable") << endl;
		cout << "Input = " << (input == TV ? "TV" : "DVD") << endl;
	}
}

void Remote::set_chan(Tv& t, int c)
{
	t.channel = c; //作为友元类访问私有变量
}


#include<iostream>
#include "Tv.h"

bool Tv::valup()
{
	if (volume < MaxVal)
	{
		volume++;
		return true;
	}
	else
		return false;
}

bool Tv::valdown()
{
	if (volume > MinVal)
	{
		volume--;
		return true;
	}
	else
		return false;
}

void Tv::chanup()
{
	if (channel < maxchannel)
	{
		channel++;
	}
	else
		channel = 1;
}

void Tv::chandown()
{
	if (channel > 1)
	{
		channel--;
	}
	else
		channel = maxchannel;
}

void Tv::settings() const
{
	using std::cout;
	using std::endl;
	cout << "TV is " << (state == Off ? "Off" : "On") << endl;
	if (state == On)
	{
		cout << "Volume serring = " << volume << endl;
		cout << "Channel setting = " << channel << endl;
		cout << "Mode = " << (mode == Antenna ? "antenna" : "cable") << endl;
		cout << "Input = " << (input == TV ? "TV" : "DVD") << endl;
	}
}

void Remote::set_chan(Tv& t, int c)
{
	t.channel = c; //作为友元类访问私有变量
}

//main.cpp
#include <iostream>
#include"tv.h"
int main()
{
	using std::cout;
	Tv s42;
	cout << "Initial settings for 42 Tv:\n";
	s42.settings();
	s42.onoff();
	s42.chanup();
	cout << "\nAdjusted setting for 42 TV:\n";
	Remote grey;
	grey.voldown(s42);
}

```
从上面的例子我们可以发现，其实只有设置频道那里用到了私有变量，所以只需要让那一个设置频道的函数作为Tv类的友元函数就行了。

但是这样就需要知道Remote类中的函数了，但是上面的例子中，Remote还没有被定义，所以这个时候需要先定义Remote类，这样又存在一个问题，那就是Tv类没有定义，所以在Remote类中不能使用Tv类中的任何东西，只能先声明，然后再Tv.ccp类中定义，而且这些定义要在Tv函数定义之后，要不然还是找不到。

```c++
//h
#pragma once
class Tv;
class  Remote
{
public:
	enum State {
		Off, On
	};
	enum {
		MinVal, MaxVal = 20
	};
	enum {
		Antenna, Cable
	};
	enum {
		TV, DVD
	};
	Remote(int m = TV) :mode(m) {}
	bool volup(Tv& t);
	bool voldown(Tv& t);
	void onoff(Tv& t);
	void chanup(Tv& t);
	void chandown(Tv& t);
	void set_chan(Tv& t, int c);
	void set_mode(Tv& t);
	void set_input(Tv& t);

private:
	int mode;
};

class Tv
{
private:
	int state;
	int volume;
	int maxchannel;
	int channel;
	int mode;
	int input;
public:
	//friend class Remote; // Remote can access Tv private parts
	friend void Remote::set_chan(Tv& t, int c);
	enum {
		Off, On
	};
	enum {
		MinVal, MaxVal = 20
	};
	enum {
		Antenna, Cable
	};
	enum {
		TV, DVD
	};

	Tv(int s = Off, int mc = 125) :state(s), volume(5), maxchannel(mc), channel(2),mode(Cable), input(TV) {}
	void onoff() { state = (state == On) ? Off : On; }
	bool ison() const { return state == On; }
	bool valup();
	bool valdown();
	void chanup();
	void chandown();
	void set_mode() { mode = (mode == Antenna) ? Cable : Antenna; }
	void set_input() { input = (input == TV) ? DVD : TV; }
	void settings() const;//display all settings
};


//cpp
#include<iostream>
#include "Tv.h"

bool Tv::valup()
{
	if (volume < MaxVal)
	{
		volume++;
		return true;
	}
	else
		return false;
}

bool Tv::valdown()
{
	if (volume > MinVal)
	{
		volume--;
		return true;
	}
	else
		return false;
}

void Tv::chanup()
{
	if (channel < maxchannel)
	{
		channel++;
	}
	else
		channel = 1;
}

void Tv::chandown()
{
	if (channel > 1)
	{
		channel--;
	}
	else
		channel = maxchannel;
}

void Tv::settings() const
{
	using std::cout;
	using std::endl;
	cout << "TV is " << (state == Off ? "Off" : "On") << endl;
	if (state == On)
	{
		cout << "Volume serring = " << volume << endl;
		cout << "Channel setting = " << channel << endl;
		cout << "Mode = " << (mode == Antenna ? "antenna" : "cable") << endl;
		cout << "Input = " << (input == TV ? "TV" : "DVD") << endl;
	}
}

bool Remote::volup(Tv& t)
{
	return t.valup();
}

bool Remote::voldown(Tv& t)
{
	return t.valdown();
}

void Remote::onoff(Tv& t)
{
	t.onoff();
}

void Remote::chanup(Tv& t)
{
	t.chanup();
}

void Remote::chandown(Tv& t)
{
	t.chandown();
}

void Remote::set_chan(Tv& t, int c)
{
	t.channel = c; //作为友元类访问私有变量
}

void Remote::set_mode(Tv& t)
{
	t.set_mode();
}

void Remote::set_input(Tv& t)
{
	t.set_input();
}


```

共同通友元,需要使用友元的另一种情况是，函数需要访问两个类的私有数据。所以可以将函数作为两个类的友元函数。

```c++
class Analyzer;
class Probe
{
	friend void aync(Analyzer& a, const Probe& p);
	friend void aync(Probe& p, const Analyzer& a);
};

class Analyzer
{
	friend void aync(Analyzer& a, const Probe& p);
	friend void aync(Probe& p, const Analyzer& a);
};

void aync(Analyzer& a, const Probe& p)
{
}

void aync(Probe& p, const Analyzer& a)
{
}

```

## 嵌套类
在c++中可以将一个类的声明放到另一个类中。在另一类中声明的类被称为嵌套类。

```c++
class Queue{
	class Node{
		pulbic:
			Item item;
			Node * next;
	}
}

Queue::Node node;
```
嵌套类的访问权限，在私有部分声明，就只能这个类访问，在保护部分声明派生类可以访问，在公共部分声明外部类也可以访问。

## 异常
程序有时候会遇到运行阶段错误，导致程序无法正常的运行下去。通常，程序员都会试图预防这种意外情况。但遇到这种情况应该怎么处理呢？

* 调用abort():

```c++
double hmean(double a, double b)
{
	if(a == -b){
		std::cout<<"untenable arguments to hmean!" << std::endl;
		std::abort()；
	}
	return 2.0 * a * b / (a + b);
}
```

* 返回错误码：这种方式比终止程序更加的灵活，通过返回true或者false来进行错误控制。

```c++
bool hmean(double a, double b, double & result)
{
	if(a == -b){
		std::cout<<"untenable arguments to hmean!" << std::endl;
		result = DBL_MAX;
		return false;
	}
	result = 2.0 * a * b / (a + b);
	return true;
}
```

* 异常机制

接下来介绍一下使用异常机制来进行异常控制。异常提供了将控制权从程序的一个部分传递到另一个部分的途径。对异常的处理有3个组成部分：

* 引发异常；
* 使用处理程序捕获异常；
* 使用try块。

程序在出现问题的时候将引发异常。例如，可以修改程序清单hmean,使他引发异常，而不是调用abort()函数。throw语句实际上是调转，即命令程序跳转到另一条语句。throw关键字便是引发异常，紧随其后的值指出了异常的特征。

程序使用异常处理程序来捕获异常，异常处理程序位于要处理问题的程序中。catch关键字表示捕获异常。处理程序以关键字catch开头，随后是位于括号中的类型声明，它指出了异常处理程序要响应的异常类型；然后是一个花括号括起来的代码， 指出要采用的措施。catch关键字和异常类型用作标签，指出当异常被引发时，程序应跳转到这个位置执行。异常处理程序也被称为catch块。

try块标识其中特定的异常可能被激活的代码块，他后面跟一个或多个catch块。try块是由关键字try指示的，关键字try的后面是一个花括号括起来的代码块，表明需要注意这些代码引发的异常。

```c++
#include<iostream>

using namespace std;
double hmean(double a, double b);
int main()
{
	double x, y, z;
	cout << "enter two number: ";
	while (cin >> x >> y) {
		try {
			z = hmean(x, y);
		}
		catch (const char* s) {
			cout << s << endl;
			cout << "Enterr a new pair numbers: ";
			continue;
		}
		cout << "Harmonic mean if " << x << "and " << y << "is " << z << endl;
		cout << "Enter next set of numbers <q to quit>: ";
	}
	cout << "Bye!\n";
	return 0;
}

double hmean(double a, double b) {
	if (a == -b)
	{
		throw " bad hmean() arguments: a == -b not allowed";
	}
	return 2.0 * a * b / (a + b);
}
```

* 讲对象作为异常类型

引发的异常函数的时候传递一个对象，这样做的重要优点是可以使用不同的异常类型来区分不同函数在不同情况下引发的异常。

```c++
class bad_hmean{

}

class badgmean{

}

try {}
catch(bad_hmean &bh){}
catch(bad_gmean &bg){}
//如果函数hmean引发异常引发bad_hmean异常，第一个catch块捕获异常。如果gmean()引发异常，异常将会跳过第一个catch块，被第二个catch块捕获。
```

如果又一个异常类继承层次结构，应该这样排列catch块：将捕获位于层次最下面的异常类的catch块语句放在最前面，将捕获基类异常的catch语句放在最后面。

## exception类
在c++中可以通过继承exception来一起捕获这些异常。

```C++
#include <exception>
class bad_hmean:public std::exception
{
	public:
	const char * what(){ return ""bad argument to hmean();}
};

cladd bad_gmean:public std::exception
{
	public:
		const char * what() { return "bad arguments to gmean()";}
}

try{
}
catch(std::exception &e)
{
cout<< e.what() << endl;
}
```