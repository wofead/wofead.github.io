---
layout:     post
title:      Lua 学习 chapter21
subtitle:   chapter21
date:       2019-7-29
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 类
2. 继承
3. 私有性
4. 单方法对象

> 星空中划过的不仅仅是流星，还有我对这个世界的畅想。

## 类

```lua
function Account:new(o)
	 o = o or {}
	self.__index = self
	setmetatable(o, self)
	return o
end

SpecialAccount  = Account:new()
s = SpecialAccount:new{limit = 1000.00}
```

## 继承
```lua
local function search(k, plist)
    for i = 1, plist do
        local v = plist[i][k]
        if v then
            return v
        end
    end
end
function createClass(...)
    local c = {}
    local parents = {...}
    setmetatable(c,{__index = function(t,k)--t代表自身
        --return search(k, parents)
        local v = search(k, parents)
        t[k] = v
        return v
    end})
    c.__index = c
    function c:new(o)
        o = o or {}
        setmetatable(o, c)
        return o
    end
    return c
end
    
```

## 私有性
```lua
function newAccount(initBalance)
    local self = { balance = initBalance }
    local withdraw = function(v)
        self.balance = self.balance - v
    end
    local deposit = function(v)
        self.balance = self.balance + v
    end
    local getBalance = function()
        return self.balance
    end

    return { withdraw = withdraw,
             deposit = deposit,
             getBalance = getBalance
    }
end
```
通过返回一个外部对象，这样就调用不到self了。

## 单方法对象

```lua
local function newObject(value)
    return function(action,v)
        if action == "get" then
            return value
        elseif action == "set" then
            value = v
        else
            error("invalid action")
        end
    end
end

local d = newObject(0)

print(d("get"))
print(d("set", 10))
print(d("get"))
```

