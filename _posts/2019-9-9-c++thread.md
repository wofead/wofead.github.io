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
3. 竞争条件与互斥锁


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
```c++
#include <iostream>
#include <thread>
#include <string>

// 仿函数
class Fctor {
public:
    // 具有一个参数 是引用
    void operator() (std::string& msg) {
        msg = "wolrd";
    }
};



int main() {
    Fctor f;
    std::string m = "hello";
	std::thread t1(f, std::ref(m));
//    std::thread t1(f, m);vs2019会报错

    t1.join();
    std::cout << m << std::endl;
    return 0;
}

// vs下： 最终是："hello"
// g++编译器： 编译报错
```

报错原因：std::thread类，内部也有若干个变量，当使用构造函数创建对象的时候，是将参数先赋值给这些变量，所以这些变量只是个副本，然后在线程启动并调用线程入口函数时，传递的参数只是这些副本，所以内部怎么操作都是改变副本，而不影响外面的变量。g++可能是比较严格，这种写法可能会导致程序发生严重的错误，索性禁止了。

而如果可以想真正传引用，可以在调用线程类构造函数的时候，用std::ref()包装一下。

同理，构造函数的第一个参数是可调用对象，默认情况下其实传递的还是一个副本。

```c++
#include <iostream>
#include <thread>
#include <string>

class A {
public:
    void f(int x, char c) {}
    int g(double x) {return 0;}
    int operator()(int N) {return 0;}
};

void foo(int x) {}

int main() {
    A a;
    std::thread t1(a, 6); // 1. 调用的是 copy_of_a()
    std::thread t2(std::ref(a), 6); // 2. a()
    std::thread t3(A(), 6); // 3. 调用的是 临时对象 temp_a()
    std::thread t4(&A::f, a, 8, 'w'); // 4. 调用的是 copy_of_a.f()
    std::thread t5(&A::f, &a, 8, 'w'); //5.  调用的是 a.f()
    std::thread t6(std::move(a), 6); // 6. 调用的是 a.f(), a不能够再被使用了
    t1.join();
    t2.join();
    t3.join();
    t4.join();
    t5.join();
    t6.join();
    return 0;
}
```

**线程对象只能移动不可复制**

线程对象之间是不能复制的，只能移动，移动的意思是，将线程的所有权在std::thread实例间进行转移。

```c++
#include <iostream>
#include <thread>
#include <string>

void some_function(std::string str) {
	std::cout << "some function." << str << std::endl;
}
void some_other_function(std::string str) {
	std::cout << "some other function." << str << std::endl;
}

void foo(int x) {}

int main() {
	
	std::thread t1(some_function,"t11");
	// std::thread t2 = t1; // 编译错误
	std::thread t2 = std::move(t1); //只能移动 t1内部已经没有线程了
	t1 = std::thread(some_other_function,"t12"); // 临时对象赋值 默认就是移动操作
	std::thread t3;
	t3 = std::move(t2); // t2内部已经没有线程了
	//t1 = std::move(t3); // 程序将会终止，因为t1内部已经有一个线程在管理了
	t1.join();
	//t2.join(); //t2里面已经没有线程了，所以会报错
	t3.join();

	return 0;
}
```

## 竞争条件与互斥锁
并发代码中最常见的错误之一就是竞争条件。而其中最常见的就是数据竞争，从整体来看，所有的线程之间共享数据的问题，都是修改数据导致的，如果所有的共享数据都是只读的，就不会发生问题。但是一般数据都是要被修改的。

在c++中，常见的cout就是共享资源，如果在多个线程中同时执行cout，你就会发现输出很混乱：
```c++
#include <iostream>
#include <thread>
#include <string>
using namespace std;

// 普通函数 无参
void function_1() {
    for(int i=0; i>-100; i--)
        cout << "From t1: " << i << endl;
}

int main()
{
    std::thread t1(function_1);

    for(int i=0; i<100; i++)
        cout << "From main: " << i << endl;

    t1.join();
    return 0;
}
```
你有很大的几率发现打印会出现类似于From t1: From main: 64这样奇怪的打印结果。cout是基于流的，会先将你要打印的内容放入缓冲区，可能刚刚一个线程刚刚放入From t1:，另一个线程就执行了，导致输出变乱。而c语言中的printf不会发生这个问题。

