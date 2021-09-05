# Weak及内存回收Collectgarbage

[toc]

弱表(weak table)是一个很有意思的东西，像C++/Java等语言是没有的。弱表的定义是：Aweak table is a table whose elements are weak references，元素为弱引用的表就叫弱表。有弱引用那么也就有强引用，有引用那么也就有非引用。我们先要厘这些基本概念：变量、值、类型、对象。

1. 变量与值：Lua是一个动态类型语言，也就是说**在Lua中，变量没有类型，它可以是任何东西，而值有类型**，所以**Lua中没有变量类型定义这种东西**。另外，**Lua中所有的值都是第一类值(first-class values)。**
2. **Lua有8种基本类型**：nil、boolean、number、string、function、userdata、thread、table。其中Nil就是nil变量的类型，nil的主要用途就是一个所有类型之外的类型，用于区别其他7中基本类型。
3. **对象objects:Tables、functins、threads、userdata。对于这几种值类型，其变量皆为引用类型（变量本身不存储类型数据，而是指向它们）。\**赋值、参数传递、函数返回等都操作的是这些值的引用，并不产生任何copy行为\****。 

## weak table的定义--弱引用

**弱表的使用就是使用弱引用，很多程度上是对内存的控制。**

* **weak表是一个表，它拥有metatable，并且metatable定义了__mode字段**；
* **weak表中的引用是\**弱引用(weakreference)\**，弱引用不会导致对象的引用计数变化。**换言之，如果一个对象只有弱引用指向它，那么gc会自动回收该对象的内存。
* __mode字段可以取以下三个值：k、v、kv。k表示table.key是weak的，也就是table的keys能够被自动gc；v表示table.value是weak的，也就是table的values能被自动gc；kv就是二者的组合。任何情况下，只要key和value中的一个被gc，那么这个key-value pair就被从表中移除了.

**对于普通的强引用表，当你把对象放进表中的时候，就产生了一个引用，那么即使其他地方没有对表中元素的任何引用，gc也不会被回收这些对象。**那么你的选择只有两种：手动释放表元素或者让它们常驻内存。

```lua
strongTable = {}
strongTable[1] = function() print("i am the first element") end
strongTable[2] = function() print("i am the second element") end
strongTable[3] ={10, 20, 30}
print(table.getn(strongTable)) -- 3
collectgarbage()
print(table.getn(strongTable)) -- 3
```

**在编程环境中，有时你并不确定手动给一个键值赋nil的时机，而是需要等所有使用者用完以后进行释放，在释放以前，是可以访问这个键值对的。这种时候，weak表就派上用场了。**

```lua
weakTable = {}  
weakTable[1] = function() print("i am the first element") end  
weakTable[2] = function() print("i am the second element") end  
weakTable[3] = {10, 20, 30}  
  
setmetatable(weakTable, {__mode = "v"}) -- 设置为弱表  
  
print(table.getn(weakTable))      -->3  
  
ele = weakTable[1]                -- 给第一个元素增加一个引用  
collectgarbage()  
print(table.getn(weakTable))      -->1，第一个函数引用为1，不能gc  
  
ele = nil                         -- 释放引用  
collectgarbage()  
print(table.getn(weakTable))      -->0，没有其他引用了，全部gc  
```

当然在实际的代码过程中，我们不一定需要手动collectgarbage，因为该函数是在后台自动运行的，它有自己的运行周期和规律，对编程者来说是透明的。另一例子：

```lua
a = {}  
b = {}  
setmetatable(a,b)  
b.__mode = "k"  --now 'a' has weak keys  
  
key = {}   --create first key  
a[key] = 1  
  
key = {}   --create second key   
a[key] = 2  
  
for k,v in pairs(a) do  
    print(v) --1   2  
end  
collectgarbage()  --forces a garbage collection cycle  
for k,v in pairs(a) do  
    print(v) --2    
    --[[第二个赋值语句key={}覆盖了第一个key的值。当垃圾收集器工作时，  
    在其他地方没有指向第一个key的引用，所以它被收集了，因此相对应的table中的入口也同时被移除了。  
    可是，第二个key，仍然是占用活动的变量key，所以它不会被收集。--]]      
end 
```

要注意，***\*只有对象才可以从一个weak table中被收集。比如数字和布尔值类型的值，都是不会被收集的\****。

关于字符串的一些细微差别：从上面的实现来看，尽管字符串是可以被收集的，他们仍然跟其他可收集对象有所区别。其他对象，比如tables和函数，他们都是显示的被创建。比如，不管什么时候当Lua遇到{}时，它建立了一个新的table。任何时候这个 function（）... end建立了一个新的函数（实际上是一个闭包）。然而，Lua见到“a”..“b”的时候会创建一个新的字符串？如果系统中已经有一个字符串“ab”的话怎么办？Lua会重新建立一个新的？编译器可以在程序运行之前创建字符串么？这无关紧要：这些是实现的细节。因此，从程序员的角度来看，字符串是值而不是对象。所以，就像数值或布尔值，一个字符串不会从weak tables中被移除（除非它所关联的vaule被收集）。

