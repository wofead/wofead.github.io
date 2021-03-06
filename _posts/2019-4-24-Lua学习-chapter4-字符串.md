---
layout:     post
title:      Lua 学习 chapter4
subtitle:   chapter4
date:       2019-4-24
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 字符串常量
2. 强制类型转换
3. 字符串的标准库
4. Unicode编码

> I think you should do your best when you are work.

## 字符串常量

在lua中你可以使用一对双引号或者单引号来声明字符串常量。

它们两个是等价的，唯一区别在于，使用双引号声明的字符串中出现单引号，单引号不需要转义。在单引号中出现双引号也是一样的。

Lua中支持C语言的转义字符：

1. \q 响铃
2. \b 退格
3. \f 换页
4. \n 换行
5. \t 水平制表
6. \v 垂直制表
7. \\ 反斜杠
8. \" 双引号
9. \' 单引号

在使用长字符串或者多行字符串的时候可以使用,**[[ str]]**这样表示

```lua
page = [[
fsdfsd
fdsfsd
fsdf
sf
]]
```

## 强制类型转换

自动转换类型： 10 .. 2 --> 102 这个102就是字符串；"10" + 1 --> 11.0 算术运算的时候会把字符串自动转换成浮点数。

tonumber() 将字符串装换成数值，如果不能转换就会输出:nill,也可以使用这个来讲各种进制转换成十进制：

tonumber("100101",2)  --> 37 等等，如果所转换的数不是这个进制的数，就会返回nil。

tostring() 将数字转换成字符串。

## 字符串的标准库
1. string.rep("abc",3) -->"abcabcabc" 重复你次所需字符串
2. string.reverse("abc") --> "cba"  反转
3. string.lower("ABC") --> "abc" 小写
4. string.upper("abc") --> "ABC" 大写
5. string.sub("abcdefg",2,-2) -->"bcdef" 截取字符串，包含2和-2， 函数不会修改原字符串，2表示第二个，-2表示倒数第二个
6. string.byte("abc",1) -->97  输出第一个数的内部数值表示，默认不带参数输出第一个字符，带1个输出对应位置的谁。两个输出之间的。
7. string.char(99,98,97) --> cba 和byte相反
8. string.format("%d",10) -->10  %s字符串 %x(小写字母的十六进制) %X(大写字母的十六进制) %f浮点数
9. string.find("abcde",bcd) -->2 4  返回开始到结束
10. string.gsub("hello world","ll","..")  -->he..o world 1  将匹配的字符串替换 并输出替换的次数。

这个函数都可以通过语法糖来调用：

```lua
local str = "hello world"
str:gsub("ll","..")
```

## Unicode编码
lua5.3开始支持utf-8编码的Unicode字符串标准。在lua中。使用8个字节来编码字符，所以可以向操作其它字符串一样读写和存储utf-8的字符串。

但是在windows中文件名使用utf-16的编码。所以要么使用外部库，要么修改lua的语言标准库。

函数lower、byte、char以及format中的%c都不在适用，因为它们都是针对以一个字节字符，但是utf-8是8个字节，其它的函数以字节为单位索引而不是字符，所以适用。

1. utf8.len("fdasf")  -->5  返回制定字符串中utf-8的字符个数，如果包含无效字符，则会输出nil以及无效字符的位置。
2. utf8.char(114) --> r  可以带多个参数，把内部值转换成字符，这里即把Unicode的值，通过utf-8编码展现。
3. utf8.codepoint("r",1) -->114  类似于string.byte
4. utf8.offset(s,5) -->把字符位置转换成字节位置
5. string.sub("asdf",utf8.offset(s,-2)) --> df 
6. utf8.codes("fdsf")  遍历输出字符，for i,v in utf8.codes("fdsg") do print(i,v) end
