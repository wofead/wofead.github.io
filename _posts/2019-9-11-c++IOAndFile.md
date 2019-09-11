---
layout:     post
title:      IO
subtitle:   c++
date:       2019-9-9
author:     Jow
header-img: img/about-bg-walle.jpg
catalog: 	 true 
tags:
    - C++

---

### 目录
1. 流和缓冲区
2. 流、缓冲区和iostream文件
3. 使用cout进行输出


> Study hard, work hard.


## 流和缓冲区
c++程序把输入和输出看作字节流。输入时，程序从输入流中抽取字节；输出时，程序将字节插入到输入流中。对于面向文件的文本的程序，每个字节代表一个字符，更通俗的说，字节可以构成字符或者数值数据的二进制表示。流充当了程序和流源或流目标之间的桥梁。通过使用流，c++程序处理输入的方式将独立于其去向。因此管理输入包括两步：

* 将流与输入去向的程序关联起来。
* 将流与文件连接起来。


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

通常，通过使用缓冲区可以更高效的处理输入和输出。缓冲区使用做中介了内存块，它将信息从设备传输到程序或从程序传输给设备的临时存储工具。通常，像磁盘驱动器这样的设备以512字节（或更多）的块为单位来传输信息，而程序通常每次只能处理一个字节的信息。缓冲区帮助匹配这两种不同的信息传递速率。

输出时，程序首先填满缓冲区，然后把整块数据传输给硬盘，并清空缓冲区，以备下一批输出使用。c++程序通常在用户按下回车键的时候刷新输入缓冲区。


## 流、缓冲区和iostream文件
管理流和缓冲区的工作有点复杂，但iostream文件中包含一些专门设计用来实现管理流和缓冲区的类。
* streambuf类为缓冲区提供内存，并提供了用于填充缓冲区、访问缓冲区内容、刷新缓冲区和管理缓冲区内存的类方法。
* ios_base类表示流的一特征，如是否可读、是二进制流还是文本流。
* ostream类是从ios类派生来的，提供输出方法。
* istream类也是从ios类派生而来的，提供输入方法。
* iostream类是基于istream和ostream类的，因此继承了输入方法和输出方法。

c++的iostream类库管理了很多细节。
* cin对象对应于标准输入流。
* cout对象对应于标准输出流。
* cerr对象与标准错误流对应，可用于显示错误消息。（不会缓冲，直接输出）
* clog对象也对应着标准错误流。（会缓冲）

ostream类定义了上述语句中使用的operate<<()函数，ostream类还支持cout数据成员以及其他大量的类方法。

## 使用cout进行输出
本书结合使用cout和<<运算符来进行 输出。在out类中重载了<<运算符。针对于每种类型都有与之对应的重载函数。

```c++
cout << 88;
ostream &operate<<(int);
```

从上面的返回值我们可以发现所有的化身的返回值都是ostream &类型，所以cout的输出是可以拼接的。

除了各种的operate<<()函数外，ostream类还提供了了put()方法和write()方法。前者用于显示字符，而后者用于显示字符串。write函数的第一个参数表示显示的字符串，第二个参数表示显示个数。

```c++
cout.put('w').put('i');
cout.write("dsfaf",5);
```

如果程序使用cout将字节发送到标准输出，由于ostream类对cout对象处理的输出进行缓冲，所以输出不会立即发送到目标地址，而是被存储在缓冲区中，知道缓冲区填满。然后程序将刷新缓冲区，把内容发送出去，并清空缓冲区。 一般情况先换行符会导致缓冲区刷新。

cout的输出是可以格式化的：
1. 修改显示时间使用的计数系统：可以使用dec，hex和oct控制符来控制显示数字。hex(cout);cout << hex;这两种方式修改输出模式。
2. 调整字段宽度：cout.width(5);输出长度为5，右靠齐，左侧补0.
3. 填充字符：cout.fill('*')，这个是和width配合使用的。
4. 设置浮点数的显示精度：cout.precission(2),精度设置为2.
5. 打印末尾的0和小数点：cout.setf(ios_base::showpoint);
6. setf还有很多其他的用途。
7. 标准控制符

|常量|含义|
|:-:|:-:|
|ios_base::boolalpha|输入和输出bool值，可以为true或false|
|ios_base::showpoint|对于输出，使用c++基数前缀(0,0x)|
|ios_base::showbase|显示末尾的小数点|
|ios_base::uppercase|对于16进制输出，使用大写字母，E表示法|
|ios_base::showpos|在整数前面加上+|
