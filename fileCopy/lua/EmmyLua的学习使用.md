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
1. Postfix Completion
2. Annotations

> 一直都在使用Idea加上EmmyLua进行开发，但是当初别人为什么这样选择和选择的优点在哪里自己一概不知，直到前段时间翻到EmmyLua带有注解功能，才明白使用的好处。


## Postfix Completion

* par：将选择的内容阔起来
* var:进行相同名字的替换
* if
* if_not
* if_nil
* if_not_nil
* for
* for_p
* for_i
* return
* decrease
* increase

## 

* @class class declaration annotation  ： ---@class MY_TYPE[:PARENT_TYPE] [@comment]
* @type type annotation ： ---@type MY_TYPE[|OTHER_TYPE] [@comment]
* @alias 别名注解 ： ---@alias NEW_NAME TYPE
* @param parameter type annotation ： ---@param param_name MY_TYPE[|other_type] [@comment]
* @return function return type annotation ： ---@return MY_TYPE[|OTHER_TYPE] [@comment]
* @field field annotation ： ---@field [public|protected|private] field_name FIELD_TYPE[|OTHER_TYPE] [@comment]
* @generic generic annotation ： ---@generic T1 [: PARENT_TYPE] [, T2 [: PARENT_TYPE]]
* @vararg 不定参数注解 ： ---@vararg TYPE
* @language language injection ： ---@language LANGUAGE_ID
* array type ： ---@type MY_TYPE[]
* table type ： ---@type table<KEY_TYPE, VALUE_TYPE>  eg: ---@type table<string, Car>
* function types : ---@type fun(param:MY_TYPE):RETURN_TYPE
* 字面量类型 :---@alias Handler fun(type: string, data: any):void  ---@param event string | "'onClosed'" | "'onData'"  ---@param handler Handler | "function(type, data) print(data) end"
* @see references : ---@Emmy#sayHello


