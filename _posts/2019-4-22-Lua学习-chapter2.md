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
book={0,0,0,0,0,0,0,0}
function addQueen(a,n)
    if n>N then
        for i=2,N do
            if checkAttack(a,i,a[i])==false then
                return
            end
        end
        printTable(a)
    else
        for c=1,N do
            if book[c]==0 then
                a[n]=c
                book[c]=1
                addQueen(a,n+1)
                book[c]=0
            end
        end
    end
end
```


