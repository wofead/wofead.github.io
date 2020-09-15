# LuaState的学习与理解

[toc]

## 什么是LuaState

lua_state中放的是lua虚拟机(lua虚拟机？？)中的环境表、注册表、运行堆栈以及虚拟机的山下文数据。

从一个主线程(特指lua虚拟机中的线程，即coroutine)中创建出来的新的lua_state会共享大部分的数据，但会拥有一个独立的运行堆栈。所以一个线程对象拥有一个lua_state。

lua_state共享的数据部分是全局状态（包含gc数据）。lua_state的运行堆栈为调用栈。lua_state的数据栈包含当前调用栈信息。

虚拟机对象包含执行状态机和全局状态机，其中执行状态机包含数据栈和调用栈。

## lua_state线程对象

### lua_state的上下文数据

从c层看待lua，lua的虚拟机对象就是一个lua_state。但实际上，真正的lua虚拟机对象被隐藏起来了。那就是lstate.h中定义的结构体 global_State。同一lua虚拟机中的所有执行线程，共享了一块全局数据global_stae。lua_stae是暴露给用户的数据类型，即表示一个lua程序的执行状态，也指代lua的一个线程。每个线程拥有的独立的数据栈以及函数调用栈，还有独立的调试钩子和错误处理设置。所以我们不应当简单的把lua_state看成一个静态的数据集，它是一个lua线程的执行状态。所有的lua c API都是围绕这个状态机，或把数据压入堆栈，或取出，或执行栈顶的函数，或继续上次被中断的执行过程。

## 全局状态机

共享的一块全局数据 global_state。

从lua的使用者角度看，global_state是不可见的。我们无法用公开的api取到它的指针，也不需要引用它。但分析lua的实现就不能绕开这个部分。

global_state里面有对主线程的引用，有注册表管理所有全局数据，有全局字符串表，有内存管理函数，有gc需要的把所有对象串联起来的相关信息，以及一切lua在工作时需要的工作内存。

通过lua_newstate创建一个新的lua虚拟机时，第一块申请的内存将用来保存主线程和这个全局状态机。lua的实现尽可能的避免内存碎片，同时也减少内存分配和释放的次数。它采用了一个小技巧，使用一个LG结构，把主线程lua_state和global_state分配在一起。

```c

typedef struct LG {
  lua_State l;
  global_State g;
} LG;
```

## 执行状态机的数据栈和执行栈

数据栈：lua中的数据可以分为两类：值和引用。值类型可以任意复制，而引用类型共享一份数据，由GC负责维护生命周期。lua使用一个联合union Value来保存数据。

```c
typedef union {
  GCObject *gc;
  void *p;
  lua_Number n;
  int b;
} Value;
```

调用栈：lua 把调用栈和数据栈分开保存。调用栈放在一个叫做 CallInfo 的结构中，以数组的形式储存在虚拟机对象（线程对象）里

```c

typedef struct CallInfo {
  StkId base;  /* base for this function */
  StkId func;  /* function index in the stack */
  StkId top;  /* top for this function */
  const Instruction *savedpc;
  int nresults;  /* expected number of results from this function */
  int tailcalls;  /* number of tail calls lost under this entry */
} CallInfo;
```

正在调用的函数一定存在于数据栈上，在 CallInfo 结构中，func 指向正在执行的函数在数据栈上的位置。需要记录这个信息，是因为如果当前是一个 lua 函数，且传入的参数个数不定的时候，需要用这个位置和当前数据栈底的位置相减，获得不定参数的准确数量。