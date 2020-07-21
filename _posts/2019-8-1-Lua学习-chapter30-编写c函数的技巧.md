---
layout:     post
title:      Lua 学习 chapter30 编写c函数的技巧
subtitle:   扩展应用
date:       2019-8-1
author:     Jow
header-img: img/lua-study.jpg
catalog: 	 true 
tags:
    - Lua
    - lua program dedign

---

### 目录
1. 数组操作
2. 字符串操作
3. 在c函数中保存状态


> 生活总需要一点仪式感，然后慢慢的像那个趋向完美的自己靠近。

## 数组操作

Lua中的数组就是以特殊的方式使用边。像lua_setttable and lua_gettable这种用来操作的通用函数，也可以用于操作数组，不过C API为使用整数索引的表访问和更新提供了专门的函数：

```c
void lua_geti (lua_State *L, int index, int key);
void lua_seti (lua_State *L, int index, int key);
```
lua_geti相当于：

```c
lua_pushnumber(L, key);
lua_gettable(L,t);
```

lua_seti相当于：

```c
lua_pushnumber(L, key);
lua_insert(L, -2);
lua_settable(L,t);
```

eg:函数map，该函数对数组中的所有元素调用一个指定的函数，然后用此函数返回的结果替换掉对应的数组元素。

```c
int l_map(lua_State* L) {
	int i, n;
	luaL_checktype(L, 1, LUA_TTABLE); //确保指定的参数具有指定的类型
	luaL_checktype(L, 2, LUA_TFUNCTION);
	n = luaL_len(L, 1); //类似长度运算符
	for (int i = 0; i < n; i++)
	{
		lua_pushvalue(L, 2);
		lua_geti(L, 1, i);
		lua_call(L, 1, 1); //类似以pcall但是会传播错误，而不是返回错误码
		lua_seti(L, 1, i);
	}
	return 0;
}
```

## 字符串操作

当c函数接收到一个lua字符串为参数是，必须遵守两条规则，在使用字符串期间不能从栈中将其弹出，而且不应该修改字符串。

标准API为两种最用的字符串操作提供了支持，即字符串提取和字符串连接。要提取子串，那么基本的操作lua_pushlstring可以获取字符串长度作为额外的参数。因此，如果要把字符串s从i到j(include)的子串传递给lua：

```c
lua_pushlstring(L, s + i, j - i +1);
```

eg:函数根据分隔符来分隔字符串：

```lua
static int l_split(lua_State* L) {
	const char* s = luaL_checkstring(L, 1);
	const char* sep = luaL_checkstring(L, 2);
	const char* e;
	int i = 1;
	lua_newtable(L);

	while ((e = strchr(s, *sep)) != NULL)
	{
		lua_pushlstring(L, s, e - s);
		lua_rawseti(L, -2, i++);
		s = e + 1;
	}
	lua_pushstring(L, s);
	lua_rawseti(L, -2, i);
	return 1;
}
```

## 在c函数中保存状态

C API提供了两个类似的地方来存储非局部数据，即注册表(registry)和上值(upvalue).

**注册表**

注册表是一张只能被C代码访问的全局表。通常情况下，我们使用注册表来存储多个模块间的共享数据。

注册表总是位于伪索引LUA_REGISTRYINDEX中。伪索引就像是一个栈中的索引，但是它所关联的值不在栈中。lua API中大多数接受索引作为参数的函数也能将伪索引作为参数，像lua_remove和lua_insert这种操作栈本身的函数除外。eg:获取注册表中键为“key”的值，可以使用如下的调用。

```c
lua_getfield(L, LUA_REGISTRYINDEX, "Key");

static char key = 'k'
lua_pushstring(L, mystr);
lua_rawsetp(L,LUA_REGISTRYINDEX,(void *) &key);//设置值到注册表中

lua_rawgetp(L,LUA_REGISTRYINDEX,(void *) &key);//从注册表中取值
mystr = lua_tostring(L,-1);
```

**upvalue**

如果函数f2定义在函数f1中，那么f2为f1的内嵌函数，f1位f2的外包函数，外包和内嵌都具有传递性，f2的内嵌一定是f1的内嵌，f1的外包一定也是f2的外包。

内嵌函数可以访问外包函数的局部变量，这些局部变量被称为该内嵌函数的外部局部变量或者说**upvalue**。