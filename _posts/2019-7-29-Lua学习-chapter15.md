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
                local fname = string.format("%s[%s]", name, k)
                save(fname, v, saved)
            end
        end
    else
        error("cannot serialize a" .. type(o))
    end
end
```



