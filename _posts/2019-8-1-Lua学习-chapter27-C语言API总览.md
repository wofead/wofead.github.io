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
1. 前言
2. 第一个示例
2. lua堆栈操作
3. 处理应用代码中的错误
4. 内存分配

> 只有疯狂过，你才知道自己究竟能不能成功。

## 前言

lua是一种嵌入式语言，这就意味着lua并不是一个独立运行的应用，而是一个库，它可以链接到其它应用程序，将lua的功能融入到这些应用。

由于lua存在解释器（可执行的lua），所以我们可以独立的使用它，这个解释器是由lua标准库实现的独立解释器，它负责与用户交互，将用户的文件和字符串传递给lua标准库，由标准库完成主要工作。

因为能被当作库来扩展某个应用程序，所以lua是一个嵌入式语言。同时，使用了lua语言的程序也可以在lua环境中注册新的函数，比如用c语言实现的函数，从而增加一些无法直接用lua语言编写的功能，因此lua也是一种可扩展的语言。

上述的两种对lua语言的定位，分别对应c语言和lua语言之间的两种交互方式。在第一种形式中，c语言拥有控制权，而lua语言被用作库，这种交互形式中c代码被称为应用代码。在第二种中，lua语言拥有控制权，而c语言被用作库，因此c代码被称为库代码。应用代码和库代码都是用相同的API与lua语言通信，这些API被称为C API。

