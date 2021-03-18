---
layout:     post
title:      Lua 学习 chapter9
subtitle:   chapter9
date:       2019-5-26
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录

1. 函数是第一类值
2. 高阶函数


> Just be handsome.

## 函数是第一类值

语法糖：

```lua
function foo(x) return 2*x end
function (x) body end --就是韩式的构造器
function derivative (f, delta)
delta = delta or 1e-4  --0.0001
return function(x)
		(f(x + delta) - f(x)) / delta)
	end
```
例如table.sort函数的第二个函数是以函数作为参数的，这种函数我们称之为高阶函数。


在lua中所有的函数都是匿名的。当讨论函数名的时候，实际上是保存该函数的变量。

如上面的derivative函数，求导函数。

局部函数对于包而言尤其有用：由于lua语言将每个程序段作为一个函数处理，所以在一段程序中声明的函数就是局部函数，这些函数只在该程序段可见。词法定界保证了程序段中的其他函数可以使用这些局部函数。

在使用局部函数的时候，注意递归，当在局部函数中递归自己的时候会导致未定义问题，所以应该先声明，再定义。

## 高阶函数

```lua
local function disk(cx, cy, r)
    return function(x, y)
        return (x - cx) ^ 2 + (y - cy) ^ 2 <= r ^ 2
    end
end

local function rect(left, right, top, bottom)
    return function(x, y)
        return x <= right and x >= left and y <= top and y >= bottom
    end
end

local function union(r1, r2)
    return function(x, y)
        return r1(x, y) or r2(x, y)
    end
end

local function intersection(r1, r2)
    return function(x, y)
        return r1(x, y) and r2(x, y)
    end
end

local function difference(r1, r2)
    return function(x, y)
        return r1(x, y) and not r2(x, y)
    end
end

local function trance(r, dx, dy)
    return function(x, y)
        return r(x - dx, y - dy)
    end
end
```

## 闭包

lua编译一个函数是，其中包括了函数体对用的虚拟机指令、函数用到的常量值（数、文本字符串等等）和一些调试信息。在运行时，每当Lua执行一个形如function...end这样的函数时，他就会创建一个新的**数据对象**，其中包含了相应**函数原型的引用、环境（用来查找全局变量的表）的引用以及一个由所有upvalue引用组成的数组**，而这个数据对象就称为**闭包**。由此可见，函数是编译期的概念，是静态的，而闭包是运行期间的概念，是动态的。

upvalue实际上是局部变量，而局部变量保存在函数堆栈框架上的，所以只要upvalue还没有离开自己的作用域，他就一直生存在函数堆栈上。这种情况下，闭包将通过指向堆栈上的upvalue的引用来访问他们，一旦upvalue即将离开自己的作用域，在从堆栈上消除之前，闭包就会为它分配空间并保存当前的值，以便内嵌函数可以访问到这个upvalue，一般情况下，都是内嵌函数将这个upvalue复制到自己管理的空间中，以便将来访问。