---
layout:     post
title:      Lua 学习 chapter7
subtitle:   chapter7
date:       2019-5-6
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 简单I/O模型
2. 多个返回值
3. 可变长函数参数
4. 
> I want to have a talk with you my heart.

## 简单I/O模型
对于文件操作，I/O提供了两种不同的模型。简单模型虚拟了一个当前输入流和一个当前的输出流，其I/O是通过这些流实现的。
I/O库把当前的输入流初始化为进程的标准输入(C中的stdin),将当前的输出流初始化进程的标准输出(C中的stdout)。因此当执行io.read()这样的语句的时候就可以从标准的输入中读取一行。

函数的io.input和函数io.output用来改变当前的输入输出流。
io.input(fileName)会以只读的方式打开指定的文件，并将文件设置为当前的输入流。之后所有的输入都将来自于这个文件，除非重新调用io.input函数来改变。对于输出与之类似。

**函数write()**:
由于调用该函数可以传入多个参数，所以应该避免使用io.write(a..b..b),而是使用write(a,b,c)。
作为原则应该只在用后**即弃**的代码或者调试代码的时候调用print输出，当需要完全控制输出是，应该使用io.write()输出，和print不同，
函数io.write最终的输出不会添加诸如制表符或者换行符这样的额外内容。
此外函数io.write允许对输出进行重定向，而函数print只能使用标准输出。函数print会为其参数调用tostring函数。

```lua
io.write("sin(3)=",math.sin(3),"\n") -- >sin(3) = 0.1411200080
io.write(string.format("sin(3)=%.4f\n",math.sin(3))) -- >sin(3) = 0.1411

io.read("a") -->从当前位置开始的读取当前输入文件的全部内容
```

函数io.read可以从当前的输入流中读取字符串，其参数决定了其要读取的数据：

1. "a"  读取整个文件
2. "l"  读取下一行 （丢弃换行符）  默认参数
3. "L"  读取下一行（保留换行符）
4. "n"  读取一个数值
5. num  以字符串读取num个字符

获取一个文件的所有行数可以使用 io.lines() 函数

```lua
local lines
```