### 弱应用实例

```lua
t = {};      
-- 使用一个table作为t的key值  
key1 = {name = "key1"};  
t[key1] = 1;  
key1 = nil;  
      
-- 又使用一个table作为t的key值  
key2 = {name = "key2"};  
t[key2] = 1;  
key2 = nil;  
     
-- 强制进行一次垃圾收集  
collectgarbage();  
      
for key, value in pairs(t) do  
    print(key.name .. ":" .. value);  
end  
--输出  
--key1:1  
--key2:1
```

虽然我们在给t赋值之后，key1和key2都赋值为nil了。但是，已经添加到table中的key值是不会因此而被当做垃圾的。
换句话说，key1本身已经是nil值，但它曾经所指向的内容依然存放在t中。key2也是一样的情况。所以我们最后还是能输出key1和key2的name字段。
如果我们把某个table作为另一个table的key值后，希望当table设为nil值时，另一个table的那一条字段也被删除。应该如何实现？
这时候就要用到弱引用table了，弱引用table的实现也是利用了元表。我们来看看下面的代码，和之前几乎一样，只是加了一句代码：

```lua
t = {};      
-- 给t设置一个元表，增加__mode元方法，赋值为“k”  
setmetatable(t, {__mode = "k"});  
      
-- 使用一个table作为t的key值  
key1 = {name = "key1"};  
t[key1] = 1;  
key1 = nil;  
      
-- 又使用一个table作为t的key值  
key2 = {name = "key2"};  
t[key2] = 1;  
key2 = nil;  
      
-- 强制进行一次垃圾收集  
collectgarbage();  
      
for key, value in pairs(t) do  
    print(key.name .. ":" .. value);  
end  
--输出 为空  
```

留意，在t被创建后，立刻给它设置了元表，元表里有一个__mode字段，赋值为”k”字符串。如果这个时候大家运行代码，会发现什么都没有输出，因为，t的所有字段都不存在了。 
这就是弱引用table的其中一种，给table添加__mode元方法，如果这个元方法的值包含了字符串”k”，就代表这个table的key都是弱引用的。
一旦其他地方对于key值的引用取消了（设置为nil），那么，这个table里的这个字段也会被删除。
通俗地说，因为t的key被设置为弱引用，所以，执行t[key1] = 1后，t中确实存在这个字段。随后，又执行了key1 = nil，此时，除了t本身以外，就没有任何地方对key1保持引用，所以t的key1字段也会被删除。

## 内存回收

接着说下Lua的内存回收。Lua内存是自动收集的, 这点跟Java类似, 不被任何对象或全局变量引用的数据，将被首先标记为回收,不需要开发者做任何事情.但是，正如Java也会有内存泄露一样, Lua也会有, 只不过,跟C++的不同，它是由于代码执行所装载的资源，并没有被彻底销毁而导致,其中，最臭名昭著的就是不小心把局部变量声明成了全局变量(忘了加local修饰符)。 类似这样造成的内存泄露, 跟任何其他语言的内存泄露一样，容易产生，却难以察觉, 给开发的应用带来潜在的很大隐患.

那么, 有没有一些有效的解决办法, 来解决这个这个隐患呢, 答案就是collectgarbage. collectgarbage就是开放给Lua开发人员, 用于监听Lua的内存使用情况(collectgarbage("count")), 同时,它还提供了collectgarbage("collect"),允许在适当的时候进行显式的回收.

现在，通过测试代码来看看，如何玩转collectgarbage.

首先,为了有明显的对比, 先来看没有产生泄露的情况, 运行以下的test1(代码如下):

```lua
function:test1
    collectgarbage("collect")
    local c1 = collectgarbage("count")
    print("开始的内存大小：", c1)
    local colen = {} --在这里，colen是本地变量
    print("现在，声明5000个数组，并加入到colen中....")
    for i=1,5000 do
    	table.insert(colen, {})    
    end
    local c2 = collectgarbage("count")
    print("现在内存为：", c2)
end
```

这里看到, 被local 声明的colen加了5000数组, test1调用后, 内存增加了大概300K(25906K-25620K).

现在，我们来做内存回收(调用mem函数, 代码如下):

```lua
function:mem()
	print("调用GC")
    local c2 = collectgarbage("count")
    collectgarbage("collect")
    print("收集后， 当前Lua内存为：", c2)
end
```

( 为了保证内存的稳定,以上注意mem被调用了多次, 再第2次, 可以看到内存开始下降, 最后,大概在25618K稳定下来)
 好了,  从最初的 25620K,  到回收后的 25618K,  两者并没有发生变化(还少了2K，嘿嘿, 这应该是误差了), 也就是说,函数test1的执行，并没有产生无法回收的内存，没有泄露出现.
