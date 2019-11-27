---
layout:     post
title:      复合类型
subtitle:   c++
date:       2019-9-14
author:     Jow
header-img: img/about-bg-walle.jpg
catalog: 	 true 
tags:
    - C++

---

### 目录
1. 数组的初始化
2. 字符串
3. 共用体
4. 指针的使用


> Study hard, work hard.


## 数组的初始化
可以使用大括号直接对数组进行初始化，在这个过程中还可以省略等号。

```c++
int counts[4] {1,2,3,4};
int counts1[4] {};

long plifs[] = {25,92,3.0};// not allowed，浮点数变为整数进行了缩窄转换
char slifs[4] {'h','i',123232,'\0'};//not allowed, 插入类型存储整型最大127

```

## 嵌套类
字符数组和字符串的区别在于是否在数组的结尾存在'\0'字符。而且字符串数组长度比字符串长度大一，因为最后的字符串结尾的原因。

```c++
char charArr[3] {'a','s','d'};
char string[4] {'a'','s','d','\0'}
```

使用getline()和get()读取一行字符串，它们两个都能够读取一行字符串，但是getline()会丢弃换行符，而get()会将换行符保留在输入序列中。

在使用getline的时候它会将换行符替换成空字符，存储到字符串数组中。

在使用get的时候，由于它会把换行符留在输入序列，所以我们需要再调用一次get函数把这个换行符给抛弃掉。

```c++
cin.get(name, ArSize).get();
cin.get(dessert,ArSize);

cin >> year; //在使用cin进行输入取值的时候，换行符也会被留在输入队列中，所以在进行下次取值的时候要对输入序列进行get().
```


## 共用体
共用体是一种数据格式，他能够存储不同的数据类型，但只能同时存储其中的一种类型。

## 指针的使用
声明一个指针，这个时候会给指针分配一个int类型的内存空间，但是不会分配用来存储指针所指向的数据的内存。下面这个例子就是错误的使用。

```c++
int *ptr;
ptr =100;// error，因为100并没有内存空间。
int *ptr1 = new int;
*ptr1 = 100; //right 
```

我们一定要在对指针应用解除引用运算符(*)之前，将指针初始化为一个确定的、适当的地址。

指针本身对象的值都被存储在栈中，而new出来的对象，被存储在堆中，栈中的对象由程序自动回收，但是堆中的数据，需要程序员手动释放这些数据。

总之，使用new和delete时，应该遵守以下规则：
* 不要使用delete来释放不是new分配的内存。
* 不要使用delete释放同一个内存块两次。
* 如果使用new[]为数组分配内存，则应该使用delete[]来释放。
* 使用new为一个实体分配内存，则应该使用delete来释放。
* 对空指针应用delete是安全的。

使用数组声明来创建数组时，将采用静态联编，即数组的长度在编译时设置。使用new[]运算符创建数组时，将采用动态联编，即将在运行时为数组分配空间，其长度也在运行时设置。

c++有3中管理数据内存的方式：自动存储、静态存储和动态储存。
1. 自动存储：在函数内部定义的常规变量使用自动存储空间，被称为自动变量，这意味着他们在所属的函数被调用时自动产生，在该函数结束时消亡。
2. 静态存储：静态存储是整个程序执行期间都存在的存储方式。使变量成为静态的方式有两种：一是在函数外面定义它；另一种是在声明变量时使用关键字static。
3. 动态存储：new和delete运算符提供了一种比自动和静态存储更灵活的方法。它们管理一个内存池，使用new来申请内存，而delete来释放内存。
