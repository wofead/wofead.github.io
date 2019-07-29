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