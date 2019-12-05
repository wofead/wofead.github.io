---
layout:     post
title:      Lua文件的架子啊
subtitle:   unity game
date:       2019-12-5
author:     Jow
header-img: img/about-bg-walle.jpg
catalog: 	 true 
tags:
    - ddt

---

### 目录
1. lua文件索引的建立
2. lua文件的加载

> Sometimes, you should know what you want, and what's your aim is. Then try your best to chase pursue it.

## lua文件索引的建立

在之前的项目中，发现在游戏刚启动的时候会把所有的lua脚本文件都加载到内存中，这样就会导致一个问题，有可能这个文件没有使用，但是它还是被加载内存中了，而且进入初次进入游戏的时间会很长，因为需要将所有lua脚本文件加载到内存中，所以就想，为什么不像c或者java那样，我们用到什么文件就加载什么文件，，这样不就不需要刚启动游戏的时候就加载所有文件了。

首先我们需要对所有的lua文件建立索引，这样当我们需要使用那个lua文件的时候才能去加载它，索引的方式为文件名对应的文件目录。现在假设我们的所有lua问价你都在Src目录下，那么我们开始遍历Src目录先的所有文件以及子目录下的lua文件，判断是否为lua文件的依据就是后缀名是否为".lua"。

接下来我们生成两个全局表，一个是LuaClassList，一个是LuaClass。格式如下：

```lua
LuaClassList = 
{

	app                                     = "App",
	debug                                   = "Debug",
	gameconfig                              = "GameConfig",
	main                                    = "Main",
	animatorcomponent                       = "Common.Component.AnimatorComponent"
}

LuaClass = 
{

    ---@type App
	App                                     = nil,
    ---@type Debug
	Debug                                   = nil,
    ---@type GameConfig
	GameConfig                              = nil,
    ---@type Main
	Main                                    = nil,
    ---@type AnimatorComponent
	AnimatorComponent                       = nil
}
```

这两个类一个是用来找到require的地址的，一个是用来索引到具体的类文件中的。

## lua文件的加载

既然我们选择动态加载，那么我们以后访问类和创建类都应该通过LuaClass（除了一些在游戏初始已经加载的常用的核心类），这样的话，就可以根据需要的类来选择性的将lua脚本加载到内存中。

```lua
require "Core.LuaClassList"

local function initLuaClass()
	local metaTable={}
	metaTable.__index = function(t,key)
		local keyLow = string.lower(key)
		if LuaClassList[keyLow] == nil then
			error("Error: No Lua Script:" .. key)
			return nil
		end
		t[key] = require(LuaClassList[keyLow])
		return t[key]
	end
	setmetatable(LuaClass, metaTable)
end

-- 执行初始化函数, 给LuaClass设置元表
initLuaClass()

```

由于我们有时候会在不重新启动游戏的情况下重新加载新的lua脚本，所有我们有时候需要清理已经被缓存在内存中的lua脚本，函数如下：

```lua
function LuaClass.clearLua(luaObj)
	local luaClassName
	for key, value in pairs(LuaClass) do
		-- body
		if value == luaObj then
				luaClassName = key
				local luaNameLow = string.lower(key)
				package.loaded[LuaClassList[luaNameLow]] = nil
				LuaClass[key] = nil
				break
		end
	end
	-- return LuaClass[luaClassName]
	
end
```