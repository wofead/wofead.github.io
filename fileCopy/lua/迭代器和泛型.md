---
layout:     post
title:      Lua 学习 chapter18
subtitle:   chapter18
date:       2019-7-29
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 迭代器
2. 泛型for的语法
3. 无状态迭代器
4. 按顺序遍历表
5. 迭代器的真实含义

> Continue, come on.

## 迭代器
迭代去就是指for i,k in pairs(values) do  和 ipairs。

```lua
local function iter (t, i)
	i = i + 1
	local v = t[i]
	if v then
		return i,v
	end
end

function ipairs(t)
	return iter, t, 0
end

function pairs(t)
	return next, t, nil
end
```

对于一个迭代器最重要的就是，索引值的改变，这个值需要每次进入其中还存在，要实现这个功能就需要闭包了。一个闭包就是一个可以访问其自身环境中的一个或者多个局部变量的函数。这些变量将连续调用过程中的值并将其保存在闭包中，从而使得闭包能够自己迭代器所处的位置。当然，要创建一个新的闭包，我们还需要创建非局部变量。因此一个闭包结构通常涉及两个函数：闭包本身和一个用于创建该闭包及其封装变量的工厂。

```lua
function valuse(t)
	local i =0
	return function() i = i + 1;return t[i] end
end

t = {1,2,3}
iter = values(t) --迭代器（工厂）
while true do
	local element = iter()
	if element == nil then break end
	print(element)
end

--泛型for的使用
for element in values(t) do
	pirnt(element)
end
```

泛型for为一次迭代循环做了所有的记录工作：它在内部保存了迭代函数，因此不需要变量iter；它在每次做新的迭代时都会再次调用迭代器，并在迭代器返回nil时结束循环。

## 泛型for的语法

泛型for在循环过程中在其内部保存了迭代函数。实际上，泛型for保存了三个值：一个迭代函数，一个不可变状态和一个控制变量。

```lua
for var-list in exp-list do
	body
end
```

var-list是由一个或多个变量名组成的列表，以逗号分隔；exp-list是一个或多个表达式组成的列表，同样以逗号分隔。通常，表达式列表只有一个元素，即一句对迭代器工厂的调用。例如，在如下的代码中，变量列表式是k,v，表达式列表只有一个元素paris(t)：

```lua
for k,v in pairs(t) do print((k,v) end
```

我们把变量列表中的第一个（或唯一的）变量称为控制变量，其值在循环过程中永远不会是nil，因为当其值为nil时循环就结束了。

for做的第一件事情是对in后面的表达式求值。这些表达式应该返回三个值供for保存：迭代函数、不可变状态和控制变量的初始化值。类似于多重赋值，只有最后一个表达式能够产生不止一个值，表达式列表的结果只会保存三个，多余的值会被丢弃，不足则以nil补齐。

上面的步骤完成之后，for使用不可变状态和控制变量作为参数来调用迭代函数。

```lua
for var_1, ... , var_n in explist do block end

do 
	local _f,_s,_var = explist
	while true do
		local var_1, ..., var_n = _f(_s,_var)
		_var = var_1
		if _var == nil then break end
		block
	end
end
```

上面的代码，假设迭代函数为f，不可变状态为s，控制变量的初始值为a0，那么在循环中控制变量的值依次为a1 = f(s,a0),a2 = f(s,a1),以此类推，知道ai为nil。

## 无状态迭代器

是一种自身不保存任何状态的迭代器。因此，可以在多循环中使用同一个无状态的迭代器，从而避免创建新的闭包开销。

```lua
--ipairs

local function iter(t,i)
	i = i + 1
	local v = t[i]
	if v then
		return i,v
	end
end

function ipairs (t)
	return iter,t,0
end
```

当调用for循环中的ipairs(t)时，会返回迭代函数iter、不可变状态表t和控制变量的初始值0.

函数pairs与函数ipairs类似，也用于遍历一个表中的所有元素。不同的是，函数pairs的迭代函数是lua语言中的一个基恩函数next：

```lua
function pairs (t)
	return next,t,nil
end
```

在调用next(t,k)时，k是表t的一个键，该函数会以随机次序返回表中的下一个键及k对应的值。

## 按顺序遍历表

使用ipairs来遍历，因为后者会根据1,2,3等有序的键值遍历表。

## 迭代器的真实含义

迭代器有点像生成器，因为迭代过程都是由for完成的，但是其他语言好像也是这样子的，java和c++。