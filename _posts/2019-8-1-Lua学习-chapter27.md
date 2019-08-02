---
layout:     post
title:      Lua 学习 chapter27
subtitle:   chapter27
date:       2019-8-1
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. lua堆栈操作

> 只有疯狂过，你才知道自己究竟能不能成功。

## lua堆栈操作
针对于lua堆栈的操作。
```c
static void stackDump(lua_State* L) {
	int i;
	int top = lua_gettop(L);
	for (int i = 0; i <= top; i++)
	{
		int t = lua_type(L, i);
		switch (t)
		{
		case LUA_TSTRING: {
			printf("%s", lua_tostring(L, i));
			break;
		}
		case LUA_TBOOLEAN: {
			printf(lua_toboolean(L, i) ? "true" : "false");
			break;
		}
		case LUA_TNUMBER: {
			printf("%g", lua_tonumber(L, i));
			break;
		}
		default:
			printf("%s", lua_typename(L, t));
			break;
		}
		printf(" ");
	}
	printf("\n");
}

static void test() {
	lua_State* L = luaL_newstate();

	lua_pushboolean(L, 1);
	lua_pushnumber(L, 10);
	lua_pushnil(L);
	lua_pushstring(L, "hello");
	stackDump(L);//true 10 nil hello
	lua_pushvalue(L, -4);//将指定索引的值压到栈顶
	stackDump(L); //true 10 nil hello true
	lua_replace(L, 3);//pop栈顶元素，并将pop的值设置到指定索引
	stackDump(L);// true 10 true hello
	lua_settop(L, 6);//设置栈中元素个数,0的话清空栈，大于原来个数补nil
	stackDump(L);//true 10 true hello nil nil
	lua_rotate(L, 3, 1);//旋转指定位置的值，并把栈顶元素补充过来
	stackDump(L);//true 10 nil true hello nil
	lua_remove(L, -3);//移除指定位置的值
	stackDump(L);//true 10 nil hello nil
	lua_close(L);
}
```


