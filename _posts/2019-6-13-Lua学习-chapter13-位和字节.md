---
layout:     post
title:      Lua 学习 chapter13
subtitle:   chapter13
date:       2019-6-13
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 位和字节
2. 无符号的输出
3. 打包和解包二进制数据


> What is the cost of lies? It's not that we'll mistake them for the truth. The real danger is that if we hear enough lies, then we no longer recognize the truth at all. What can we do then? What else is left but to abandon even the hope of truth and content ourselves instaed with stories? In these stories, it doesn't matter who the heroes are. All we want to know is: who is to blame? No friends. Or, at least, not important ones.

## 位和字节

在lua中位运算和其它语言基本一致，包含按位与、按位或、按位异或、按位取反、逻辑右移以及逻辑左移。

```lua
print(string.format("%x", 0xff & 0xabcd))
print(string.format("%x", 0xff | 0xabcd))
print(string.format("%x", 0xff ~ 0xabcd))
print(string.format("%x", ~0xff))
print(string.format("%x", 0xff >> -12))
print(string.format("%x", 0xff << 12))
print(string.format("%x", 0xff << 80))
--[[
cd
abff
ab32
ffffffffffffff00
ff000
ff000
0
--]]

```

## 无符号的输出

整形表示中使用一个比特来存储符号位，所以用来表示最大的整数就是2^63 - 1,而不是64次方。

在lua中怎么输入一个无符号的整数呢？这么对一个整数进行无符号的比较呢？

我们可以使用string.format进行格式化输出。下面这两种方式就可以对一个数字进行无符号输出。如果比较两个无符号的话可以使用math.ult(x,y),来进行比较。

```lua
string.format("%u",x)
string.format("0x%X",x)
```

无符号的除法：

```lua
local function udiv(n, d)
    if d < 0 then
        --说明d大于2^63
        if math.ult(n, d) then
            return 0
        else
            return 1
        end
    end
    local q = ((n >> 1) // d) << 1 --现除以2，再除以d，再乘2
    local r = n - q * d --校正，n其实比d大，但是二分之n比d小的情况
    if not math.ult(r, d) then
        q = q + 1
    end
    return q
end
```

## 打包和解包二进制数据

函数string.pack会把值打包为二进制字符串，而unpack则从中提取这些值。 

在下面的例子里面，打包成什么格式的字符串取决于什么样的编码方式，不同的数据编码方式也不同。

```lua
s = string.pack("iii", 3, 6, 23)
string.unpack("iii",s)
```

整形：

1. b(char)
2. h(short)
3. i(int)
4. l(long)
5. j(lua语言中整数的大小)
6. in(n代表大小，7会产生7个字节的整形数）

字符串：

1. z(\0结尾的字符串)
2. cn(定长字符串使用选项cn,n表示被打包字符串的字节数)
3. sn(显式长度的字符串在存储的时候会在字符串前加上该字符安的长度，n是用来保存字符串长度的无符号整形数字大小）

浮点数：

1. f(单精度浮点数)
2. d(双精度浮点数)
3. n(用于lua语言浮点数)