C API是一个函数、常量和类型组成的集合，有了它，c语言代码就能与lua语言交互。**C API包括读写lua全局变量的函数、调用lua函数的函数、运行lua代码段的函数以及注册c函数(以便于其后可被lua代码调用）的函数等**。通过调用C API，C代码几乎可以做lua代码能够做的所有事情。

C API遵循C语言的操作模式，与lua模式有很大的区别。所以在c的时候可能会抛弃易用性，但是在效率上，c代码可能会高一些。



### C++数据传递到虚拟栈中

 push functions (C -> stack)------>C空间与虚拟栈之间的操作.

C语言向虚拟栈中压人符合Lua数据类型(nil,number,string,table,function,userdata,thread)的变量.

```c
LUA_API void (lua_pushnil) (lua_State *L);
LUA_API void (lua_pushnumber) (lua_State *L, lua_Number n);--double,float
LUA_API void (lua_pushinteger) (lua_State *L, lua_Integer n);--int,long
LUA_API void (lua_pushlstring) (lua_State *L, const char *s, size_t l);--任意的字符串(char*类型，允许包含'\0'字符)
LUA_API void (lua_pushstring) (lua_State *L, const char *s);--以'\0'结束的字符串（const char*）
LUA_API const char *(lua_pushvfstring) (lua_State *L, const char *fmt,va_list argp);
LUA_API const char *(lua_pushfstring) (lua_State *L, const char *fmt, ...);
LUA_API void (lua_pushcclosure) (lua_State *L, lua_CFunction fn, int n);
LUA_API void (lua_pushboolean) (lua_State *L, int b);
LUA_API void (lua_pushlightuserdata) (lua_State *L, void *p);
LUA_API int (lua_pushthread) (lua_State *L);

```

**Lua中的字符串不是以零为结束符的；它们依赖于一个明确的长度，因此可以包含任意的二进制数据。**将字符串压入串的正式函数是lua_pushlstring，它要求一个明确的长度作为参数。对于以零结束的字符串，你可以用lua_pushstring（它用strlen来计算字符串长度）。

Lua从来不保持一个指向外部字符串（或任何其它对象，除了C函数——它总是静态指针）的指针。对于它保持的所有字符串，Lua要么做一份内部的拷贝要么重新利用已经存在的字符串。因此，一旦这些函数返回之后你可以自由的修改或是释放你的缓冲区。

## 虚拟栈数据传递到C++中

access functions (stack -> C)------>C空间与虚拟栈之间的操作。

API提供了一套lua_is*函数来检查一个元素是否是一个指定的类型，*可以是任何Lua类型。lua_isnumber和lua_isstring函数不检查这个值是否是指定的类型，而是看它是否能被转换成指定的那种类型。

```c
UA_API int (lua_isnumber) (lua_State *L, int idx);
LUA_API int (lua_isstring) (lua_State *L, int idx);
LUA_API int (lua_iscfunction) (lua_State *L, int idx);
LUA_API int (lua_isuserdata) (lua_State *L, int idx);
LUA_API int (lua_type) (lua_State *L, int idx);
LUA_API const char *(lua_typename) (lua_State *L, int tp);
```

虚拟栈上的lua类型的数据转换成符合C++语言数据类型的数举，int, double, char*, fuction, void, struct/class(userdata),指针。

```c
LUA_API lua_Number (lua_tonumber) (lua_State *L, int idx);
LUA_API lua_Integer (lua_tointeger) (lua_State *L, int idx);
LUA_API int (lua_toboolean) (lua_State *L, int idx);
LUA_API const char *(lua_tolstring) (lua_State *L, int idx, size_t *len);
LUA_API size_t (lua_objlen) (lua_State *L, int idx);
LUA_API lua_CFunction (lua_tocfunction) (lua_State *L, int idx);
LUA_API void *(lua_touserdata) (lua_State *L, int idx);
LUA_API lua_State *(lua_tothread) (lua_State *L, int idx);
LUA_API const void *(lua_topointer) (lua_State *L, int idx);
```

**lua_toboolean、lua_tonumber和lua_strlen返回0，其他函数返回NULL。由于ANSI C没有提供有效的可以用来判断错误发生数字值，所以返回的0是没有什么用处的。对于其他函数而言，我们一般不需要使用对应的lua_is\*函数：我们只需要调用lua_is\*，测试返回结果是否为NULL即可。**

**Lua_tostring函数返回一个指向字符串的内部拷贝的指针。你不能修改它（使你想起那里有一个const）**。只要这个指针对应的值还在栈内，Lua会保证这个指针一直有效。**当一个C函数返回后，Lua会清理他的栈，所以，有一个原则：永远不要将指向Lua字符串的指针保存到访问他们的外部函数中。**

**lua_tostring返回的字符串结尾总会有一个字符结束标志0，但是字符串中间也可能包含0，lua_strlen返回字符串的实际长度**。

### Lua数据传递到虚拟栈中

get functions (Lua -> stack)------>Lua空间与虚拟栈之间的操作

```c
LUA_API void (lua_gettable) (lua_State *L, int idx);//把 t[k] 值压入堆栈，这里的 t 是指有效索引 index 指向的值，而 k 则是栈顶放的值。这个函数会弹出堆栈上的 key （把结果放在栈上相同位置）。在 Lua 中，这个函数可能触发对应 "index" 事件的元方法
lua_getglobal(L, "mytable") <== push mytable
lua_pushnumber(L, 1) <== push key 1
lua_gettable(L, -2) <== pop key 1, push mytable[1]
 
LUA_API void (lua_getfield) (lua_State *L, int idx, const char *k);//把 t[k] 值压入堆栈，这里的 t 是指有效索引 index 指向的值。在 Lua 中，这个函数可能触发对应 "index" 事件的元方法
lua_getglobal(L, "mytable") <== push mytable
lua_getfield(L, -1, "x") <== push mytable["x"]，作用同下面两行调用
--lua_pushstring(L, "x") <== push key "x"
--lua_gettable(L,-2) <== pop key "x", push mytable["x"]
 
LUA_API void (lua_rawget) (lua_State *L, int idx);//类似于 Lua_gettable，但是作一次直接访问（不触发元方法）。
LUA_API void (lua_rawgeti) (lua_State *L, int idx, int n);//把 t[n] 的值压栈，这里的 t 是指给定索引 index 处的一个值。这是一个直接访问；就是说，它不会触发元方法。
lua_getglobal(L, "mytable") <== push mytable
lua_rawgeti(L, -1, 1) <== push mytable[1]，作用同下面两行调用
--lua_pushnumber(L, 1) <== push key 1
--lua_rawget(L,-2) <== pop key 1, push mytable[1]
 
LUA_API void (lua_createtable) (lua_State *L, int narr, int nrec);//创建一个新的空 table 压入堆栈。这个新 table 将被预分配 narr 个元素的数组空间以及 nrec 个元素的非数组空间。当你明确知道表中需要多少个元素时，预分配就非常有用。如果你不知道，可以使用函数 Lua_newtable。
 
LUA_API void *(lua_newuserdata) (lua_State *L, size_t sz);//这个函数分配分配一块指定大小的内存块，把内存块地址作为一个完整的 userdata 压入堆栈，并返回这个地址。
userdata 代表 Lua 中的 C 值。完整的 userdata 代表一块内存。它是一个对象（就像 table 那样的对象）：你必须创建它，它有着自己的元表，而且它在被回收时，可以被监测到。一个完整的 userdata 只和它自己相等（在等于的原生作用下）。
当 Lua 通过 gc 元方法回收一个完整的 userdata 时， Lua 调用这个元方法并把 userdata 标记为已终止。等到这个 userdata 再次被收集的时候，Lua 会释放掉相关的内存。
LUA_API int (lua_getmetatable) (lua_State *L, int objindex);//把给定索引指向的值的元表压入堆栈。如果索引无效，或是这个值没有元表，函数将返回 0 并且不会向栈上压任何东西。
LUA_API void (lua_getfenv) (lua_State *L, int idx);//把索引处值的环境表压入堆栈。
```



### 虚拟栈数据传递到Lua空间中

set functions (stack -> Lua)------>Lua空间与虚拟栈之间的操作

```c
LUA_API void (lua_settable) (lua_State *L, int idx);作一个等价于 t[k] = v 的操作，这里 t 是一个给定有效索引 index 处的值， v 指栈顶的值，而 k 是栈顶之下的那个值。这个函数会把键和值都从堆栈中弹出。和在 Lua 中一样，这个函数可能触发 "newindex" 事件的元方法。eg:
lua_getglobal(L, "mytable") <== push mytable
lua_pushnumber(L, 1) <== push key 1
lua_pushstring(L, "abc") <== push value "abc"
lua_settable(L, -3) <== mytable[1] = "abc", pop key & value
 
LUA_API void (lua_setfield) (lua_State *L, int idx, const char *k);//做一个等价于 t[k] = v 的操作，这里 t 是给出的有效索引 index 处的值，而 v 是栈顶的那个值。这个函数将把这个值弹出堆栈。跟在 Lua 中一样，这个函数可能触发一个 "newindex" 事件的元方法。eg:
lua_getglobal(L, "mytable") <== push mytable
lua_pushstring(L, "abc") <== push value "abc"
lua_setfield(L, -2, "x") <== mytable["x"] = "abc", pop value "abc"
 
LUA_API void (lua_rawset) (lua_State *L, int idx);//类似于 Lua_settable，但是是作一个直接赋值（不触发元方法）。
LUA_API void (lua_rawseti) (lua_State *L, int idx, int n);//等价于 t[n] = v，这里的 t 是指给定索引 index 处的一个值，而 v 是栈顶的值。函数将把这个值弹出栈。赋值操作是直接的；就是说，不会触发元方法。
lua_getglobal(L, "mytable") <== push mytable
lua_pushstring(L, "abc") <== push value "abc"
lua_rawseti(L, -2, 1) <== mytable[1] = "abc", pop value "abc"
 
LUA_API int (lua_setmetatable) (lua_State *L, int objindex);//把一个 table 弹出堆栈，并将其设为给定索引处的值的 metatable 。
LUA_API int (lua_setfenv) (lua_State *L, int idx);//从堆栈上弹出一个 table 并把它设为指定索引处值的新环境。如果指定索引处的值即不是函数又不是线程或是 userdata ， Lua_setfenv 会返回 0 ，否则返回 1 。
```

## 虚拟栈基本操作

basic stack manipulation--基础栈操作

```c
LUA_API int (lua_gettop) (lua_State *L);//获取栈的高度，它也是栈顶元素的索引。注意一个负数索引-x对应于正数索引gettop-x+1
LUA_API void (lua_settop) (lua_State *L, int idx);//设置栈的高度。如果开始的栈顶高于新的栈顶，顶部的值被丢弃。否则，为了得到指定的大小这个函数压入相应个数的空值（nil）到栈上。特别的，lua_settop(L,0)清空堆栈。
LUA_API void (lua_pushvalue) (lua_State *L, int idx);//压入堆栈上指定索引的一个抟贝到栈顶,【增加一个元素到栈顶】
LUA_API void (lua_remove) (lua_State *L, int idx);//移除指定索引位置的元素，并将其上面所有的元素下移来填补这个位置的空白，【删除了一个元素】
LUA_API void (lua_insert) (lua_State *L, int idx);//移动栈顶元素到指定索引的位置，并将这个索引位置上面的元素全部上移至栈顶被移动留下的空隔，【没有增加一个元素，移动了元素的位置】
LUA_API void (lua_replace) (lua_State *L, int idx);//从栈顶弹出元素值并将其设置到指定索引位置，没有任何移动操作。【删除了一个元素，替换掉指定的元素】
LUA_API int (lua_checkstack) (lua_State *L, int sz);//检查栈上是否有能插入n个元素的空间;没有返回0
LUA_API void (lua_xmove) (lua_State *from, lua_State *to, int n);//将一个堆栈上的从栈顶起的n个元素 移到另一个堆栈上
//lua的堆栈保持着后进先出的原则。如果栈开始于 10 20 30 40 50*（自底向上；`*´ 标记了栈顶），那么：
lua_pushvalue(L, 3)    --> 10 20 30 40 50 30*
lua_pushvalue(L, -1)   --> 10 20 30 40 50 30 30*
lua_remove(L, -3)      --> 10 20 30 40 30 30*
lua_remove(L, 6)      --> 10 20 30 40 30*
lua_insert(L, 1)      --> 30 10 20 30 40*
lua_insert(L, -1)      --> 30 10 20 30 40* (no effect)
lua_replace(L, 2)      --> 30 40 20 30*
lua_settop(L, -3)      --> 30 40*
lua_settop(L, 6)      --> 30 40 nil nil nil nil*
```

## 宏定义

```c
//some useful macros
#define lua_pop(L,n) lua_settop(L, -(n)-1)
#define lua_newtable(L) lua_createtable(L, 0, 0)
#define lua_register(L,n,f) (lua_pushcfunction(L, (f)), lua_setglobal(L, (n)))
#define lua_pushcfunction(L,f) lua_pushcclosure(L, (f), 0)
#define lua_strlen(L,i) lua_objlen(L, (i))
 
#define lua_isfunction(L,n) (lua_type(L, (n)) == LUA_TFUNCTION)
#define lua_istable(L,n) (lua_type(L, (n)) == LUA_TTABLE)
#define lua_islightuserdata(L,n) (lua_type(L, (n)) == LUA_TLIGHTUSERDATA)
#define lua_isnil(L,n) (lua_type(L, (n)) == LUA_TNIL)
#define lua_isboolean(L,n) (lua_type(L, (n)) == LUA_TBOOLEAN)
#define lua_isthread(L,n) (lua_type(L, (n)) == LUA_TTHREAD)
#define lua_isnone(L,n) (lua_type(L, (n)) == LUA_TNONE)
#define lua_isnoneornil(L, n) (lua_type(L, (n)) <= 0)
 
#define lua_pushliteral(L, s) lua_pushlstring(L, "" s, (sizeof(s)/sizeof(char))-1)
#define lua_setglobal(L,s) lua_setfield(L, LUA_GLOBALSINDEX, (s))
#define lua_getglobal(L,s) lua_getfield(L, LUA_GLOBALSINDEX, (s))
#define lua_tostring(L,i) lua_tolstring(L, (i), NULL)
 
 
//compatibility macros and functions
#define lua_open() luaL_newstate()
#define lua_getregistry(L) lua_pushvalue(L, LUA_REGISTRYINDEX)
#define lua_getgccount(L) lua_gc(L, LUA_GCCOUNT, 0)
#define lua_Chunkreader lua_Reader
#define lua_Chunkwriter lua_Writer
```



## 第一个示例

```c
#include<stdio.h>
#include<string.h>

extern "C" {
#include <lua.h>
#include <lauxlib.h>
#include <lualib.h>
}
int main(void)
{
	char buff[256];
	int error;
	lua_State* L = luaL_newstate();//打开lua
	luaL_openlibs(L);//打开标准库

	while (fgets(buff, sizeof(buff), stdin) != NULL) {
		error = luaL_loadstring(L, buff) || lua_pcall(L, 0, 0, 0);
		if (error)
		{
			fprintf(stderr, "%s\n", lua_tostring(L, -1));
			lua_pop(L, 1);//从栈中弹出错误信息
		}
	}
	lua_close(L);
	return 0;
}

-- 测试输入
qewqr
[string "qewqr..."]:2: syntax error near <eof>
print("hello")
hello
local t = {}
t.a = 5
[string "t.a = 5..."]:1: attempt to index a nil value (global 't')
t= {}
t.a = 5
print(t.a)
5
```

接下来我们来熟悉一下各种头文件的提供了那些函数，其中头文件**lua.h**声明了lua提供的**基础函数**，其中包括创建新的lua环境的函数，调用lua函数的函数、读写环境中的全局变量的函数，以及注册供lua语言调用的新函数的函数。**lua.h中声明的所有的内容都有一个前缀lua_**(eg:lua_pcall).

头文件**lauxlib.h**声明了**辅助库**(auxiliary library, auxlib)所提供的的**函数**，其中所有的声明均以**luaL_开头**（eg:luaL_loadstring)。**辅助库使用lua.h提供的基础API来提供更高层次的抽象，特别是对标准库用到的相关机制进行抽象**。

lua标准库没有定义任何c语言全局变量，它将**所有的状态都保存在动态的结构体lua_State**中，**lua中的所有函数都接收一个指向该结构的指针作为参数**。这种设计使得lua是可重入的，并且可以直接用于编写所线程代码。

函数**luaL_newstate用于创建一个新的lua状态**。当它创建一个新的状态时，新的环境中没有包含预定一个的函数，甚至连print都没有。为了保持lua语言的精炼，所有的标准库都被组织成不同的包，这样我们在不需要使用某些包的时候可以忽略它们。头文件lualib.h中声明了用于打开这些库的函数。**函数luaL_openlibs用于打开所有的标准库**。



**Lua以一个严格的LIFO规则（后进先出；也就是说，始终存取栈顶）来操作栈。**当你调用Lua时，它只会改变栈顶部分。你的Ｃ代码却有更多的自由；更明确的来讲，你可以查询栈上的任何元素，甚至是在任何一个位置插入和删除元素。栈中的元素通过索引值进行定位，其中栈顶是-1，栈底是1。 **栈成员访问支持索引**。需要注意的是：**堆栈操作是基于栈顶的，就是说它只会去操作栈顶的值**。

当创建好一个状态并在其中加载了标准库之后，就可以处理用户的输入了。程序会首先调用**函数luaL_loadstring来编译用户输入的每一行内容**。如果没有错误，则返回**零**，并**向栈中压入编译后得到的函数**。然后，程序调用**函数lua_pcall从栈中弹出编译后的函数**，并以保护模式运行。如果没有发生错误，pcall一样返回**零**，如果发生错误，这两个函数都会像栈中压入一条错误信息。然后我们可以通过**lua_tostring来获取错误信息**。

在其中有一个lua_pop函数，该函数表示从当前lua状态栈中弹出几个元素，如lua_pop( pLua, 2 )表示从栈顶弹出2个元素，当第二个参数填入-1时弹出所有元素即lua_pop( pLua, -1 ).

 

## lua堆栈操作

lua和c之间的通信主要组件是无处不在的虚拟栈，几乎所有的API调用都是在操作这个栈中的值，lua与c之间的所有数据交换都是通过这个栈完成的。此外，还可以利用栈保存中的结果。

在对lua栈操作的时候，当循环向栈中压入元素的时候，需要调用函数lua_checkstack来检查栈中是否有足够的空间。

C API提供了一系列lua_is\*的函数，其中\*可以是任意一种lua数据类型。这些函数包括lua_isnil,lua_isnumber,lua_isstring和lua_istable，lua_type返回栈中元素的类型，包含：LUA_TSTRING，LUA_TBOOLEAN，LUA_TNUMBER，LUA_TSTRING等。

针对于lua堆栈的操作。

C API使用索引（index）来引用栈中的元素。。第一个被压如栈的元素索引为1，第二个被压入的元素索引为2，-1表示栈顶元素，-2表示在它之前被压入栈的元素。

```c
static void stackDump(lua_State* L) {
	int i;
	int top = lua_gettop(L); //栈深度
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
	lua_rotate(L, 3, 1);//将指定元素向栈顶转动n个位置，并把栈顶元素补充过来
	stackDump(L);//true 10 true nil  hello nil
	lua_remove(L, -3);//移除指定位置的值
	stackDump(L);//true 10 nil hello nil
	lua_close(L);
}
```

## 处理应用代码中的错误

```c
int secure_foo(lua_State *L){
	lua_pushcfucntion(L, foo);
	return(lua_pcall(L,0,0,0) == 0)
}
```

## 内存分配

lua语言核心对内存不进行任何假设，它既不会调用malloc也不会调用realloc来分配内存。相反lua语言核心只会通过一个分配内存函数来分配和释放内存，当用户创建状态时必须提供函数。

**luaL_newstate是一个默认分配函数创建Lua状态的辅助函数。该默认分配函数使用了c语言标准库的标准函数malloc-realloc-freee**，对于大多数程序来岁，这几个函数够用了。但是要完全控制lua的内存分配也很容易，使用原始的lua_newstate来创建我们自己的lua状态即可。

```c
lua_State *lua_newstate(lua_Alloc f, void *ud);
```

该函数有两个参数：一个是分配函数，另一个是用户数据。用这种方式创建的lua状态会通过调用f完成所有的内存分配和释放，甚至结构lua_State也是由f分配的。

```c
typedef void *(*lua_Alloc)(void *ud, void *ptr, size_t osize,size_t nsize);
```

第一个参数始终为lua_newstate所提供的的用户数据；第二个参数正是被(重)分配或者释放的块的地址；第三个参数是原始块的大小；最后一个参数请求块大小。如果ptr不是NULL，lua会保证其之前分配的大小就是osize(如果是NULL，那么这个块之前的大小肯定是零，所以lua使用osize来存放某些调试信息）。

```c
void *l_alloc(void *ud,void *ptr,size_t osize,size_t nsize){
	(void) ud;(void)osize;
	if(nsize ==0){
		free(ptr);
		return NULL;
	}
	else
		return realloc(ptr, nsize);
}
```
