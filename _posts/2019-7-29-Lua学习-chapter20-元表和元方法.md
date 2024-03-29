---
layout:     post
title:      EmmyLua 学习使用
subtitle:   Postfix Templates
date:       2019-4-12
author:     Jow
header-img: img/post-bg-2015.jpg
catalog: 	 true 
tags:
    - Lua
    - ToolStudy

---

### 目录
1. lua元表
1. class的探索

> 一直都在使用Idea加上EmmyLua进行开发，但是当初别人为什么这样选择和选择的优点在哪里自己一概不知，直到前段时间翻到EmmyLua带有注解功能，才明白使用的好处。


## lua元表

### 基本类型

1. nil
2. boolean
3. number
4. string
5. function
6. userdata
7. thread
8. table

要想对lua有更加深入的了解，不深入的了解lua的元表是不行的，在lua中元表是你构建一个复杂的数据结构的基础，所以需要对元表有非常深入的了解。

元表允许改变table的行为，每个行为关联对应的原方法。

在这里有两个关于元表非常重要的方法：

* setmetatable(table,metatable)
* getmetatable(table)

**元表(metatable)中存在 __metatable 键值,则会设置和返回失败**

**__metatable键值是用于安全考虑，可以防止获取和修改元表中的内容。**

元方法是关联两个元表的桥梁，是非常重要的，下面介绍几个重要的元方法。

### 元方法 ###

元方法一共分为两种：

1. 系统使用的元方法
2. 自定义的元方法

#### 系统使用的元方法 ####

算术元方法以及逻辑元方法：

__cancat用于字符串

|模式|描述|
|----|----|
|__add|加|
|__sub|减|
|__mul|乘|
|__div|除|
|__mod|取余|
|__unm|负号|
|__pow|取幂|
|__concat|...|
|__eq|==|
|__lt|<|
|__le|<=|

```lua
local table1 = {1,2,3}
local table2 = {4,5,6}
local fn = function (t1,t2)          -----table会被传入作为参数
  local newtable = {}
  for i=1,3 do
    newtable[i] = t1[i] + t2[i]
  end
   return newtable
end
setmetatable(table2,{__add = fn})
local newtable = table1+table2
for k,v in pairs(newtable) do
  print("newtable --",k,v)
end
--打印结果：
--newtable --     1    5
--newtable --     2    7
--newtable --     3    9
```

```lua
local a = { 1, 2, 3 }
local b = { 2, 3, 4, 5 }

local function metatableAdd(t1, t2)
    local newTable = {}
    local count = 1
    while t1[count] or t2[count] do
        newTable[count] = (t1[count] or 0) + (t2[count] or 0)
        count = count + 1
    end
    return newTable
end

local function metatableMul(t1, t2)
    local set = {}
    local result = {}
    for i, v in ipairs(t1) do
        set[v] = v
    end
    for i, v in ipairs(t2) do
        set[v] = v
    end
    for k, v in pairs(set) do
        table.insert(result,v)
    end
    return result
end

local function metatableToString(set)
    local l = {}
    for i, v in pairs(set) do
        table.insert(l,v)
    end
    return "{" .. table.concat(l,", ") .. "}"
end

setmetatable(b, { __add = metatableAdd, __mul = metatableMul, __tostring = metatableToString })

local c = b + a

for i, v in ipairs(c) do
    print(v)
end

print("------")

local d = b * a

for i, v in ipairs(d) do
    print(v)
end

-- 打印b的时候会调用tostring
print(b)

local function indexFunc(t, key)
    print(#t,key)
end

local metaTableOfB = getmetatable(b)

metaTableOfB.__index = indexFunc

print(b.z)

-- 输出
--4	z
--nil

local indexTable = {z = 90}

metaTableOfB.__index = indexTable

print(b.z, b.c)

-- 90 nil
```

**通过上面的例子我们可以理解元表和元方法，可以知道元表是元方法的载体，没有元表，就不存在元方法，没有元方法，元表也没有任何意义。**

**我们在对同一类型的东西设置元方法的时候可以对其封装，类似于创建一个对象，每个类被创建出来的时候具有的元方法是一样的。**

