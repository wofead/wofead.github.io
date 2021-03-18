---
layout:     post
title:      lambda
subtitle:   c++
date:       2019-9-9
author:     Jow
header-img: img/post-bg-infinity.jpg
catalog: 	 true 
tags:
    - C++

---

### 目录
1. 匿名函数
2. std::function
3. std::bind


> Study hard, work hard.


## 匿名函数
c++11提供了对匿名函数的支持，称为Lambda函数(也叫Lambda表达式),Lambda表达式具体形式如下：
> \[capture\](parameters) ->return-type{body}
> \[capture\]{body}

如果没有参数圆括号和return可以省略。


```c++
[](int x, int y) { return x + y; } // 隐式返回类型
[](int& x) { ++x; }   // 没有return语句 -> lambda 函数的返回类型是'void'
[]() { ++global_x; }  // 没有参数,仅访问某个全局变量
[]{ ++global_x; }     // 与上一个相同,省略了()
[](int x, int y) -> int { int z = x + y; return z; }
```

Lambda函数可以引用在它之外声明的变量，这些变量的集合叫做闭包，闭包被定义在Lambda表达式声明中的[]内，这个机制允许这些变量被按值或引用补货，下面这些例子就是：

* []        //未定义变量.试图在Lambda内使用任何外部变量都是错误的.
* [x, &y]   //x 按值捕获, y 按引用捕获.
* [&]       //用到的任何外部变量都隐式按引用捕获
* [=]       //用到的任何外部变量都隐式按值捕获
* [&, x]    //x显式地按值捕获. 其它变量按引用捕获
* [=, &z]   //z按引用捕获. 其它变量按值捕获


```c++
std::vector<int> some_list;
int total = 0;
for (int i=0;i<5;++i) some_list.push_back(i);
std::for_each(begin(some_list), end(some_list), [&total](int x) 
{
    total += x;
});
//此例计算list中所有元素的总和. 变量total被存为lambda函数闭包的一部分. 因为它是栈变量(局部变量)total的引用,所以可以改变它的值.  

std::vector<int> some_list;
int total = 0;
int value = 5;
std::for_each(begin(some_list), end(some_list), [&, value, this](int x) 
{
total += x * value * this->some_func();
});
//此例中total会存为引用, value则会存一份值拷贝. 对this的捕获比较特殊, 它只能按值捕获. this只有当包含它的最
//靠近它的函数不是静态成员函数时才能被捕获.对protect和priviate成员来说, 这个lambda函数与创建它的成员函数有相同的访问控制.
// 如果this被捕获了,不管是显式还隐式的,那么它的类的作用域对Lambda函数就是可见的. 访问this的成员不必使用this->语法,可以直接访问.
```

lambda函数是一个依赖于实现的函数对象类型,这个类型的名字只有编译器知道. 如果用户想把lambda函数做为一个参数来传递, 那么形参的类型必须是模板类型或者必须能创建一个std::function类似的对象去捕获lambda函数.使用 auto关键字可以帮助存储lambda函数,  

```c++
auto my_lambda_func = [&](int x) { /*...*/ };
auto my_onheap_lambda_func = new auto([=](int x) { /*...*/ });
my_lambda_func(1);
```
## std::function
类模板std::function是一种通用、多态的函数封装。std::function的实例可以对任何可以调用的目标实体进行存储、复制和调用操作，这个目标实体包含普通函数、lambda表达式、函数指针以及其他函数对象。std::function对象是对c++中现有的可调用实体的一种类型安全的包裹。

通常std::function是一个函数对象类，它包装其它任意的函数对象，被包装的函数对象具有类型为T1, …,TN的N个参数，并且返回一个可转换到R类型的值。std::function使用 模板转换构造函数接收被包装的函数对象；特别是，闭包类型可以隐式地转换为std::function。std::function统一和简化了相同类型可调用实体的使用方式，使得编码变得更简单。

> 通过std::function对C++中各种可调用实体（普通函数、Lambda表达式、函数指针、以及其它函数对象等）的封装，形成一个新的可调用的std::function对象；让我们不再纠结那么多的可调用实体。一切变的简单粗暴。