好了，现在运行有泄露的test2(代码如下), test2跟test1相比,只有一处不同:就是colen被误声明为全局:

```lua
function:test2
	collectgarbage("collect")
    local c1 = collectgarbage("count")
    print("开始的内存大小：", c1)
    colen = {} --在这里，colen是本地变量
    print("现在，声明5000个数组，并加入到colen中....")
    for i=1,5000 do
    	table.insert(colen, {})    
    end
    local c2 = collectgarbage("count")
    print("现在内存为：", c2)
end
```

为了保证函数回收被执行，这次，总共调用了7次mem函数(看以上打印行数), 那么，从上面的结果我们看, 很不幸, 从第1次，到最后第7次, 内存都还是稳定在25905K左右, 也就是说, 跟调用test2前相比，即使Lua进行了内存回收, 内存却不会将下来  看来 ,  这 300K(25906K-25620K) 内存 ,  由于已放到了全局函数中，是永远没有机会被回收到了!

## 总结：

**如何监测Lua的编程产生内存泄露:**

1. 针对会产生泄露的函数,先调用collectgarbage("count"),取得最初的内存使用
2. 函数调用后, collectgarbage("collect")进行收集, 并使用collectgarbage("count")再取得当前内存, 最后记录两次的使用差
3. 从test1的收集可看到, collectgarbage("collect")被调用，并不保证一次成功, 所以, 大可以调用多次

**如何避免Lua应用中出现的内存使用过大行为:**

1. 当然是代码实现不出现泄露, (废话*&%$()
2. 在测试中，其实还发现, Lua中被分配的内存，其实并不会自动回收(个人估计要么就是Lua虚拟机没有做这个事情，要么就是回收的时机是在C层), 所以, 为了避免内存过大, 应用的运行时，可能需要定期的（调用collectgarbage("collect")，又或者collectgarbage("step")）进行显式回收。

最后，结合weak table与内存回收，我们说下内存泄漏查证。需要说明的是，lua本身并不存在真正的内存泄漏，只是因为使用上面的原因导致无法gc，从而导致逻上的泄漏:)。

 参考GCObject的声明可以发现，lua中的复杂数据类型变量的传递都是基引用的。当lua从根开始gc扫描的时候，只要还有一个地方有对此变量的引用，那
么这个变量就不会被collect。这种情况造成的危害取决于多大程度上依赖于引
用，如果有适当的间接层/弱引用来隔离这个问题，可能问题会有所缓解。

 以下是一些常见的错误引用情景:
 1. 本应该local 的变量进入global空间或者module空间了(忘记写local)，如果这是一个table/function/udata等类型的变量的话，非常不幸的，这个变量将不会
  被正确gc了 ----除非你再显式的释放。这是非常容易犯的错误，一直在想为什么lua变量不是默认local呢？ 当然这个话题会引发另外一场争论。

  ```lua
  local function test_user(id)
   userobj = get_user_by_id(id) --这里总是会有一个玩家对象泄漏
   print("only test", userobj:get_name())
  end
  ```

 2. c/c++部分调用的lua_ref是否有正常lua_unref释放？ 通过debug.getregistry()可以查到这些ref.

 3. 其他各种各样的实际bug造成的泄漏。

 当怀疑系统有泄漏以后，我们可以怎么查到这些泄漏呢？我强烈建议大家建立一个weak table, 把你所有创建过的能够称之为资源的，包含但不限于“战斗对象，
玩家，npc，物品，场景，邮件”等等对象全部扔到这个table里面。当你知道玩家已经下线、战斗已经销毁了，但通过连续的强制full gc以后weak table里面还有
这个变量，这就证明了这个变量的引用没有被完全释放，于是问题就被发现了，我们又有事情干了@_@。

 知道有泄漏是比较容易的，能够完全揪出来就不是很容易了。是的，它究竟在哪儿呢? 一开始在此项目里面也是先发现比如某npc泄漏了，然后就去查代码，看看
究竟哪个地方写得不对。这种方式效率极低，基本上查不到什么问题。在迟一点的时候才使用现在的方案：从_G深度遍历所有的table、metatable、funciton's upvalue、function's env、registentry(lua_ref)。 目前所知的所有引用必定存在于这几个空间， 遍历完成以后一定可以找到那个“迷失了的引用”。 这种方式在
脚本层就可以完成所有事情，甚至你可以在运营环境中在线查证，其遍历的速度是非常快的，但内存开销非常大(:，可以考虑一边遍历一边gc，当然还要记得
避免重复搜索。 在应用此方案以后，此项目解决了脚本中所有的泄漏问题。

 **一点总结：**

1. 如果系统性能还能够承受的话，建议不要直接引用对象，可以多做一层间接层。
2. lua里面的弱引用是非常有用的。
3. 比较大的物理内存是必要的，这可以为大家查证问题争取足够多的时间:) 
4. 可以把查找泄漏的部分写入到关机逻辑里面，每次关机的时候自动查找泄漏，然后出具报告。