解决方法就是对cout这个共享资源进行保护。在c++中，可以使用互斥锁std::mutex进行资源保护，共有两种操作：锁定和解锁。
```c++
#include <iostream>
#include <thread>
#include <string>
#include <mutex>
using namespace std;

std::mutex mu;
// 使用锁保护
void shared_print(string msg, int id) {
    mu.lock(); // 上锁
    cout << msg << id << endl;
    mu.unlock(); // 解锁
}


void function_1() {
    for(int i=0; i>-100; i--)
        shared_print(string("From t1: "), i);
}

int main()
{
    std::thread t1(function_1);

    for(int i=0; i<100; i++)
        shared_print(string("From main: "), i);

    t1.join();
    return 0;
}
```

这样会存在一个问题，如果锁了忘记解锁会怎么样，这就会导致线程阻塞。

解决这个问题也很简单，使用c++中常见的RAII技术，即获取资源即初始化(Resource Acquisition Is Initialization)技术，这是c++中管理资源的常用方式。简单的说就是在类的构造函数中创建资源，在析构函数中释放资源，因为就算发生了异常，c++也能保证类的析构函数能够执行。我们不需要自己写个类包装mutex，c++库已经提供了std::lock_guard类模板，使用方法如下：
```c++
void shared_print(string msg, int id) {
    //构造的时候帮忙上锁，析构的时候释放锁
    std::lock_guard<std::mutex> guard(mu);
    //mu.lock(); // 上锁
    cout << msg << id << endl;
    //mu.unlock(); // 解锁
}
```

上面的std::mutex互斥元是个全局变量，他是为shared_print()准备的，这个时候，我们最好将他们绑定在一起，比如说，可以封装成一个类。由于cout是个全局共享的变量，没法完全封装，就算你封装了，外面还是能够使用cout，并且不用通过锁。下面使用文件流举例：
```c++
#include <iostream>
#include <thread>
#include <string>
#include <mutex>
#include <fstream>
using namespace std;

std::mutex mu;
class LogFile {
    std::mutex m_mutex;
    ofstream f;
public:
    LogFile() {
        f.open("log.txt");
    }
    ~LogFile() {
        f.close();
    }
    void shared_print(string msg, int id) {
        std::lock_guard<std::mutex> guard(mu);
        f << msg << id << endl;
    }
// Never return f to the outside world
    ofstream& getStream() {
        return f;  //never do this !!!
    }
// Never pass f as an argument to user provided function
    void process(void fun(ostream&)) {
        fun(f);
    }
};

void function_1(LogFile& log) {
    for(int i=0; i>-100; i--)
        log.shared_print(string("From t1: "), i);
}

int main()
{
    LogFile log;
    std::thread t1(function_1, std::ref(log));

    for(int i=0; i<100; i++)
        log.shared_print(string("From main: "), i);

    t1.join();
    return 0;
}
```
上面的LogFile类封装了一个mutex和一个ofstream对象，然后shared_print函数在mutex的保护下，是线程安全的。使用的时候，先定义一个LogFile的实例log，主线程中直接使用，子线程中通过引用传递过去（也可以使用单例来实现）,这样就能保证资源被互斥锁保护着，外面没办法使用但是使用资源。

但是这个时候还是得小心了！用互斥元保护数据并不只是像上面那样保护每个函数，就能够完全的保证线程安全，如果将资源的指针或者引用不小心传递出来了，所有的保护都白费了！要记住一下两点：
1. 不要提供函数让用户获取资源。
2. 不要资源传递给用户的函数。