```c++
template< class R, class... Args >
class function<R(Args...)>

std::function<void()> f1;

std::function<int (int , int)> f2;
```

R是返回值类型，Args是函数的参数类型，实例一个std::function对象很简单，就是将可调用对象的返回值类型和参数类型作为模板参数传递给std::function模板类。

std::function包含于头文件 #include<functional>中，可将各种可调用实体进行封装统一，包括
* 普通函数
* lambda表达式
* 函数指针
* 仿函数
* 类成员函数
* 静态成员函数

```c++
#include <iostream>
#include <functional>
 
using namespace std;
 
std::function<bool(int, int)> fun;
//普通函数
bool compare_com(int a, int b)
{
    return a > b;
}
//lambda表达式
auto compare_lambda = [](int a, int b){ return a > b;};
//仿函数
class compare_class
{
public:
    bool operator()(int a, int b)
    {
        return a > b;
    }   
};
//类成员函数
class compare
{
public:
    bool compare_member(int a, int b)
    {
        return a > b;
    }
    static bool compare_static_member(int a, int b)
    {
        return a > b;
    }
};
int main()
{
    bool result;
    fun = compare_com;
    result = fun(10, 1);
    cout << "普通函数输出, result is " << result << endl;
 
    fun = compare_lambda;
    result = fun(10, 1);
    cout << "lambda表达式输出, result is " << result << endl;
 
    fun = compare_class();
    result = fun(10, 1);
    cout << "仿函数输出, result is " << result << endl;
 
    fun = compare::compare_static_member;
    result = fun(10, 1);
    cout << "类静态成员函数输出, result is " << result << endl;
 
    ////类普通成员函数比较特殊，需要使用bind函数，并且需要实例化对象，成员函数要加取地址符
    compare temp;
    fun = std::bind(&compare::compare_member, temp, std::placeholders::_1, std::placeholders::_2);
    result = fun(10, 1);
    cout << "类普通成员函数输出, result is " << result << endl;
}
```

## std::bind
std::bind函数将可调用对象(用法中所述6类)和可调用对象的参数进行绑定，返回新的可调用对象(std::function类型，参数列表可能改变)，返回的新的std::function可调用对象的参数列表根据bind函数实参中std::placeholders::_x从小到大对应的参数确定。

```c++
#include <iostream>
using namespace std;
class A
{
public:
    void fun_3(int k,int m)
    {
        cout<<k<<" "<<m<<endl;
    }
};
 
void fun(int x,int y,int z)
{
    cout<<x<<"  "<<y<<"  "<<z<<endl;
}
 
void fun_2(int &a,int &b)
{
    a++;
    b++;
    cout<<a<<"  "<<b<<endl;
}
 
int main(int argc, const char * argv[])
{
    auto f1 = std::bind(fun,1,2,3); //表示绑定函数 fun 的第一，二，三个参数值为： 1 2 3
    f1(); //print:1  2  3
 
    auto f2 = std::bind(fun, placeholders::_1,placeholders::_2,3);
    //表示绑定函数 fun 的第三个参数为 3，而fun 的第一，二个参数分别有调用 f2 的第一，二个参数指定
    f2(1,2);//print:1  2  3
 
    auto f3 = std::bind(fun,placeholders::_2,placeholders::_1,3);
    //表示绑定函数 fun 的第三个参数为 3，而fun 的第一，二个参数分别有调用 f3 的第二，一个参数指定
    //注意： f2  和  f3 的区别。
    f3(1,2);//print:2  1  3
 
 
    int n = 2;
    int m = 3;
 
    auto f4 = std::bind(fun_2, n,placeholders::_1);
    f4(m); //print:3  4
 
    cout<<m<<endl;//print:4  说明：bind对于不事先绑定的参数，通过std::placeholders传递的参数是通过引用传递的
    cout<<n<<endl;//print:2  说明：bind对于预先绑定的函数参数是通过值传递的
 
 
    A a;
    auto f5 = std::bind(&A::fun_3, a,placeholders::_1,placeholders::_2);
    f5(10,20);//print:10 20
 
    std::function<void(int,int)> fc = std::bind(&A::fun_3, a,std::placeholders::_1,std::placeholders::_2);
    fc(10,20);//print:10 20
 
    return 0;
}
```