两表相加，必须至少其中一个表设置了带__add键的元表，否侧会报错（其他运算符同理），程序会执行__add对应的函数。如果两个表都设置了有__add键的元表，程序会去执行“+”左侧的表中的元表的__add对应的函数。


table访问的元方法：

**__index 和 __newindex**

**__index**的作用，访问当前table的时候，发现没有这个键值（或者键值对应的值为空），那个table就会去寻找metatable中__index元，如果存在__index,就去__index包含表格,就去表中寻找键值。当__index为函数时，则调用方法。方法会默认传入两个参数，一个是self,另一个是key值

```lua
mytable = setmetatable({key1 = "value1"}, { __index = { key2 = "metatablevalue" } })
print(mytable.key1,mytable.key2) --value1    metatablevalue

local function fn(table, key)
	print("table and key",table,key)
end

mytablefun = setmetatabke({key1 = "value1"}, { __index = fn )

```

总结：

Lua 查找一个表元素时的规则，其实就是如下 3 个步骤:

1. 在表中查找，如果找到，返回该元素，找不到则继续
2. 判断该表是否有元表，如果没有元表，返回 nil，有元表则继续。
3. 判断元表有没有 __index 方法，如果 __index 方法为 nil，则返回 nil；如果 __index 方法是一个表，则重复 1、2、3；如果 __index 方法是一个函数，则返回该函数的返回值。

**查询：访问表中不存的字段 :rawget(t, i)**

**__newindex**,方法是用来对表进行更新，__index则用来对表访问 。

当你给表的一个缺少的索引赋值，解释器就会查找__newindex 元方法：如果存在则调用这个函数而不进行赋值操作。

当table不存在键值的时候，__newindex为table，则可以_newindex包含的table来访问这个key值，如果__newindex为函数，直接直接调用函数。

**更新：向表中不存在索引赋值 :rawset(t, k, v)**

**rawget与rawset 直接访问和设置table而不会去访问元方法__index 和 __newindex,也就不会去更新表了，说白了就是操作自身，而不是去访问元表**

```lua
mymetatable = {}
mytable = setmetatable({key1 = "value1"}, { __newindex = mymetatable })

print(mytable.key1)

mytable.newkey = "新值2"
print(mytable.newkey,mymetatable.newkey)

mytable.key1 = "新值1"
print(mytable.key1,mymetatable.key1)

-- value1
-- nil    新值2
-- 新值1    nil

function fn (table,key,value)
    print(table,"\n",key,"\n",value)
end
local tableB = {k1 = "Hi"}
setmetatable(tableB,{__newindex = fn})    ---会将table、key、value传给fn作为参数
tableB.k2 = "Good"
--打印结果：
--table: 000000000256a650
--        k2
--        Good
```

**__call 元方法**让表成为函数，可以带参数，即此元方法在 Lua 调用一个值时调用

```lua
function table_maxn(t)
    local mn = 0
    for k, v in pairs(t) do
        if mn < k then
            mn = k
        end
    end
    return mn
end

-- 定义元方法__call
mytable = setmetatable({10}, {
  __call = function(mytable, newtable)
        sum = 0
        for i = 1, table_maxn(mytable) do
                sum = sum + mytable[i]
        end
    for i = 1, table_maxn(newtable) do
                sum = sum + newtable[i]
        end
        return sum
  end
})
newtable = {10,20,30}
print(mytable(newtable))
```

**__tostring,可以自定义table的输出行为**

```lua
local table = {q,b,c}
local function fn(t)
	return "Hello here"
end
setmetatable(table,{__tostring = fn})
print(table)

--打印结果：

--Hello here
```
## class的探索

在游戏开发的过程中，一般使用的都是面向对象的开发方式，但在脚本语言中是不存在类、方法以及属性这些概念的，所以要使用这个特性的第一步就是实现一个叫class的全局函数，通过调用这个函数来实现类的生成。但是cocos的这个class还是存在一些问题，接下来认真的解析和测试一下class。

