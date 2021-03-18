---
layout:     post
title:      智能指针
subtitle:   c++
date:       2019-8-29
author:     Jow
header-img: img/post-bg-infinity.jpg
catalog: 	 true 
tags:
    - DDT

---

### 目录
1. 智能指针的种类
2. 使用时需要注意的事项
3. weal_ptr


> Study hard, work hard.


## 智能指针的种类
一共有四种智能指针，包含auto_ptr,unique_ptr,shared_ptr以及weak_ptr.创建智能指针对象，必须包含文件memory，该文件模板定义。然后使用模板语法来实例化所需类型的指针。
```c++
template<class X> 
class auto_ptr{
public:
	explicit auto_ptr(X* p =0) throw();
}

auto_ptr<double> pd(new double);
auto_ptr<string> ps(new string);
unique_ptr<double> upd(new double);
shared_ptr<double> spd(new double);
```

## 使用的时候需要注意事项
```c++
auto_ptr<string> ps(new string("I reigned lonely as a cloud."));
auto_ptr<string> vocation;
vocation = ps;
auto_ptr<string> films[5] = {
	auto_ptr<string>(new string("Fowl Balls1"),
	auto_ptr<string>(new string("Fowl Balls2"),
	auto_ptr<string>(new string("Fowl Balls3"),
	auto_ptr<string>(new string("Fowl Balls4"),
	auto_ptr<string>(new string("Fowl Balls5"),
}
auto_ptr<string> pwin;
pwin = films[2];//films[2] loses owership
```
上面的代码将会导致一个问题，两个指针将指向通过一个string对象。这是不能接受的，因为程序试图删除一个对象两次。

解决方法：
* 定义赋值运算符，使之执行深复制。
* 建立所有权概念，对于特定的对象，只用一个智能指针可以拥有它，这样只用拥有对象的智能指针的构造函数会删除该对象。然后让赋值操作转让所有权。这就是用于auto_ptr和unique_ptr的策略，但unique_ptr的策略更严格。
* 创建智能更高的指针，跟踪引用特定对象的智能指针数。这称为引用计数，赋值时引用加一，删除时减一，这是shared_ptr的策略。

auto_ptr存在的问题：

将一个智能指针对象赋值给另一个智能指针对象会导致前者的对象所有权转让给后者，导致前者不能引用该对象，就会导致空指针。就像上面的filems[2]智能指针的对象。

解决方案:
1. 使用shared_ptr，通过引用计数来实现共同所有。
2. 使用unique_ptr,通过在编译阶段报错来避免程序崩溃。

## weak_ptr
weak_ptr的用途：
1. 解决空悬指针问题
2. 解决循环引用问题

**weak_ptr没有*操作和->操作**

weak_ptr是不控制所指对象生存周期的智能指针，它指向又shared_ptr管理的对象。将一个weak_ptr绑定到一个shared_ptr不会改变shared_ptr的计数器。一旦最后一个指向对象的shared_ptr被销毁，对象就会被释放，即使有weak_ptr指向这个对象，对象也会被销毁。
1. weak_ptr<T> w:空weak_ptr，可以指向类型为*T的对象
2. weak_ptr<T> w(sp):与shared_sp sp指向相同的对象weak_ptr。
3. w = p:p可以是一个shared_ptr或一个weak_ptr。然后赋值后w指向p所指的对象。
4. w.reset():将w置为空
5. w.use_count:与w共享对象的shared_ptr的数量
6. w.expired()：w.use_count()为0，返回true，否则返回false
7. w.lock():如果expired为true，返回一个空shared_ptr,否则返回一个指向w所指对象的shared_ptr.

```c++
#include<iostream>
#include<memory>
#include<vector>
using namespace std;
class Test {
private:
	int data;
public:
	Test(int d = 0) :data(d) { cout << "new" << data << endl; }
	~Test() { cout << "del" << data << endl; }
	void func() { cout << "func" << endl; }
};

class teacher {
public:
	teacher() { cout << "teacher()" << endl; }
	~teacher() { cout << "del teacher" << endl; }
	shared_ptr<student> stu;
};

class student {
public:
	student() { cout << "student()" << endl; }
	~student() { cout << "del student" << endl; }
	//如果换成shared_ptr<teacher> tea;就会形成循环引用，导致内存泄漏    
	weak_ptr<teacher> tea;
};

int main()
{
	//test3 lock使用  
	//shared_ptr<int> sptr;
	//sptr.reset(new int);
	//*sptr = 10;
	//weak_ptr<int> weak1 = sptr;
	//sptr.reset(new int);
	//*sptr = 5;
	//weak_ptr<int> weak2 = sptr;
	//// weak1 is expired!                                                          
	//if (auto tmp = weak1.lock())
	//	cout << *tmp << '\n';
	//else
	//	cout << "weak1 is expired\n";
	//// weak2 points to new data (5)                                               
	//if (auto tmp = weak2.lock())
	//	cout << *tmp << '\n';
	//else
	//	cout << "weak2 is expired\n";
	//test4 循环引用，导致即使是智能指针也不能释放内存                            
 //用weak_ptr解决了循环引用，导致的内存不能释放的问题                          
	shared_ptr<teacher> tptr(new teacher);//计数器1                               
	shared_ptr<student> sptr(new student);//计数器1                               
	tptr->stu = sptr;//sptr的计数器2                                              
	sptr->tea = tptr;//不增加tptr的引用计数,因为tea是weak指针                     
	cout << tptr.use_count() << endl;//1                                          
	cout << sptr.use_count() << endl;//2                                          

	return 0;

}
```

会导致内存泄漏的原因分析：
1. 出了函数作用域时，由于析构和构造的顺序是相反的，会先析构共享智能指针sptr，资源B的引用计数就变成了1；接下来继续析构共享智能指针tptr，资源A的引用计数也变成了1。由于tptr和sptr的引用计数都不为1，说明还有共享智能指针在使用着它们，所以不会调用资源的析构函数！
2. 这样就陷入了死循环。