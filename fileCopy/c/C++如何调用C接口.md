# C++如何调用C接口

## C++如何调用C接口

为什么会有这样的情况呢？想象一下，有些接口是用C实现的，并提供了库，那么C++中该如何使用呢？我们先不做任何区别对待，看看普通情况下会发生什么意想不到的事情。
首先提供一个C接口：

```c
//来源：公众号【编程珠玑】// 博客：https://www.yanbinghu.com
//test.c
#include"test.h"
void testCfun()
{
    printf("I am c fun\n");
    return;
}
```

```
gcc -c test.c
```

另外提供一个头文件test.h：

```c
#include<stdio.h>
void testCfun();
```

我们的C++代码调用如下：

```c++
//来源：公众号【编程珠玑】 博客：https://www.yanbinghu.com
//main.cpp
#include"test.h"
#include<iostream>
using namespace std;
int main(void)
{
    /*调用C接口*/
    cout<<"start to call c function"<<endl;
    testCfun();
    cout<<"end to call c function"<<endl;
    return 0;
}
```

编译：

```
$ g++ -o main main.cpp test.o
/tmp/ccmwVJqM.o: In function `main':
main.cpp:(.text+0x21): undefined reference to `testCfun()'
collect2: error: ld returned 1 exit status
```

很不幸，最后的链接报错了，说找不到testCfun，但是我们确实定义了这个函数。为什么会找不到呢？现在你还会认为C++直接就可以调用C接口了吗？

## 真相

我们都知道，C++中函数支持重载，而C并不支持。C++为了支持函数重载，它在“生成”函数符号信息时，不能仅仅通过函数名，因为重载函数的函数名都是一样的，所以它还要根据入参，命名空间等信息来确定唯一的函数签名。或者说**C++生成函数签名的方式与C不一致**，所以即便是函数名一样，对于C和C++来说，它们最终的函数签名还是不一样。当然这里又是另外一回事了，我们不细说。我们看看两个文件里的函数符号有什么区别：

```
$ nm test.o|grep testCfun
0000000000000000 T testCfun
$ nm main.o|grep testCfun
                U _Z8testCfunv
```

所以它们两个能链接在一起才真是奇怪了呢！名字都不同，还怎么链接？

## 如何处理

那么如何处理呢？很显然，我们必须告诉链接器，这是一个C接口，而不是C++接口，所以需要加入 extern C，我们修改test.h

```c
#include<stdio.h>
extern "C"{
void testCfun();
}
```

这里用extern "C"将testCfun接口包裹起来，告诉编译器，这里的是C代码哈，你要按C代码的方式处理。再次编译：

```
$ g++ -o main main.cpp test.o
$ ./main
start to call c function
I am c fun
end to call c function
```

看终端输出，完美！

## 优化

虽然上面的C接口可以被C++正常调用了，但是如果这个C接口要被C代码调用呢？增加main.c内容如下

```
//main.c
#include"test.h"
int main(void)
{
    /*调用C接口*/
    testCfun();
    return 0;
}
```

编译：

```
$ gcc -o main main.c test.c
In file included from main.c:2:0:
test.h:2:8: error: expected identifier or '(' before string constant
 extern "C"{
        ^
In file included from test.c:2:0:
test.h:2:8: error: expected identifier or '(' before string constant
 extern "C"{
```

不出意外，又报错了，很显然，**C语言中并没有extern "C"这样的写法**，所以为了能使得test.c的代码既能被C++调用，也能被C调用，需要改写成下面这样：

```c
#include<stdio.h>
#ifdef __cplusplus
extern "C"{
#endif

void testCfun();

#ifdef __cplusplus
}
#endif
```

这里通过__cplusplus宏来控制是否需要extern “C”，如果是C++编译器，那么extern "C"部分就会被预处理进去，这样test.c代码就可以既用于C++，也可以用于C啦。

赶快去你的C项目代码头文件中看看，是不是也有这样的代码段呢？

## 问题

为什么我们在C++代码中可以直接调用一些标准C库函数呢？即使你在main函数中调用printf等函数，它也不会出现链接错误。因为库函数已经有了类似的处理了。

如果你还是不确定，你可以先预处理：

```
$ g++ -E main.i main.cpp
```

去生成的main.i文件中找一找，是不是有extern "C"。

## 总结

C++支持重载，而C不支持，C++并不能直接调用C代码写好的接口，因此如果你的C代码想要能够被C调用，也想被C++调用，那么别忘了extern "C"。