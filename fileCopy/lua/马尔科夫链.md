---
layout:     post
title:      Lua 学习 chapter19
subtitle:   chapter19
date:       2019-7-29
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 马尔科夫链

> Continue, come on.

## 马尔科夫链

```lua
local function allwords()
    local line = io.read()
    local pos = 1
    return function()
        while line do
            local w,e = string.match(line, "(%w+[,;.:]?)()",pos)
            if w then
                pos = e
                return w
            else
                line = io.read()
                pos = 1
            end
        end
        return nil
    end
end

local function prefix(w1,w2)
    return w1 .. " " .. w2
end

local statetab = {}

local function insert(prefix, value)
    local list = statetab[prefix]
    if list == nil then
        statetab[prefix] = {value}
    else
        table.insert(list,value)
    end
end

local MAXGEN = 200
local NOWORD = "\n"

--创建表
local file = io.open("essay")
io.input(file)
local w1, w2 = NOWORD,NOWORD
for nextword in allwords() do
    insert(prefix(w1,w2),nextword)
    w1 = w2
    w2 = nextword
end

insert(prefix(w1,w2),NOWORD)

--生成文本

w1 = NOWORD; w2 = NOWORD
for i = 1, MAXGEN do
    local temp = prefix(w1,w2)
    local list = statetab[temp]
    --从列表中随机选出一个元素
    local r = math.random(#list)
    local nextword = list[r]
    if nextword == NOWORD then return end
    io.write(nextword," ")
    w1 = w2; w2 = nextword
end
```

上面的代码，allwords这个迭代器函数构造的还是十分的巧妙的，利用迭代器遍历文章中的每一个单词，然后将其存到一张表中，方便进行随机整合。