[https://blog.csdn.net/mywcyfl/article/details/37706247](https://blog.csdn.net/mywcyfl/article/details/37706247)

```lua
function class(classname, super)
    local superType = type(super)
    local cls

    --如果父类既不是函数也不是table则说明父类为空
    if superType ~= "function" and superType ~= "table" then
        superType = nil
        super = nil
    end

    --如果父类的类型是函数或者是C对象
    if superType == "function" or (super and super.__ctype == 1) then
        -- inherited from native C++ Object
        cls = {}
        --如果父类是表则复制成员并且设置这个类的继承信息
        --如果是函数类型则设置构造方法并且设置ctor函数
        if superType == "table" then
            -- copy fields from super
            for k,v in pairs(super) do cls[k] = v end
            cls.__create = super.__create
            cls.super    = super
        else
            cls.__create = super
            cls.ctor = function() end
        end

        --设置类型的名称
        cls.__cname = classname
        cls.__ctype = 1

        --定义该类型的创建实例的函数为基类的构造函数后复制到子类实例
        --并且调用子数的ctor方法
        function cls.new(...)
            local instance = cls.__create(...)
            -- copy fields from class to native object
            for k,v in pairs(cls) do instance[k] = v end
            instance.class = cls
            instance:ctor(...)
            return instance
        end

    else
        --如果是继承自普通的lua表,则设置一下原型，并且构造实例后也会调用ctor方法
        -- inherited from Lua Object
        if super then
            cls = {}
            setmetatable(cls, {__index = super})
            cls.super = super
        else
            cls = {ctor = function() end}
        end

        cls.__cname = classname
        cls.__ctype = 2 -- lua
        cls.__index = cls

        function cls.new(...)
            local instance = setmetatable({}, cls)
            instance.class = cls
            instance:ctor(...)
            return instance
        end
    end

    return cls
end
```

**更新一版class**

```lua
local function _call(self, ...)
    return self:create(...)
end
function class(classname, ...)
    ---@class Class
    local cls = {__cname = classname}

    local supers = {...}
    for _, super in ipairs(supers) do
        local superType = type(super)
        assert(
            superType == "nil" or superType == "table" or superType == "function",
            string.format('class() - create class "%s" with invalid super class type "%s"', classname, superType)
        )
        if superType == "function" then
            assert(
                cls.__create == nil,
                string.format('class() - create class "%s" with more than one creating function', classname)
            )
            -- if super is function, set it to __create
            cls.__create = super
        elseif superType == "table" then
            cls.__supers = cls.__supers or {}
            cls.__supers[#cls.__supers + 1] = super
            if not cls.super then
                -- set first super pure lua class as class.super
                cls.super = super
            end
        else
            error(string.format('class() - create class "%s" with invalid super type', classname), 0)
        end
    end

    cls.__index = cls
    if not cls.__supers or #cls.__supers == 1 then
        local super = cls.super
        setmetatable(
            cls,
            {
                __call = _call,
                __index = super
            }
        )
    else
        setmetatable(
            cls,
            {
                __call = _call,
                __index = function(_, key)
                    local supers = cls.__supers
                    for i = 1, #supers do
                        local super = supers[i]
                        if super[key] then
                            return super[key]
                        end
                    end
                end
            }
        )
    end

    if not cls.ctor then
        -- add default constructor
        cls.ctor = function()
        end
    end
    cls.new = function(...)
        local instance
        if cls.__create then
            instance = cls.__create(...)
        else
            instance = {}
        end
        setmetatableindex(instance, cls)
        instance:ctor(...)
        return instance
    end
    cls.create = function(_, ...)
        return cls.new(...)
    end

    return cls
end
local setmetatableindex_
setmetatableindex_ = function(t, index)
    if type(t) == "userdata" then
        local meta = getmetatable(t)
        --printSL("对象元表：",meta)
        if not meta then
            meta = {}
            setmetatable(t, meta)
        end
        setmetatableindex_(meta, index)
    else
        local mt = getmetatable(t)
        if not mt then
            mt = {}
        end
        if not mt.__index then
            mt.__index = index
            setmetatable(t, mt)
        elseif mt.__index ~= index then
            setmetatableindex_(mt, index)
        end
    end
end
setmetatableindex = setmetatableindex_
```

