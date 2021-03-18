---
layout:     post
title:      Lua 学习 chapter24
subtitle:   chapter24
date:       2019-7-30
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 将协程用作迭代器
2. 事件驱动式编程

> 人人真真的生活过，学习过，改变过，努力过，才能创造出一个满意的自己。

## 将协程用作迭代器

不断的resume来实现迭代。

```lua
local function printResult(a)
    for i = 1, #a do
        io.write(a[i], " ")
    end
    io.write("\n")
end

local function permgen(a,n)
    n = n or #a
    if n <= 1 then
        coroutine.yield(a)
    else
        for i = 1, n do
            a[n],a[i] = a[i],a[n]
            permgen(a, n -1)
            a[n],a[i] = a[i],a[n]
        end
    end
end

local function permutations(a)
    local co = coroutine.create(function() permgen(a) end)
    return function()
        local code, res = coroutine.resume(co)
        return res
    end
end

for v in permutations({"a","b","c"}) do
    printResult(v)
end
```

## 事件驱动式编程

假设我们有一个I/O库：

1. lib.runloop()：运行事件循环，在其中处理所有发生的事件并调用对应的回调函数。
2. lib.readline(steam,callback)：从指定流中读取一行，并在读取完成后带着读取的结果调用指定的回调函数
3. lib.writeline(stream,line,callback)：该函数写入一行
4. lib.stop():打破循环，用于程序结束

```lua
local lib = {}
lib.readline = function(callback)
    print("等待输入：")
    lib.waitInput = true
    lib.callback = callback
end
lib.writeline = function(line,callback)
    io.write(line .. "from others")
    lib.waitWrite = true
    lib.callback = callback
end

lib.stop = function()
    lib.flag = false
end

lib.runloop = function()
    lib.flag = true
    while lib.flag do
        if lib.waitInput then
            local line = io.read()
            lib.waitInput = false
            lib.callback(line)
        elseif lib.waitWrite then
            lib.waitWrite = false
            lib.callback()
        end
    end
end
local function run (code)
    local co = coroutine.wrap(function()
        code()
        lib.stop()
    end)
    co()
    lib.runloop()
end

function putline(line)
    local co = coroutine.running()
    print(co)
    local callback = (function() coroutine.resume(co) end)
    lib.writeline(line,callback)
    coroutine.yield()
end

function getline(line)
    local co = coroutine.running()
    print(co)
    local callback = (function(l)
        coroutine.resume(co,l)
    end)
    lib.readline(callback)
    line = coroutine.yield()
    return line
end

run(function ()
    local t= {}
    while true do
        local line = getline()
        if not line then
            break
        end
        table.insert(t,line)
    end
    for i = #t, 1, - 1 do
        putline(t[i] .. "\n")
    end
end)
```