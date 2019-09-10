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
4. 死锁
5. unique_lock
6. 条件变量
7. atomic
8. future和promise


> Study hard, work hard.


## 与多线程相关的头文件
c++11新标准中引入四个头文件来支持多线程编程，它们分别是atomic,thread,mutex,condition_varible和future。

* atomic:该文件主要声明了两个类,sdt::atomic和std::atomic_flag,另外还声明了一套c风格的原子类型和C兼容的原子操作的函数。
* thread:该文件主要声明了std::thread类，另外std::this_thread命名空间也在改头文件中。
* mutex:改头文件主要是声明了与互斥量(mutex)相关的类，包括std::mutex系列类，std::lock_guard,std::unique_lock，以及其他的类型函数。
* condition_variable:该文件主要声明了与条件变量相关的类，包括std::condition_variable和condition_variable_any。
* future:改头文件中主要声明了std::promise,std::package_task两个Provider类，以及std::future和std::shared_future两个Future类，另外还有一些与之相关的类型和函数，std::async()函数声明在此有文件中。


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

## 死锁
死锁的条件：
1. 资源互斥，某个资源在某一时刻只能被一个线程持有。
2. 持有一个以上的互斥资源的线程在等待其它进程持有的互斥资源；
3. 不可抢占，只有在某互斥资源的持有线程释放了该资源之后，其它线程才能去持有该资源。
4. 环形等待，两个或者两个以上的线程各自持有某些互斥资源，并且各自等待其它线程所持有的的互斥资源。

在一些复杂的并行编程场景，如何避免死锁是一个很重要的话题，在实践中，当我们看到有两个锁嵌套加锁的时候就要特别提高警惕，它极有可能满足了条件 2 或者 4。

如果你讲某个mutex上锁了，却一直不释放，另一个线程访问该锁保护资源的时候就会发生死锁，这种情况可以使用local_guard保证析构的时候能够释放锁，但是当一个操作需要使用两个互斥元的时候，仅仅使用lock_guard并不能保证不会发生死锁，如下面的例子：

```c++
#include <iostream>
#include <thread>
#include <string>
#include <mutex>
#include <fstream>
using namespace std;

class LogFile {
    std::mutex _mu;
    std::mutex _mu2;
    ofstream f;
public:
    LogFile() {
        f.open("log.txt");
    }
    ~LogFile() {
        f.close();
    }
    void shared_print(string msg, int id) {
        std::lock_guard<std::mutex> guard(_mu);
        std::lock_guard<std::mutex> guard2(_mu2);
        f << msg << id << endl;
        cout << msg << id << endl;
    }
    void shared_print2(string msg, int id) {
        std::lock_guard<std::mutex> guard(_mu2);
        std::lock_guard<std::mutex> guard2(_mu);
        f << msg << id << endl;
        cout << msg << id << endl;
    }
};

void function_1(LogFile& log) {
    for(int i=0; i>-100; i--)
        log.shared_print2(string("From t1: "), i);
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
运行之后你就会发现程序卡住，这就是发生了死锁。
```c++
Thread A              Thread B
_mu.lock()          _mu2.lock()
   //死锁               //死锁
_mu2.lock()         _mu.lock()

if(&_mu < &_mu2){
    _mu.lock();
    _mu2.unlock();
}
else {
    _mu2.lock();
    _mu.lock();
}

