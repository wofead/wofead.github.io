# Lua、C和C#的相互调用

[toc]

这三者之间的调用一共分为以下几种情况：

* C调用Lua
* C调用C#
* C#调用C
* C#调用Lua
* Lua调用C
* Lua调用C#

## Lua调用C

本质上是定义一个lua_CFunction,然后通过把关联到Lua中的一个table，默认的比如lua_register,实际上是把这个函数关联到global表的对应key上，其它的也可以关联到自己定义的table上，比如：

* lua_rawget获取注册表中的table
* lua_pushstring导出的函数名
* lua_pushcfunction，导出的函数
* lua_rawset设置关联

```c
lua_pushstring(l,"export_table");
lua_rawget(l,LUA_REGISTRYINDEX);
if(lua_istable(l,-1)){
    lua_pushstring(l,"function_name");
    lua_pushcfunction(l,lua_CFunction_define);
    //相当于export_table[function_name] = lua_CFunction_define;
    lua_rawset(l,-3);
}
lua_pop(l,1);
```

## C调用Lua

通过lua_call和lua_pcall实现，先把函数压栈，这里的函数是在lua中的function，由于上面C函数可以关联到lua的某个table中，所以，理论上也可以是C函数，然后把返回结果再压栈。具体参数含义见API说明。

> 作者：被遗失De跳刀
> 链接：https://zhuanlan.zhihu.com/p/25985695
> 来源：知乎
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
>
> 
>
> The following example shows how the host program can do the equivalent to this Lua code:
>
> ```lua
> a = f("how", t.x, 14)
> ```
>
> Here it is in C:
>
> ```c
>  lua_getfield(L, LUA_GLOBALSINDEX, "f"); /* function to be called */
>  lua_pushstring(L, "how");                        /* 1st argument */
>  lua_getfield(L, LUA_GLOBALSINDEX, "t");   /* table to be indexed */
>  lua_getfield(L, -1, "x");        /* push result of t.x (2nd arg) */
>  lua_remove(L, -2);                  /* remove 't' from the stack */
>  lua_pushinteger(L, 14);                          /* 3rd argument */
>  lua_call(L, 3, 1);     /* call 'f' with 3 arguments and 1 result */
>  lua_setfield(L, LUA_GLOBALSINDEX, "a");        /* set global 'a' */
> ```
>
> Note that the code above is “balanced”: at its end, the stack is back to its original configuration. This is considered good programming practice.

## C#调用C

C#调用C的代码是通过P/invoke, 即平台调用，.net 提供了一种托管代码调用非托管代码的机制。
通过DllImport特性实现，把c的相关函数声明成 static， extern的形式，还可以为方法的参数和返回值指定自定义封送处理信息。

具体可以参考[MSDN的描述](http://link.zhihu.com/?target=https%3A//docs.microsoft.com/en-us/dotnet/standard/native-interop/pinvoke)

## C调用C#

同时.Net也提供了从非托管代码访问托管代码的方式，是通过函数指针实现的。C代码调用C＃是通过delegate实现的，即把需要被调用的C#函数都声明成delegate，然后通过把函数地址通过DllImport已经导出的函数传入非托管代码（C代码），其中**Marshal.GetFunctionPointerForDelegate**可以获取函数指针。

有了上面的过程，下面的就好说了

## C#调用Lua

C# —》C —》Lua

- 把Lua相关的API，DLlIMPORT到C#
- 把需要调用的Lua函数用上面导出的函数压栈
- 调用导出的C#函数，pcall



## Lua调用C#

Lua —》C —》C#, 以下部分代码摘自Slua

- 把需要被调用的C#函数声明为delegate，由于Lua与C的通讯是通过lua_CFunction，所以这里我们声明的形式也要是一样的

```text
public delegate int LuaCSFunction(IntPtr luaState);
```

- 把Lua调用C相关的Lua API通过DllImport到C#，比如lua_pushcfunction

```text
[DllImport(LUADLL, CallingConvention = CallingConvention.Cdecl)]
public static extern void lua_pushcclosure(IntPtr l, IntPtr f, int nup);

public static void lua_pushcclosure(IntPtr l, LuaCSFunction f, int nup)
{
    IntPtr fn = Marshal.GetFunctionPointerForDelegate(f);
    lua_pushcclosure(l, fn, nup);
}
```

- 通过C#调用Lua的相关API，注册C#函数到Lua，与上面Lua调用C的注册过程是一样的

```text
LuaDLL.lua_pushcfunction(L, print);
LuaDLL.lua_setglobal(L, "print");
```

print定义如下

```text
static int print(IntPtr L)
```