---
layout:     post
title:      Lua 学习 chapter2
subtitle:   chapter2
date:       2019-4-22
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. Eeight Queen
2. 判断是否可以放置
3. 递归放置
4. 只打印第一个值
5. 排列的方式
6. 对比

> Love is you power, chek your power, and try your best.

## Eight Queen
8皇后问题：
在一个8*8的棋盘上放置八个皇后，她们之间相互不能影响。
通过一个表来存储，键值表示行，值表示列。

## 判断是否可以放置
因为键值已经保证不会再同一行，所以我们需要判断的就只有列和斜方向的。
在斜方向，左斜和右斜。右斜列减行相同则同处一斜行。左斜行列相加则处于同斜行。

```lua
local function checkAttack(table, n, c)
    for i = 1, n - 1 do
        if (table[i] == c) or
                (table[i] - i == c - n) or --行列式
                (table[i] + i == c + n) then
            return false
        end
    end
    return true
end
```

## 递归放置
通过判断是否每一行都放置了棋子来决定是否打印这个棋谱。
在addQueen中，每一个节点都会一直的递归，所有的节点递归结束。
当然在中间不满足条件的后面的节点就不会递归了。

```lua
local function addQueen(table, n)
    if n > N then
        printTable(table)
    else
        for c = 1, N do
            if checkAttack(table, n, c) then
                table[n] = c
                addQueen(table, n + 1)
            end
        end
    end
end
```

## 只打印第一个值
在addQueen前面加一个判断，如果打印了一个可行解就不再进行冗余递归了。


## 排列的方式
```lua
---@param t table|number
local function deepCopy(t)
    local result = {}
    if type(t) == "number" then
        table.insert(t, 1, t)
    else
        for i, v in ipairs(t) do
            result[i] = t[i]
        end
    end
    return result
end

---@param table table
---@param n number
---return table
local function insertNumber(t, n)
    local result = {}
    for i = 1, #t + 1 do
        result[i] = deepCopy(t)
        table.insert(result[i], i, n)
    end
    return result
end

---@param t1 table
---@param t2 table
---@return table
local function mergeTable(t1, t2)
    local length = #t1
    for i = 1, #t2 do
        table.insert(t1, length + i, t2[i])
    end
end

---@param table table
---@param n number
---@return table
local function arrangement(t, n)
    local temp = {}
    for i = 1, #t do
        local tt = insertNumber(t[i], n)
        mergeTable(temp, tt)
    end
    return temp
end

local function getArrangement()
    local table = { { 1, 2 }, { 2, 1 } }
    for i = 3, N do
        table = arrangement(table, i)
    end
    return table
end

local tr = getArrangement()
for i, v in ipairs(tr) do
    local flag = true
    for j, k in ipairs(v) do
        if not checkAttack(v, j, k) then
            flag = false
            break
        end
    end
    if flag then
        printTable(v)
    end
end

print("Total number:", #tr)
print("Total answer2:" .. totalResult)
print("Method2 time:", os.clock() - t1)

```

## 对比

![](https://i.imgur.com/GTTE5EV.png)

![](https://i.imgur.com/slauuLs.png)

方法2：排列花费1.604秒，检测花费将近1.1秒，这说明排列还是浪费了很多时间在检测和排列上面的。