std::lock(_mu, _mu2);
std::lock_guard<std::mutex> guard(_mu2, std::adopt_lock);
std::lock_guard<std::mutex> guard2(_mu, std::adopt_lock);
```

解决方法：
1. 可以比较mutex的地址，每次都先锁地址小的。
2. 使用层次锁，将互斥锁包装一下，给锁定义一个层次的属性，每次按层次由高到低的顺序上锁。

这两种办法其实都是严格规定上锁顺序，只不过实现方式不同。


c++标准库中提供了std::lock()函数，能够保证将多个互斥锁同时上锁。同时，lock_guard也需要做修改，因为互斥锁已经被上锁了，那么lock_guard构造的时候不应该上锁，只是需要在析构的时候释放锁就行了，使用std::adopt_lock表示无需上锁。

关于锁，总结一下：
1. 建议尽量同时只对一个互斥上锁。
2. 不要在互斥锁保护的区域使用用户自定义的代码，因为用户的代码可能操作了其他的互斥锁。
3. 如果相同时对多个互斥锁上锁，要使用std::lock().
4. 给锁定义顺序(使用层次锁，或者比较地址等），每次以同样的顺序进行上锁。

## unique_lock
互斥锁保证了线程间的同步，但是却将并行操作变成了串行操作，这对性能有很大的影响，所以我们要尽可能的减小锁定的区域，也就是使用细粒度锁。

这一点lock_guard做的不好，不够灵活，lock_guard只能保证在析构的时候执行解锁操作，lock_guard本身并没有提供加锁和解锁的接口，但是有些时候会有这种需求。看下面的例子。

```c++
class LogFile {
    std::mutex _mu;
    ofstream f;
public:
    LogFile() {
        f.open("log.txt");
    }
    ~LogFile() {
        f.close();
    }
    void shared_print(string msg, int id) {
        {
            std::lock_guard<std::mutex> guard(_mu);
            //do something 1
        }
        //do something 2
        {
            std::lock_guard<std::mutex> guard(_mu);
            // do something 3
            f << msg << id << endl;
            cout << msg << id << endl;
        }
    }

};
```
上面的代码中，一个函数内部有两段代码需要进行保护，这个时候使用lock_guard就需要创建两个局部对象来管理同一个互斥锁（其实也可以只创建一个，但是锁的力度太大，效率不行），修改方法是使用unique_lock。它提供了lock()和unlock()接口，能记录现在处于上锁还是没上锁状态，在析构的时候，会根据当前状态来决定是否要进行解锁（lock_guard就一定会解锁）。上面的代码修改如下：

```c++
class LogFile {
    std::mutex _mu;
    ofstream f;
public:
    LogFile() {
        f.open("log.txt");
    }
    ~LogFile() {
        f.close();
    }
    void shared_print(string msg, int id) {

        std::unique_lock<std::mutex> guard(_mu);
        //do something 1
        guard.unlock(); //临时解锁

        //do something 2

        guard.lock(); //继续上锁
        // do something 3
        f << msg << id << endl;
        cout << msg << id << endl;
        // 结束时析构guard会临时解锁
        // 这句话可要可不要，不写，析构的时候也会自动执行
        // guard.ulock();
    }

};
```

上面的代码可以看到，在无需加锁的操作时，可以先临时释放锁，然后需要继续保护的时候，可以继续上锁，这样就无需重复的实例化lock_guard对象，还能减少锁的区域。同样，可以使用std::defer_lock设置初始化的时候不进行默认的上锁操作：

```c++
void shared_print(string msg, int id) {
    std::unique_lock<std::mutex> guard(_mu, std::defer_lock);
    //do something 1

    guard.lock();
    // do something protected
    guard.unlock(); //临时解锁

    //do something 2

    guard.lock(); //继续上锁
    // do something 3
    f << msg << id << endl;
    cout << msg << id << endl;
    // 结束时析构guard会临时解锁
}
```
这样使用起来就比lock_guard更加灵活！然后这也是有代价的，因为它内部需要维护锁的状态，所以效率要比lock_guard低一点，在lock_guard能解决问题的时候，就是用lock_guard，反之，使用unique_lock。

后面在学习条件变量的时候，还会有unique_lock的用武之地。

另外，请注意，unique_lock和lock_guard都不能复制，lock_guard不能移动，但是unique_lock可以！
```c++
// unique_lock 可以移动，不能复制
std::unique_lock<std::mutex> guard1(_mu);
std::unique_lock<std::mutex> guard2 = guard1;  // error
std::unique_lock<std::mutex> guard2 = std::move(guard1); // ok

// lock_guard 不能移动，不能复制
std::lock_guard<std::mutex> guard1(_mu);
std::lock_guard<std::mutex> guard2 = guard1;  // error
std::lock_guard<std::mutex> guard2 = std::move(guard1); // error
```

## 条件变量
互斥锁std::mutex是一种常见的线程间同步的手段，但是有些情况下不太高效。

假设想实现一个简单的消费者生产者模型，一个线程往队列里面放入数据，一个线程从队列中取数据，取数据前需要判断一下队列中确实有数据，由于这个队列是线程间共享的，所以，需要使用互斥锁进行保护，一个线程在往队列添加数据的时候，另一个线程不能取，反之亦然。用互斥锁实现如下：
```c++
#include <iostream>
#include <deque>
#include <thread>
#include <mutex>

std::deque<int> q;
std::mutex mu;

void function_1() {
    int count = 10;
    while (count > 0) {
        std::unique_lock<std::mutex> locker(mu);
        q.push_front(count);
        locker.unlock();
        std::this_thread::sleep_for(std::chrono::seconds(1));
        count--;
    }
}

void function_2() {
    int data = 0;
    while ( data != 1) {
        std::unique_lock<std::mutex> locker(mu);
        if (!q.empty()) {
            data = q.back();
            q.pop_back();
            locker.unlock();
            std::cout << "t2 got a value from t1: " << data << std::endl;
        } else {
            locker.unlock();
        }
    }
}
int main() {
    std::thread t1(function_1);
    std::thread t2(function_2);
    t1.join();
    t2.join();
    return 0;
}

