---
layout:     post
title:      thread
subtitle:   c++
date:       2019-9-9
author:     Jow
header-img: img/post-bg-infinity.jpg
catalog: 	 true 
tags:
    - C++

---

### 目录
1. 与多线程相关的头文件
2. c++ thread构造函数


> Study hard, work hard.


## 与多线程相关的头文件
c++11新标准中引入四个头文件来支持多线程编程，它们分别是<atomic>,<thread>,<mutex>,<condition_varible>和<future>。

* <atomic>:该文件主要声明了两个类,sdt::atomic和std::atomic_flag,另外还声明了一套c风格的原子类型和C兼容的原子操作的函数。
* <thread>:该文件主要声明了std::thread类，另外std::this_thread命名空间也在改头文件中。
* <mutex>:改头文件主要是声明了与互斥量(mutex)相关的类，包括std::mutex系列类，std::lock_guard,std::unique_lock，以及其他的类型函数。
* <condition_variable>:该文件主要声明了与条件变量相关的类，包括std::condition_variable和condition_variable_any。
* <future>:改头文件中主要声明了std::promise,std::package_task两个Provider类，以及std::future和std::shared_future两个Future类，另外还有一些与之相关的类型和函数，std::async()函数声明在此有文件中。


```c++
#include<iostream>
#include<thread>

use namespace std;
void th_func(){
	cout<< "this is my firdt thread." << endl;
}

int main(){
	thread t(th_func);
	t.join();
	
	return 0;
}
```

* 首先，构建一个std::thread对象t，构造的时候传递了一个参数，这个参数是一个函数，这个函数就是这个线程的入口函数，函数执行完了，整个线程也就执行完了。
* 线程创建成功后，就会立即启动，并没有一个类似start的函数来显式的启动线程。
* 一旦线程开始运行，就需要显式的决定是要等待它完成(join),或者分离它让它自行运行(detach).当然要在thread对象销毁之前作出这个决定。在例子中，t是一个栈上的对象，所以在main执行完之前要作出决定。
* 例子中选择join，主线程就会一直阻塞，知道子线程完成，join()函数的另一个任务就是回收改线程使用的资源。

线程对象和对象内存管理的线程的声明周期并不一样，如果线程执行的快，可能内部的线程已经结束了，但是线程对象还活着，也有可能线程对象已经被析构了，内部的线程还在运行。

假设t线程是一个执行很慢的线程，主线程并不想等它结束结束整个人物，直接删除join是不行的，程序会终止(析构t的时候会调用std:terminate,程序会打印terminate called without an active exception).

与之对应，我们可以调用t.detach()，从而将t线程放在后台运行，所有的控制权被转交给c++运行时库，以确保与线程相关的资源在线程退出后能被正确的回收。这种分离的线程被才称为守护线程。线程被分离后还是能在后台运行，只是对象被析构了，主线程不能够和对象名这个线程进行通信。


```c++
#include<thread>
#include<iostream>
using namespace std;

void func1() {
	this_thread::sleep_for(chrono::microseconds(500));
	cout << "I'm func1()." << endl;
}

void test() {
	thread t(func1);
	t.detach();
	if (t.joinable())
	{
		t.join();
	}
	cout << "test is finish." << endl;
}

int main()
{
	test();
	this_thread::sleep_for(chrono::microseconds(1000));
	cout << "Main is finish." << endl;
	return 0;

}
```
1. 由于线程入口函数内部有500ms的延时，所以在还没有打印的时候，test()已经执行完成了，t已经被吸够了，但是它负责的那个线程还是能够运行，这就是detach()的作用。
2. 如果去掉main函数中的1s延时，就会发现什么都没有，因为主线程执行的太快了，整个程序已经结束了，那个后台线程被c++运行时库回收了。
3. detach换成join，test函数就是在线程执行完成之后结束。

一个线程被分离了，就不能够被join了。如果非要调用，程序就会崩溃，可以使用joinable来判断一个线程对象能否调用join.

**可被jionable的thread对象必须在它们销毁之前被主线程join或者将其detached**

## c++ thread构造函数
1. 默认构造函数，创建一个空的thread对象。
2. 初始化构造函数，创建一个thread对象，该thread对象可以被joinable，新产生的线程会调用fn函数，该函数的参数由args给出。
3. 拷贝构造函数(被禁用),意味着thread不可以被拷贝构造。
4. move构造函数，move构造函数，调用成功之后x不代表任何thread执行对象。


std::thread类的构造函数是使用可变板书模板实现的，也就是说，可以传递人一个参数，第一个参数的入口函数，而后面的若干个参数是该函数的参数。

第一个参数是可调用对象(Callable Objects):
* 函数指针
* 重载了operate()运算符的类对象，即仿函数
* lambda表达式(匿名函数)
* std::function

```c++
// 普通函数 无参
void function_1() {
}

// 普通函数 1个参数
void function_2(int i) {
}

// 普通函数 2个参数
void function_3(int i, std::string m) {
}

std::thread t1(function_1);
std::thread t2(function_2, 1);
std::thread t3(function_3, 1, "hello");

t1.join();
t2.join();
t3.join();
```
**在这里注意不能将重载函数作为线程的入口函数，会发生编译错误**

```c++
// 仿函数
class Fctor {
public:
    // 具有一个参数
    void operator() () {

    }
};
Fctor f;
std::thread t1(f);  
// std::thread t2(Fctor()); // 编译错误 
std::thread t3((Fctor())); // ok
std::thread t4{Fctor()}; // ok

//匿名函数
std::thread t1([](){
    std::cout << "hello" << std::endl;
});

std::thread t2([](std::string m){
    std::cout << "hello " << m << std::endl;
}, "world");

//std::function
class A{
public:
    void func1(){
    }

    void func2(int i){
    }
    void func3(int i, int j){
    }
};

A a;
std::function<void(void)> f1 = std::bind(&A::func1, &a);
std::function<void(void)> f2 = std::bind(&A::func2, &a, 1);
std::function<void(int)> f3 = std::bind(&A::func2, &a, std::placeholders::_1);
std::function<void(int)> f4 = std::bind(&A::func3, &a, 1, std::placeholders::_1);
std::function<void(int, int)> f5 = std::bind(&A::func3, &a, std::placeholders::_1, std::placeholders::_2);

std::thread t1(f1);
std::thread t2(f2);
std::thread t3(f3, 1);
std::thread t4(f4, 1);
std::thread t5(f5, 1, 2);
```

一个仿函数类生成的对象，使用起来就像一个函数一样，比如上面的对象f，当使用f()时就调用operator()运算符。所以也可以让它成为线程类的第一个参数，如果这个仿函数有参数，同样的可以写在线程类的后几个参数上。

而t2之所以编译错误，是因为编译器并没有将Fctor()解释为一个临时对象，而是将其解释为一个函数声明，编译器认为你声明了一个函数，这个函数不接受参数，同时返回一个Factor对象。解决办法就是在Factor()外包一层小括号()，或者在调用std::thread的构造函数时使用{}，这是c++11中的新的同意初始化语法。

但是，如果重载的operator()运算符有参数，就不会发生上面的错误。

先提出一个问题：如果线程入口函数的的参数是引用类型，在线程内部修改该变量，主线程的变量会改变吗？

