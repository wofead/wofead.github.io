---
layout:     post
title:      cocos项目的编译
subtitle:   ddt
date:       2019-11-15
author:     Jow
header-img: img/about-bg-walle.jpg
catalog: 	 true 
tags:
    - ddt

---

### 目录
1. project

> Sunny.

## project

编译顺序：

1. libcocos2d:如果出现汇编内联失败，注释掉CCLuaStack.cpp文件中264和265并且return 0。   afxres.h未找到，将这一行代码注释掉，替换成#include<windows.h>。
2. libspine
3. LibBaseGame： afxres.h未找到，将这一行代码注释掉，替换成#include<windows.h>。
4. liblua
5. libhttp
6. libjemalloc
7. libnet
8. libsaspine
9. libwidget
10. libaudio，出现function不是std空间的函数，在对应的头文件中添加#include <functional>。这两个文件添加头文件functional。AudioCache和AudioPlayer。
11. libhelper，出现__cpuid找不到，注释掉deviceHelper的11到16行

将MangoEngineStartup设置为启动项目，然后运行即可。