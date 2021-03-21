# ECS Lua 框架

[toc]

一个完成的ECS框架主要包含entity，component和system这个三个部分，实体是一系列组件的聚合，而系统是拿到一系类具有某些组件的实体之后对其做一些事情，组件就是数据的集合，它只存放数据，我们对数据进行修改，然后在系统中进行获取，进而发生下一步动作。

简单来说：

**Entity:**可以说是传统引擎中的Game Object。它仅仅是C/Component的组合。它的意义在于生命期的管理。

**System:**专注于干好分内事情，每件事要么是作用于游戏世界里同类的一组对象的每单个个体的，要么是关心这类对象的某种特定的交互行为。

**Component:**把每个可能单独使用的对象属性归纳为一个个 Componen。每个 Entity 是由多个 Component 组合而成，共享一个生命期；而 Component 之间可以组合在一起作为 System 筛选的标准。

游戏的业务循环就是在调用很多不同的系统，每个系统自己遍历自己感兴趣的对象，只有预定义的组件部分可以被子系统感知到，这样每个系统就能具备很强的内聚性。

Component 是纯数据组合，没有任何操作这个数据的方法；而 System 是纯方法组合，它自己没有内部状态。它要么做成无副作用的纯函数，根据它所能见到的对象 Component 组合计算出某种结果；要么用来更新特定 Component 的状态。

 Utility 函数的概念，查询连个Entity的敌对关系，Utility 函数共享给不同的 System 调用。

## Component

组件，纯数据组件，里面不包含任何方法，用来记录抹泪属性的状态，几乎有很少的组件被创建出来后会更新，但是有些还是会更新。

```lua
local LuaClass = LuaClass
local super = nil
---@class EcsComponent
local EcsComponent = class("EcsComponent", super)

function EcsComponent:ctor()
    
end

return EcsComponent
```

作为所有组件的基类，这个基类里面几乎什么都没有。在C#中，ECS component就是一个Struct，一个Struct代表一个component。

## Entity

实体,是组件的集合，管理这个实体的声明周期，从创建到销毁。

一个实体不能存在于虚空之中，它被创建出来的时候就应该放在一个世界中，并且又一个专属于它自己的标识，一般我们都是适用一个id来表示它，这个id注意是这个实体在这个世界中的唯一标识，所以要保证它应该是唯一的。