//输出结果
//t2 got a value from t1: 10
//t2 got a value from t1: 9
//t2 got a value from t1: 8
//t2 got a value from t1: 7
//t2 got a value from t1: 6
//t2 got a value from t1: 5
//t2 got a value from t1: 4
//t2 got a value from t1: 3
//t2 got a value from t1: 2
//t2 got a value from t1: 1

void function_2() {
    int data = 0;
    while ( data != 1) {
        std::unique_lock<std::mutex> locker(mu);
        if (!q.empty()) {
            data = q.back();
            q.pop_back();
            locker.unlock();
            std::cout << "t2 got a value from t1: " << data << std::endl;
        } else {
            locker.unlock();
            std::this_thread::sleep_for(std::chrono::milliseconds(500));
        }
    }
}
```

可以看到，互斥锁其实可以完成这个任务，但是却存在着性能问题。

首先，function_1函数是生产者，在生产过程中，std::this_thread::sleep_for(std::chrono::seconds(1));表示延时1s，所以这个生产的过程是很慢的；function_2函数是消费者，存在着一个while循环，只有在接收到表示结束的数据的时候，才会停止，每次循环内部，都是先加锁，判断队列不空，然后就取出一个数，最后解锁。所以说，在1s内，做了很多无用功！这样的话，CPU占用率会很高，可能达到100%（单核）.

解决办法之一是给消费者也加一个小延时，如果一次判断后，发现队列是空的，就惩罚一下自己，延时500ms，这样可以减小CPU的占用率。

然后困难之处在于，如何确定这个延时时间呢，假如生产者生产的很快，消费者却延时500ms，也不是很好，如果生产者生产的更慢，那么消费者延时500ms，还是不必要的占用了CPU。

这就引出了条件变量（condition variable）,c++11中提供了#include <condition_variable>头文件，其中的std::condition_variable可以和std::mutex结合一起使用，其中有两个重要的接口，notify_one()和wait()，wait()可以让线程陷入休眠状态，在消费者生产者模型中，如果生产者发现队列中没有东西，就可以让自己休眠，但是不能一直不干活啊，notify_one()就是唤醒处于wait中的其中一个条件变量（可能当时有很多条件变量都处于wait状态）。那什么时刻使用notify_one()比较好呢，当然是在生产者往队列中放数据的时候了，队列中有数据，就可以赶紧叫醒等待中的线程起来干活了。

使用条件变量修改后如下：
```c++
#include <iostream>
#include <deque>
#include <thread>
#include <mutex>
#include <condition_variable>

std::deque<int> q;
std::mutex mu;
std::condition_variable cond;

void function_1() {
    int count = 10;
    while (count > 0) {
        std::unique_lock<std::mutex> locker(mu);
        q.push_front(count);
        locker.unlock();
        cond.notify_one();  // Notify one waiting thread, if there is one.
        std::this_thread::sleep_for(std::chrono::seconds(1));
        count--;
    }
}

void function_2() {
    int data = 0;
    while ( data != 1) {
        std::unique_lock<std::mutex> locker(mu);
        while(q.empty())
            cond.wait(locker); // Unlock mu and wait to be notified
        data = q.back();
        q.pop_back();
        locker.unlock();
        std::cout << "t2 got a value from t1: " << data << std::endl;
    }
}
int main() {
    std::thread t1(function_1);
    std::thread t2(function_2);
    t1.join();
    t2.join();
    return 0;
}
```

上面的代码有三个注意事项：
1. 在function_2中，在判断队列是否为空的时候，使用的是while(q.empty()),而不是if(q.empty())，这是因为wait()从阻塞到返回，不一定就是由于notify_one()函数造成的，还有可能由于系统的不确定原因唤醒（可能和条件变量的实现机制有关），这个的时机和频率都是不确定的，被称作伪唤醒，如果在错误的时候被唤醒了，执行后面的语句就会错误，所以需要再次判断队列是否为空，如果还是为空，就继续wait()阻塞。
2. 在管理互斥锁的时候，使用的是std::unique_lock而不是std::lock_guard，而且事实上也不能使用std::lock_guard，这需要先解释下wait()函数所做的事情。可以看到，在wait()函数之前，使用互斥锁保护了，如果wait的时候什么都没做，岂不是一直持有互斥锁？那生产者也会一直卡住，不能够将数据放入队列中了。所以，wait()函数会先调用互斥锁的unlock()函数，然后再将自己睡眠，在被唤醒后，又会继续持有锁，保护后面的队列操作。而lock_guard没有lock和unlock接口，而unique_lock提供了。这就是必须使用unique_lock的原因。
3. 使用细粒度锁，尽量减小锁的范围，在notify_one()的时候，不需要处于互斥锁的保护范围内，所以在唤醒条件变量之前可以将锁unlock()。

还可以将cond.wait(locker);换一种写法，wait()的第二个参数可以传入一个函数表示检查条件，这里使用lambda函数最为简单，如果这个函数返回的是true，wait()函数不会阻塞会直接返回，如果这个函数返回的是false，wait()函数就会阻塞着等待唤醒，如果被伪唤醒，会继续判断函数返回值。

```c++
void function_2() {
    int data = 0;
    while ( data != 1) {
        std::unique_lock<std::mutex> locker(mu);
        cond.wait(locker, [](){ return !q.empty();} );  // Unlock mu and wait to be notified
        data = q.back();
        q.pop_back();
        locker.unlock();
        std::cout << "t2 got a value from t1: " << data << std::endl;
    }
}
```

除了notify_one()函数，c++还提供了notify_all()函数，可以同时唤醒所有处于wait状态的条件变量。

## atomic ##
atomic：原子类型是对数据的封装，可以防止数据竞争，同步多线程间的内存访问。头文件主要包含两个类：atomic和atomic_flag。

```c++
#include <iostream>       // std::cout
#include <atomic>         // std::atomic, std::atomic_flag,   ATOMIC_FLAG_INIT
#include <thread>         // std::thread, std::this_thread::yield
#include <vector>         // std::vector

