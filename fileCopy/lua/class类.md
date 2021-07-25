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

## **警惕copy导致的数据冗余，和元表设置导致的深度索引**

lua里面实现class机制，比较关键的地方就是class的继承机制，以及class实例化的过程。

class继承机制的关键是怎么让子类拥有父类的方法集：

1. 完全使用setmetatable()的方式实现，每继承一次，都把父类的方法集设置为子类方法集的metatable。这样做应该最符合class的继承的思想，尽可能的复用逻辑，而且可以做到动态更新父类的方法集后，所有子类都会更新。但随着class继承的层次加深，会生成一个复杂的class方法集table(n层的metatable)，毕竟metatable是有额外开销的，所以这个方法不一定是最完美的方案。
2. 将父类方法集copy到子类方法集来实现继承，即每当定义一个新的子类时，都将父类中方法集完整copy到子类方法集中。这种方法避免了使用metatable()带来的额外开销，但却造成了一些数据冗余(其实并不多)，并丧失了父类更新子类也会自动更新的特性。
3. **方案2的改进版本(也就是云风这里使用的比较强悍的方式:P pf),即同样是采用copy父类方法集的方案，但却改进了copy的机制。将原本在class定义期执行的方法集copy工作转移到实例的运行期间，采用copy-on-use(等同于copy-on-write的设计思路)的方式，在子类实例用到父类的某个方法时，才将其copy到子类的方法集中。由于这种copy只发生一次，而且不见得子类会用到父类方法集中的所有内容(事实如此)，所以这个方案相对于方案2来说减少了冗余数据，但却几乎没有增加任何额外开销。**



**class实例化关键是实例如何享有class的方法集**：

1. **最烂的方式，方法集copy，即class实例化时，将class的方法集直接copy给实例的数据table。**这样的好处就是每个实例创建后，外界除非直接操作实例的数据table，否则其它行为都不会影响到这个实例的所有属性和特征(也许可以满足某些特殊的需求吧),同时也省掉了一次metatable的查找开销。缺点很明显，实例化过程低效，而且产生大量的冗余信息(或者这里也采用copy-on-use的思想? :p)。
2. **采用将class方法集设置为实例的metatable的方式，使实例享有class的方法集(这要求实例的数据类型必须可以拥有自己的metatable，即只能是table或userdata)。**这个方案更优雅一些，而且符合class的共享思想并且实例化开销很小，应该是实现实例化的最佳方案了。在实现子类的初始化函数时，一般的思路都是先生成一个父类的实例，再强制将当前子类的方法集设置为这个实例的metatable。



云风的class

```lua
local _class={}
 
function class(super)
	local class_type={}
	class_type.ctor=false
	class_type.super=super
	class_type.new=function(...) 
			local obj={}
			do
				local create
				create = function(c,...)
					if c.super then
						create(c.super,...)
					end
					if c.ctor then
						c.ctor(obj,...)
					end
				end
 
				create(class_type,...)
			end
			setmetatable(obj,{ __index=_class[class_type] })
			return obj
		end
	local vtbl={}
	_class[class_type]=vtbl
 
	setmetatable(class_type,{__newindex=
		function(t,k,v)
			vtbl[k]=v
		end
	})
 
	if super then
		setmetatable(vtbl,{__index=
			function(t,k)
				local ret=_class[super][k]
				vtbl[k]=ret
				return ret
			end
		})
	end
 
	return class_type
end
```



```lua
base_type=class()       -- 定义一个基类 base_type

function base_type:ctor(x)  -- 定义 base_type 的构造函数
    print("base_type ctor")
    self.x=x
end

function base_type:print_x()    -- 定义一个成员函数 base_type:print_x
    print(self.x)
end

function base_type:hello()  -- 定义另一个成员函数 base_type:hello
    print("hello base_type")
end
```

以上是基本的 class 定义的语法，完全兼容 lua 的编程习惯。我增加了一个叫做 ctor 的词，作为构造函数的名字。

下面看看怎样继承： 

```lua
test=class(base_type)	-- 定义一个类 test 继承于 base_type
 
function test:ctor()	-- 定义 test 的构造函数
	print("test ctor")
end
 
function test:hello()	-- 重载 base_type:hello 为 test:hello
	print("hello test")
end
```

