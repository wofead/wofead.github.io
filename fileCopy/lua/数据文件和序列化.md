---
layout:     post
title:      Lua 学习 chapter15
subtitle:   chapter15
date:       2019-7-29
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 数据文件
2. 序列化


> You will feel happy, because you will get a prize.

## 数据文件

通过提前定义全局函数，然后通过执行数据文件来直接调用全局函数的方式来获取数据，并执行。

其中Entry{code}和Entry({code})是相同的，后者以表作为唯一的参数来调用函数Entry。下面的data文件也是一段Lua程序，所以在调用这个文件之前你需要定义好Entry函数。

```lua
--data file
Entry{
    author = "jow",
    title = "lua",
    year = "2019",
}

Entry{
    author = "jowney",
    title = "openGL",
    year = "2019",
}
-- lua file
local authors = {}
function Entry(b)
    authors[b.author] = true
end
dofile("chapter15/data")
for i, v in pairs(authors) do
    print(i)
end
```

## 序列化

用来输出表，在这里我们可以用这个方式来输出一个表或者function。

在进行序列化的时候，lua提供了一个格式序列化的函数，string.format("%q",o),这个选项被设计为以一种能够让Lua语言安全的反序列化字符串的方式来序列化字符串，它使用双引号扩住字符串并正确的转义其中的双引号和换行符等其它字符。

```lua
function serialize(o)
    local t = type(o)
    if	t == "number" or t == "string" or t == "boolean" or t == "nil" then
        io.write(string.format("%q",o))
    end
end

function serialize(o)
    local t = type(o)
    if	t == "number" or t == "string" or t == "boolean" or t == "nil" then
        io.write(string.format("%q",o))
    elseif t == "table" then
        io.write("{\n")
        for i, v in pairs(o) do
            io.write("   ",i,"==")
            serialize(v)
            io.write(",\n")
        end
        io.write("}\n")
    else
        error("cannot serialize a" .. type(o))
    end
end

function basicSerialize(o)
    return string.format("%q", o)
end

function save(name, value, saved)
    saved = saved or {}
    io.write(name, " = ")
    local t = type(value)
    if t == "number" or t == "string" or t == "boolean" or t == "nil" then
        io.write(basicSerialize(value), "\n")
    elseif t == "table" then
        if saved[value] then
            io.write(saved[value], "\n")
        else
            saved[value] = name
            io.write("{}\n")
            for i, v in pairs(value) do
                i = basicSerialize(i)
                local fname = string.format("%s[%s]", name, i)
                save(fname, v, saved)
            end
        end
    else
        error("cannot serialize a" .. type(o))
    end
end

b = {2,3,4}
a = {x = 1, y =2, {3,4,5}}
a[3] = b
a[2] = a
a.z = a[1]
b.z = b

save("a",a)

--输出
a = {}
a[1] = {}
a[1][1] = 3
a[1][2] = 4
a[1][3] = 5
a[2] = a
a[3] = {}
a[3][1] = 2
a[3][2] = 3
a[3][3] = 4
a[3]["z"] = a[3]
a["x"] = 1
a["y"] = 2
a["z"] = a[1]
```

我们在格式化表的时候，还是要注意循环表的，每次输出完一个表就保存一下，避免进入死循环。