std::atomic<bool> ready (false);
std::atomic_flag winner = ATOMIC_FLAG_INIT;//静态存储时间的原子变量的常量初始化(宏)

void count1m (int id) {
while (!ready) { std::this_thread::yield(); }      // wait for the ready signal
for (volatile int i=0; i<1000000; ++i) {}          // go!, count to 1 million
if (!winner.test_and_set()) { std::cout << "thread #" << id << " won!\n"; }//原子地将flag设置为true并返回其先前的值 (函数)
};

int main ()
{
std::vector<std::thread> threads;
std::cout << "spawning 10 threads that count to 1 million...\n";
for (int i=1; i<=10; ++i) threads.push_back(std::thread(count1m,i));
ready = true;
for (auto& th : threads) th.join();

return 0;
}
```
1. 将原子对象放在未初始化的状态中。一个未初始化的原子对象可以通过atomicinit来初始化。
2. 用desired初始化对象。初始化不是原子性的。
3. 原子变量不是可复制的。

```c++
#include <iostream>       // std::cout
#include <atomic>         // std::atomic
#include <thread>         // std::thread, std::this_thread::yield
#include <string>
#include <mutex>

std::atomic<int> foo(0);
std::atomic<long> foo1(0);
std::mutex mu;

void printString(std::string str, long foo1) {
	std::unique_lock<std::mutex> guard(mu);
	std::cout << str << foo1 << '\n';
}

void set_foo(int x) {
	foo = x;
	while (foo1 != 100)
	{
		foo1++;
		printString("set_foo foo1: ", foo1);
	}
}

void print_foo() {
	while (foo == 0) {             // wait while foo=0
		std::this_thread::yield();
	}
	printString("foo: ", foo1);
	while (foo1 != 100)
	{
		foo1++;
		printString("print_foo foo1:  ", foo1);
	}
}

int main()
{
	std::thread first(print_foo);
	std::thread second(set_foo, 10);
	first.join();
	second.join();
	return 0;
}
```

## future和promise
future和promise的作用是在不同的线程之间传递数据。使用指针也可以完成数据的传递，但是指针非常危险，因为互斥量不能阻止指针的访问；而且指针的方式传递的数据是固定的，如果改变数据类型，那么还需要更改有关的接口，比较麻烦了promise支持泛型的操作，更加方便编程处理。

假设线程1需要线程2的数据，那么组合使用的方式如下：
1. 线程1初始化一个promise对象和一个future对象，promise传递给线程2，相当于线程2对线程1的一个承诺；future相当于一个接受一个承诺，用来获取未来线程2传递的值。
2. 线程2获取到promise后，需要对这个promise传递有关的数据，之后线程1的future就可以获取数据了。
3. 如果线程1想要获取数据，而线程2未给出数据，则线程1阻塞，直到线程2的数据到达。

```c++
#include <iostream>
#include <functional>
#include <future>
#include <thread>
#include <chrono>
#include <cstdlib>

void thread_set_promise(std::promise<int>& promiseObj) {
    std::cout << "In a thread, making data...\n";
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    promiseObj.set_value(35);
    std::cout << "Finished\n";
}

int main() {
    std::promise<int> promiseObj;
    std::future<int> futureObj = promiseObj.get_future();
    std::thread t(&thread_set_promise, std::ref(promiseObj));
    std::cout << futureObj.get() << std::endl;
    t.join();

    system("pause");
    return 0;
}
```