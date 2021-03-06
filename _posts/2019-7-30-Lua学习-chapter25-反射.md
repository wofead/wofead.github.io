---
layout:     post
title:      Lua 学习 chapter25
subtitle:   chapter25
date:       2019-7-30
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 自省机制
2. 访问变量
3. 钩子
4. 调优
5. 沙盒

> 只有疯狂过，你才知道自己究竟能不能成功。

## 自省机制
通过debug.getinfo(foo)，函数就会返回一个包含该函数有关的一些数据的表。

* source: 该字段用于说明函数定义的位置。如果函数定义在一个字符串中（通过调用load），那么source就是这个字符串：如果函数定义在一个文件中，那么source就是使用@作为前缀的文件名。
* short_src:source精简版
* linedefined:源代码中第一行
* lastlinedefined：最后一行
* what:该字段用于说明函数的类型。lua函数就是lua，c函数就是c，位于主函数就是main
* name:函数的适当名字，例如保存该函数的全局变量名称
* namewhat：说明一个字段的含义，可能是"global","local","method","field"或""（空字符串）
* nups：函数的上值个数
* nparams：函数参数个数
* isvararg:该字段表明该函数是否为可变长函数
* activelines:该字段是一个包含该函数所有活跃行的集合。活跃行（active line)是指除空行和只包含注释的行外的其他行。
* func:该字段是该函数本身

当时用数字n作为参数调用函数getinfo(2)时，可以得到有关相应栈层次上活跃函数的数据。栈层次是一个数字，代表某个时刻上活跃的的顶函数。调用getinfo的函数A的层次是1，而调用A的函数的层次是2，以此类推。

getinfo效率不高，所以这里可以通过第二个参数提高效率：

* n  选择name和namewhat
* f  选择func 
* S  选择source，short_src，what，linedefined和lastlinedefined
* l  选择currentline
* L  选择activelines
* u  选择nup、nparams和isvararg

debug.getinfo(foo,"SL")

```lua
function traceback()
    for level = 1, math.huge do
        local info = debug.getinfo(level,"Sl")
        if not info then
            break
        end
        if info.what == "C" then
            print(string.format("%d\tC fucntion", level))
        else
            print(string.format("%d\t[%s]:%d",level,info.short_src,info.currentline))
        end
    end
end

traceback()
```

## 访问变量
通过debug.getlocal来检查任意活跃函数的局部变量。还可以通过函数getupvalue来访问一个被lua函数所使用的的非局部变量。

我们还可以通过traceback函数来打印堆栈信息。

## 钩子

调试库中的钩子机制允许用户注册一个钩子函数，这个钩子函数会在程序运行中某个特定事件发生时被调用：

* 每当调用一个函数时产生的call事件
* 每当函数返回时产生的return事件
* 每当开始执行一行新代码产生的line事件
* 执行完指定数量的指令后产生的count事件

钩子函数的注册：通过debug.sethook：第一个参数是钩子函数，第二个参数是描述要监控事件掩码字符串，第三个参数是一个用于描述以何种频度获取count事件的可选参数。

要监控call、return、line事件，把这几个事件的首字母放入掩码字符串。要监控count事件，则需要在第三个参数中指定一个计数器。如果要关闭钩子，不带参数的调用sethook函数即可。

```lua
function hello(event)
    print("hello", event)
end
debug.sethook(hello,"c")
hello()
--[[输出
hello	call
hello	call
hello	call
hellohello	call
	nil
]]--


```

## 调优

可以用来分析程序使用资源的行为，但对于时间方面的调优最好还是使用c，因为钩子函数的调用开销有点大。在这里我们来测试程序执行的每个函数的调用次数。

```lua
local Counters = {}
local Names = {}
local function hook()
    local f = debug.getinfo(2,"f").func
    local count = Counters[f]
    if count == nil then
        Counters[f] = 1
        Names[f] = debug.getinfo(2,"Sn")
    else
        Counters[f] = count + 1
    end
end

-- lua profiler main-prog
local f = assert(loadfile(arg[1]))
debug.sethool(hook,"c")
f()
debug.sethook()


```


