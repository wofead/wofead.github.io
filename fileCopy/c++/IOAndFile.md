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
4. 使用cin进行输入
5. 文件的输入和输出


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

## 使用cin进行输入
典型的运算符函数原型如下：

istream & operate >>(int &);

**cin如何检查输入**

不同版本的抽取运算符查看输入流的方法是相同的。他们跳过空白(空格、换行符和制表符），直到遇到非空白符。基数对于单字符模式，情况也是如此。当抽取字符的时候，它读取从费控字符开始，到与目标类型不匹配的第一个字符之间的全部内容。例如：

```c++
int elevation;
cin >> elevation;
```

假设键入-123Z这些字符，运算符将读取字符-，1,2,3，因为它们都是整数的有效部分。但Z就不会读取了，会被留在输入流中。当输入没有满足程序的期望。抽取运算符将不会改变elevation的值，并返回0。

在流的概念中，cin和cout对象包含一个描述流状态的数据成员。它们由3个ios_base元素组成：eofbit、badit或failbit，其中每个元素都是一位，可以是1（设置）或者0（清除）.当cin操作到达文件末尾时，它将被设置为eofbit；当cin未能读到预期的字符时，它将被设置failbit。在一些无法诊断的失败破坏流时，badbit元素将被设置。当全部三个 状态位都被设置为0时，说明一起正常。

状态的设置：
> 通过使用clear和setstate函数来设置状态，clear()方法将状态设置为它的参数，默认可以不带参，这将清除3个状态位(clear()),同样可以设置参数位，清除其他两个状态位(claer(eofbit).而setstate方法只影响其参数中已经设置的位。

只有流状态良好（所有的位都被清除）的情况下，下面的测试才能返回true。（while (cin >> input)）;

**其他istream类方法**

* 方法get()和getline()方法。
* 方法get(char&)和get(void)提供不跳过空白单字符的输入功能。
* 函数get(char\*,int,char)和getline(char\*,int,char)在默认情况下读取整行而不是一个单词。
* read()函数读取指定数目的字节，并将它们存储在指定的位置中。

## 文件的输入和输出
在对文件进行操作的时候，要写入文件件，需要创建一个ofstream对象，并使用ostream方法，如<<插入运算符或write()。要读文件，需要创建一个ifsream对象，并使用istream方法，入>>抽取运算符或者get().

要写入文件必须这样做：
1. 创建一个ofstream对象来管理输入流；
2. 将该对象与特定的文件关联起来；
3. 以使用cout的方式来使用该对象，唯一的区别是输入将进入文件，而不是屏幕。

```c++
ofstream fout;
fout.open("jar.text");
//ofstream fout("jar.text");
fout << "Dull Data."
fout.close();
```

像fout这样的ofstream对象从程序那里逐字节的收集输出，当缓冲区满以后，它将缓冲区内容一同传输给目标文件。以这种方式打开文件，如果没有这个文件就会创建这个文件，如果是旧文件就会清空再次写入。

读取文件中的内容：
* 创建一个ifstream对象来管理输入流
* 将该对象与特定的文件关联起来；
* 使用cin的方式使用该对象。

```c++
ifstream fin("jamjar.text");
char ch;
fin >> ch;
char buff[80];
fin >> buff;
fin.getline(buff,80);
string line;
getline(fin, line);


//检查一个文件是否被打开成功:
if(fin.fail()){};
if(!fin){};
if(!fin.is_open()){};
fin.close();
```

**文件模式：**

文件模式描述的是文件将被如何使用：读、写、追加等。将流与文件关联时(无论是使用文件名初始化文件流对象，还是使用open()方法），都可以提供指定文件模式的第二个参数： ifstream fin("test.text",mode1);

|常量|含义|
|:-:|:-:|
|ios_base::in|打开文件，以便读取|
|ios_base::out|打开文件，以便写入|
|ios_base::ate|打开文件，并移动到文件尾|
|ios_base::app|追加到文件尾|
|ios_base::trunc|如果文件存在，则截短文件|
|ios_base::binary|二进制文件|
