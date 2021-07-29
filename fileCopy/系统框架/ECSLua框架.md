# ECS Lua 框架

[toc]

一个完成的ECS框架主要包含entity，component和system这个三个部分，实体是一系列组件的聚合，而系统是拿到一系类具有某些组件的实体之后对其做一些事情，组件就是数据的集合，它只存放数据，我们对数据进行修改，然后在系统中进行获取，进而发生下一步动作。

简单来说：

**Entity:**可以说是传统引擎中的Game Object。它仅仅是C/Component的组合。它的意义在于生命期的管理。

**System:**专注于干好分内事情，每件事要么是作用于游戏世界里同类的一组对象的每单个个体的，要么是关心这类对象的某种特定的交互行为。

**Component:**把每个可能单独使用的对象属性归纳为一个个 Component 。每个 Entity 是由多个 Component 组合而成，共享一个生命期；而 Component 之间可以组合在一起作为 System 筛选的标准。

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

在游戏中一般都会存在只存在一帧的Component，这个使用我们需要根据这个组件，添加一个特殊的系统去执行，并且在执行完毕后，把这个组件从实体中移除。

**RemoveOneFrameSystem**

```lua
local LuaClass = LuaClass
local super = nil
---@class RemoveOneFrameSystem
local RemoveOneFrameSystem = class("RemoveOneFrameSystem", super)

---@param world EcsWorld
function RemoveOneFrameSystem:ctor(world, compType)
    self.world = world
    self.compType = compType
    self.filter = self.world:getFilter({compType})
end

function RemoveOneFrameSystem:execute()
    if self.filter:isEmpty() then return end
    ---@param e EcsEntity
    for _, e in pairs(self.filter.entities) do
        e:unSet(self.compType)
        self.world:destroyEntityIfEmpty(e)
    end
end

return RemoveOneFrameSystem
```



## Entity

实体,是组件的集合，管理这个实体的声明周期，从创建到销毁。

一个实体不能存在于虚空之中，它被创建出来的时候就应该放在一个世界中，并且又一个专属于它自己的标识，一般我们都是适用一个id来表示它，这个id注意是这个实体在这个世界中的唯一标识，所以要保证它应该是唯一的。

一个实体根据定义是一系列组件的组合，所以这里面肯定存在组件相关的存储。

```lua
function EcsEntity:ctor()
    self.id = 0
    self._is_enabled = false
    self.componentNum = 0
    self._components = {}
    ---@type EcsWorld
    self._owner = nil
end
---@param world EcsWorld
function EcsEntity:activate(world)
    self._owner = world
    self._is_enabled = true
end
```

接下来就是对entity组件的增删改查的操作了。其中添加和移除是比较重要和复杂的，在添加的时候包含创建一个新的component，已经存在这个组件怎么办，缓存池已经filter的更新，以及回收。

```lua
function EcsEntity:set(compType, ...)
    if not self._is_enabled then
        error("Cannot add component entity is not enabled.")
    end

    local previousComp = self._components[compType]

    local pool = self._owner:getPool(compType)
    local new_comp = pool:get()

    if new_comp.set then
        new_comp:set(...)
    end

    self._components[compType] = new_comp
    local propertyName = self:_getComponetProteryName(compType)
    self[propertyName] = new_comp
    if not previousComp then
    	self.componentNum = self.componentNum + 1
    end

    self._owner:updateFilter(self, previousComp, new_comp)
    if previousComp then
        pool:recycle(previousComp)
    end
    return new_comp
end
function EcsEntity:_getComponetProteryName(component)
    local name = string.gsub(string.gsub(component.__cname,"Component$",""),"^%u", string.lower)
    return name
end

function EcsEntity:unSet(compType)
    if not self:has(compType) then
        --error(string.format("Cannot remove unexisting component %s", tostring(comp_type.__comp_name)))
        return
    end

    local previousComp = self._components[compType]
    self._components[compType] = nil
    local propertyName = self:_getComponetProteryName(compType)
    self[propertyName] = nil
    self.componentNum = self.componentNum - 1

    self._owner:updateFilter(self, previousComp, nil)

    local pool = self._owner:getPool(compType)
    pool:recycle(previousComp)
end


---ECSWORLD 类中
---@param compType EcsComponent
---@return EcsComponentPool
function EcsWorld:getPool(compType)
    local pool = self.componentPools[compType]
    if not pool then
        pool = LuaClass.EcsComponentPool(compType)
        self.componentPools[compType] = pool
    end
    return pool
end

```

这里存在一个组件池的概念，即在world中存在一个字典，里面放了每种组件的池子。

```lua
local LuaClass = LuaClass
local super = nil
---@class EcsComponentPool
local EcsComponentPool = class("EcsComponentPool", super)

function EcsComponentPool:ctor(compType)
    self.items = {}
    self._type = compType
    ---@type int[]
    self._reservedItems = {}
    self._reservedItemsCount = 0
end

function EcsComponentPool:get()
    local obj = nil
    if self._reservedItemsCount > 0 then
        obj = self._reservedItems[self._reservedItemsCount]
        self._reservedItemsCount = self._reservedItemsCount - 1
    else
        obj = self._type()
        self.items[#self.items + 1] = obj
    end
    return obj
end

---@param idx int
function EcsComponentPool:recycle(obj)
    if obj.reset then
        obj:reset()
    end

    self._reservedItemsCount = self._reservedItemsCount + 1
    self._reservedItems[self._reservedItemsCount] = obj
end


return EcsComponentPool
```

我们每次拿新的组件和回收都要操作这个池子。

实体的销毁

```lua
function EcsEntity:destroy()
    self._is_enabled = false
    self:removeAll()
    self.id = 0
end
```

## ECS filter

这里插讲一下filter，这个意思是过滤器的意思，在世界中，我们的实体有那么的多，我们怎么快速的找到自己想要的实体呢？这个时候过滤器就出来了，意思是筛选拥有特定组件所有实体。

```lua
--------------------------------------------
-- Author: Jax
-- Date: 2020-05-06 15:35:57
-- Desc: 
---------------------------------------------

local LuaClass = LuaClass
local super = nil
---@class EcsFilter
local EcsFilter = class("EcsFilter", super)

function EcsFilter:ctor(all_of_tb, any_of_tb, none_of_tb)
    self._all = all_of_tb
    self._any = any_of_tb
    self._none = none_of_tb

    ---@type EcsEntity[]
    self.entities = {}

    self.onAddEvent = LuaClass.EventBeat("onAddEvent")
    self.onRemoveEvent = LuaClass.EventBeat("onRemoveEvent")
    self.onUpdateEvent = LuaClass.EventBeat("onUpdateEvent")
end

---@return EcsEntity[]
function EcsFilter:getEntities(result)
    result = result or {}
    table.insertto(result, self.entities)
    return result
end

---@param entity EcsEntity
function EcsFilter:matches(entity)
    if not entity._is_enabled or entity.componentNum <= 0 then
        return false
    end
    local all_cond = not self._all or #self._all == 0 or entity:hasAll(self._all)
    if not all_cond then
        return false
    end
    local any_cond = not self._any or #self._any == 0 or entity:hasAny(self._any)
    if not any_cond then
        return false
    end
    local none_cond = not (self._none and entity:hasAny(self._none))
    return none_cond
end

---@return EcsEntity
function EcsFilter:singleEntity()
    return self.entities[1]
end

function EcsFilter:isEmpty()
    return #self.entities == 0
end

function EcsFilter:handleEntitySilently(entity)
    if self:matches(entity) then
        self:_addEntitySilently(entity)
    else
        self:_removeEntitySilently(entity)
    end
end

function EcsFilter:handleEntity(entity)
    if self:matches(entity) then
        if self:_addEntity(entity) then
            self.isChanged = true
            self.onAddEvent:update(entity)
        end
    else
        if self:_removeEntity(entity) then
            self.isChanged = true
            self.onRemoveEvent:update(entity)
        end
    end
end

function EcsFilter:updateEntity(entity)
    self.isChanged = true
    self.onUpdateEvent:update(entity)
end

function EcsFilter:_addEntitySilently(entity)
    if not self.entities[entity] then
        self.entities[entity] = entity
        self.entities[#self.entities+1] = entity
        return true
    end
    return false
end

function EcsFilter:_removeEntitySilently(entity)
    if self.entities[entity] then
        self.entities[entity] = nil
        table.removebyvalue(self.entities, entity)
        return true
    end
    --print("Group:_remove_entity_silently failed")
    return false
end

function EcsFilter:_addEntity(entity)
    return self:_addEntitySilently(entity)
end

function EcsFilter:_removeEntity(entity)
    return self:_removeEntitySilently(entity)
end

function EcsFilter:has(entity)
    return self.entities[entity] ~= nil
end

return EcsFilter
```

关于filter筛选实体的创建，在世界中，每次筛选实体的时候，都会创建一个filter，这个filter挥别缓存下来，所以之前我们对实体进行组件的更新的时候都需要更新filter。

为了下次这个实体组件发生了变换，我们需要对filter进行更新，需要将filter缓存下来。

```lua
---@return EcsFilter
function EcsWorld:getFilter(all_of_tb, any_of_tb, none_of_tb)
    local key = string_components_ex(all_of_tb).."|"..string_components_ex(any_of_tb).."|"..string_components_ex(none_of_tb)

    local filter = self.filters[key]
    if filter then
        return filter
    end
    -- printJax("key=", key)
    all_of_tb = all_of_tb and table.clone(all_of_tb)
    any_of_tb = any_of_tb and table.clone(any_of_tb)
    none_of_tb = none_of_tb and table.clone(none_of_tb)
    filter = LuaClass.EcsFilter(all_of_tb, any_of_tb, none_of_tb)


    for _, value in pairs(self.entities) do
        filter:handleEntitySilently(value)
    end

    self.filters[key] = filter

    self:_addFilterCache(all_of_tb, filter)
    self:_addFilterCache(any_of_tb, filter)
    --self:_addFilterCache(none_of_tb, filter)
    if none_of_tb and #none_of_tb > 0 then
        self.filterByComponents["none"] = self.filterByComponents["none"] or {}
        table.insert(self.filterByComponents["none"], filter)
    end

    return filter
end
function EcsWorld:_addFilterCache(arr, filter)
    if arr then
        local filterList
        for _, compType in ipairs(arr) do
            filterList = self.filterByComponents[compType]
            if not filterList then
                filterList = {}
                self.filterByComponents[compType] = filterList
            end
            table_insert(filterList, filter)
        end
    end
end
function EcsWorld:updateFilter(entity, previousComp, newComp)
    local compType = previousComp and previousComp.__index or nil
    if not compType then
        compType = newComp and newComp.__index or nil
    end

    if not compType then
        return
    end

    ---@type EcsFilter[]
    local filterList
    if newComp and previousComp then
        compType = newComp.__index
        filterList = self.filterByComponents[compType]
        if filterList then
            for _, filter in ipairs(filterList) do
                filter:updateEntity(entity)
            end
        end
    else
        if newComp then
            compType = newComp.__index
            filterList = self.filterByComponents[compType]
            if filterList then
                for _, filter in ipairs(filterList) do
                    filter:handleEntity(entity)
                end
            end
        elseif previousComp then
            compType = previousComp.__index
            filterList = self.filterByComponents[compType]
            if filterList then
                for _, filter in ipairs(filterList) do
                    filter:handleEntity(entity)
                end
            end
         end

        filterList = self.filterByComponents["none"]
        if filterList then
            for _, filter in ipairs(filterList) do
                filter:handleEntity(entity)
            end
        end
    end
end
```

## ECS World

上面我们已经讲了好多关于世界的用法了，接下里整体讲一下，世界就是实体的载体，意味着它会对整个世界内的实体进行管理，管理的方式有哪些呢？

* 组件池
* 筛选器
* 所有的实体
* 实体池

```lua
function EcsWorld:ctor(seed)
    ---@type table<string, EcsEntity>
    self.entities = {}
    ---@type EcsEntity[]
    self._entities_pool = {}
    ---@type table<string, EcsFilter>
    self.filters = {}
    ---@type table<EcsComponent, EcsFilter[]>
    self.filterByComponents = {}
    ---@type table<EcsComponent, EcsComponentPool>
    self.componentPools = {}
    self._uuid = 0

    ---@type EventDispatcher
    self.eventDispather = LuaClass.EventDispatcher()
    ---@type PhysicsWorldMgr
    self.physicsWorldMgr = LuaClass.PhysicsWorldMgr(self)
    self.physicsWorldMgr.physicsMgr.CheckCollisionEnabled = handler(self, self.checkCollisionEnabled)

    self.random = LuaClass.TSRandom.New(seed or math.random(1, 10000))
end
```



新的实体的创建和销毁：

```lua
---@return EcsEntity
function EcsWorld:newEntity(entityId)
    local entity = table.remove(self._entities_pool)
    if not entity then
        entity = LuaClass.EcsEntity()
    end

    if entityId and self.entities[entityId] then
        error("重复id = " .. entityId)
    end

    entity:activate(self)
    entity.id = entityId or self:genUUID()

    self.entities[entity.id] = entity

    return entity
end
---@param entity EcsEntity
function EcsWorld:destroyEntity(entity)
    if not self:hasEntity(entity.id) then
        return
    end

    local id = entity.id
    entity:destroy()
    self.entities[id] = nil
    table.insert(self._entities_pool, entity)
end
```

## ECS System

这个系统是所有系统的管理器，用来驱动所有的系统的。

```lua

local LuaClass = LuaClass
local super = nil
---@class EcsSystem
local EcsSystem = class("EcsSystem", super)

local table_insert = table.insert

function EcsSystem:ctor()
    self._initSystems = {}
    self._executeSystems = {}
    self._destroySystems = {}
end

function EcsSystem:add(system)
    if system.init then
        table_insert(self._initSystems, system)
    end

    if system.execute then
        table_insert(self._executeSystems, system)
    end

    if system.destroy then
        table_insert(self._destroySystems, system)
    end
    return self
end

function EcsSystem:init()
    for _, system in ipairs(self._initSystems) do
        system:init()
    end
end

function EcsSystem:execute()
    for _, system in ipairs(self._executeSystems) do
        system:execute()
    end
end

function EcsSystem:destroy()
    for _, system in ipairs(self._destroySystems) do
        system:destroy()
    end
end

function EcsSystem:oneFrame(...)
    -- body
end

return EcsSystem
```



## SimulativeBehaviour

接下来讲述对每个行为的模拟，在我们的游戏中，是帧同步的，所以每个画面都是一帧一帧计算并同步过来的，这里我们可以在每个行为进行模拟。

```lua
local LuaClass = LuaClass
local super = nil
---@class ISimulativeBehaviour
---@field sim Simulation
local ISimulativeBehaviour = class("ISimulativeBehaviour", super)

function ISimulativeBehaviour:ctor()
    
end

function ISimulativeBehaviour:start()
    -- body
end

function ISimulativeBehaviour:update()
    -- body
end

function ISimulativeBehaviour:quit()
    -- body
end

return ISimulativeBehaviour
```

在游戏中，一般拥有这些行为：

* 备份
* 实体行为（key）
* 输入行为
* 逻辑帧行为
* 回滚行为

我们把这些行为添加到模拟器中进行模拟，这就是一个场景的展开。

## Simulation

模拟，意思是模拟一个世界的运行，所以我们需要在模拟器中加入一个世界，和这个世界拥有哪些行为。

```lua
local LuaClass = LuaClass
local super = nil
---@class Simulation
local Simulation = class("Simulation", super)

function Simulation:ctor(id)
    self.id = id
    ---@type ISimulativeBehaviour[]
    self.behaviours = {}
    ---@type EcsWorld
    self.entityWorld = LuaClass.EcsWorld()
end

function Simulation:start()
    for _, behaviour in ipairs(self.behaviours) do
        behaviour:start()
    end
end

function Simulation:update()
    self.entityWorld.isActive = false
    for _, behaviour in ipairs(self.behaviours) do
        behaviour:update()
    end
    self.entityWorld.isActive = true
end

function Simulation:containBehaviour(value)
    if self.behaviours[value.__cname] then
        return false
    end
    for _, behaviour in ipairs(self.behaviours) do
        if behaviour == value then
            return true
        end
        
    end
    return false
end

function Simulation:addBehaviour(behaviour)
    if not self:containBehaviour(behaviour) then
        self.behaviours[#self.behaviours+1] = behaviour
        self.behaviours[behaviour.__cname] = behaviour
        behaviour.sim = self
    end
end

function Simulation:removeBehaviour(behaviour)
    if self:containBehaviour(behaviour) then
        table.removebyvalue(self.behaviours, behaviour)
        self.behaviours[behaviour.__cname] = nil
        behaviour.sim = nil
    end
end

function Simulation:getBehaviour(T)
    return self.behaviours[T.__cname]
end

function Simulation:destroy()
    for _, behaviour in ipairs(self.behaviours) do
        behaviour:quit()
    end
    self.entityWorld:destroy()
end

return Simulation
```

## SimulationMgr

我们可以对多个世界进行模拟，所以我们可以通过一个manager来管理所有的世界，进行初始化和更新等一系列的操作。

```lua
-- @Author: Jax
-- @Date:   2020-04-28 09:56:55
-- @Last Modified by:   Jax
-- @Last Modified time: 2020-04-28 10:09:01
local LuaClass = LuaClass
local super = nil
---@class SimulationMgr
local SimulationMgr = class("SimulationMgr", super)

function SimulationMgr:ctor()
    ---@type Simulation[]
    self.simList = {}
    ---@type Simulation
    self.clientSim = nil
end

function SimulationMgr:addSimulation(sim)
    self.simList[#self.simList+1] = sim
end

---@param sim Simulation
function SimulationMgr:destroySim(sim)
    table.removebyvalue(self.simList, sim)
    sim:destroy()
end

---@return Simulation
function SimulationMgr:getSimulation()
    return self.clientSim
end

function SimulationMgr:start()
    for _, sim in ipairs(self.simList) do
        sim:start()
    end
end

function SimulationMgr:update()
    for _, sim in ipairs(self.simList) do
        sim:update()
    end
end

return SimulationMgr
```

## FrameData

游戏中，游戏可能需要回滚，前进等一系列的操作，这个时候我们需要游戏的数据快照，在我们的游戏中，主要是实体和组件的快照，拿到这些数据之后我们就可以重新构建我们的游戏世界，并让其恢复到这个快照的瞬间。

```lua
local LuaClass = LuaClass
local super = nil
---@class EntityWorldFrameData 帧数据快照
local EntityWorldFrameData = class("EntityWorldFrameData", super)

---@param components EcsComponent[]
function EntityWorldFrameData:ctor(entityIds, components)
    self.entityIds = entityIds
    self.components = components
end

function EntityWorldFrameData:clear()
    self.entityIds = nil
    self.components = nil
end

function EntityWorldFrameData:clone()
    local cloneEntities = {}
    for index, value in ipairs(self.entityIds) do
        cloneEntities[index] = value
    end

    local cloneComponents = {}
    for index, value in ipairs(self.components) do
        cloneComponents[index] = value:clone()
    end

    local data = EntityWorldFrameData(cloneEntities, cloneComponents)
    return data
end

return EntityWorldFrameData
```

## FameCommand

如何将渲染层级的事情派发到逻辑层中，在这里我们使用FrameCommandSystem来进行同步。

```lua
App.keyFrameSender:addCurrentFrameCommand(
    LuaClass.FrameCommand.SYNC_PLAYER_POSITION,
    castEntityId,
    msg
)
```

通过使用keyFrameSender类来发送命令：

```lua
local LuaClass = LuaClass
local super = nil
---@class KeyFrameSender
local KeyFrameSender = class("KeyFrameSender", super)

function KeyFrameSender:ctor()
    ---@type PtKeyFrameCollection
    self.keyFrameCollection = LuaClass.PtKeyFrameCollection()
end

function KeyFrameSender:addCurrentFrameCommand(cmd, entityId, params)
    ---@type LogicFrameBehaviour
    local logic = App.simulationMgr:getSimulation():getBehaviour(LuaClass.LogicFrameBehaviour)
    if logic then
        --local info = LuaClass.FrameIdxInfo(logic.currentFrameIdx, cmd, entityId, params)
        ---@type FrameIdxInfo
        local info = table.create()
        info.idx = logic.currentFrameIdx
        info.cmd = cmd
        info.entityId = entityId
        info.params = params
        self:addFrameCommand(info)
    end
end

---@param info FrameIdxInfo
function KeyFrameSender:addFrameCommand(info)
    self.keyFrameCollection.frameIdx = info.idx
    self.keyFrameCollection.keyFrames[#self.keyFrameCollection.keyFrames + 1] = info
end

function KeyFrameSender:getKeyFrameCollection()
    return self.keyFrameCollection
end

function KeyFrameSender:clearFrameCommand()
    self.keyFrameCollection.frameIdx = -1
    local len = #self.keyFrameCollection.keyFrames
    for i = 1, len do
        self.keyFrameCollection.keyFrames[i] = nil
    end
end

return KeyFrameSender
```

每一帧都有一个命令收集器：

```lua
local LuaClass = LuaClass
local super = nil
---@class PtKeyFrameCollection
local PtKeyFrameCollection = class("PtKeyFrameCollection", super)

function PtKeyFrameCollection:ctor()
    self.frameIdx = -1
    ---@type FrameIdxInfo[]
    self.keyFrames = {}
end

return PtKeyFrameCollection
```

我们需要在FrameCommandSystem中，注册命令事件：

```lua
local LuaClass = LuaClass
local super = nil
---@class FrameCommandSystem
local FrameCommandSystem = class("FrameCommandSystem", super)

local PlayerState = LuaClass.FightEnum.PlayerState

---@param world EcsWorld
function FrameCommandSystem:ctor(world)
    self.world = world
    self.filter = world:getFilter({ LuaClass.KeyFrameCollectionEvent })
    self.commandHandler = {}
    self:registerCommands()
    self.world.onKeyFrameCollectionEvent = handler(self, self.onKeyFrameCollectionEvent)
end

function FrameCommandSystem:onKeyFrameCollectionEvent(collection)
    local func, frame
    local len = #collection.keyFrames
    for i = 1, len do
        frame = collection.keyFrames[i]
        func = self.commandHandler[frame.cmd]
        if func then
            func(frame)
        end
        table.release(frame.params)
        table.release(frame)
        collection.keyFrames[i] = nil
    end
end
function FrameCommandSystem:registerCommands()
    self.commandHandler[LuaClass.FrameCommand.SYNC_CREATE_ENTITY] = handler(self, self.onCreateEntity)
end
function FrameCommandSystem:onCreateEntity(frame)
end
```

## 帧同步

通过RollbackBehaviour行为来回滚关键帧信息，我们的操作就在这里面：

```lua
local LuaClass = LuaClass
local super = LuaClass.EntitasBehaviour
---@class RollbackBehaviour:EntitasBehaviour 回滚关键帧信息
local RollbackBehaviour = class("RollbackBehaviour", super)

function RollbackBehaviour:update()
    ---@type LogicFrameBehaviour
    self.logicBehaviour = self.sim:getBehaviour(LuaClass.LogicFrameBehaviour)
    ---@type BackupBehaviour
    self.backupBehaviour = self.sim:getBehaviour(LuaClass.BackupBehaviour)

    while #App.fightNetMgr.queueKeyFrameCollection > 0 do
        local pt = App.fightNetMgr.queueKeyFrameCollection[1]
        if pt and pt.frameIdx <= self.logicBehaviour.currentFrameIdx then
            ---@type PtKeyFrameCollection
            local keyframeCollection = table.remove(App.fightNetMgr.queueKeyFrameCollection, 1)
            -- printJax("RollbackBehaviour:update", pt.frameIdx, self.logicBehaviour.currentFrameIdx)
            self:rollImpl(keyframeCollection)

            table.release(keyframeCollection.keyFrames)
            table.release(keyframeCollection)
        end
    end
end

---@param collection PtKeyFrameCollection
function RollbackBehaviour:rollImpl(collection)
    --local frameIdx = collection.frameIdx
    --if frameIdx < 0 then
    --    return
    --end
    -- 回放命令存储
    --for _, frame in ipairs(collection.keyFrames) do
    --    -- self.logicBehaviour:updateKeyFrameIdxInfoAtFrameIdx(frameIdx, frame)
    --end

    if self.sim.entityWorld.onKeyFrameCollectionEvent then
        self.sim.entityWorld.onKeyFrameCollectionEvent(collection)
    end

    --e:set(LuaClass.KeyFrameCollectionEvent, collection)
end

---回滚关键帧
---@param collection PtKeyFrameCollection
-- function RollbackBehaviour:rollImpl(collection)
--     local frameIdx = collection.frameIdx
--     if frameIdx < 0 then
--         return
--     end

--     LuaClass.FightSortUtil:sortKeyFrames(collection.keyFrames)

--     -- 回放命令存储
--     for _, frame in ipairs(collection.keyFrames) do
--         self.logicBehaviour:updateKeyFrameIdxInfoAtFrameIdx(frameIdx, frame)
--     end

--     local framePrevData = self.backupBehaviour:getEntityWorldFrameByFrameIdx(frameIdx-1)
--     if framePrevData then
--         self.sim.entityWorld:rollback(framePrevData:clone(), collection)
--         while frameIdx < self.logicBehaviour.currentFrameIdx do
--             super.update(self)
--             self.backupBehaviour:setEntityWorldFrameByFrameIdx(frameIdx, self.sim.entityWorld:snapshot())
--             frameIdx = frameIdx + 1
--         end
--     else
--         printJax("no framePrevData", frameIdx)
--     end
-- end

return RollbackBehaviour
```

## 逻辑帧行为

```lua
local LuaClass = LuaClass
local super = LuaClass.ISimulativeBehaviour
---@class LogicFrameBehaviour:ISimulativeBehaviour
local LogicFrameBehaviour = class("LogicFrameBehaviour", super)

function LogicFrameBehaviour:ctor()
    super.ctor(self)
    ---@type FrameIdxInfo[][]
    self._frameIdxInfos = {}
end

function LogicFrameBehaviour:getFrameIdxInfos()
    return self._frameIdxInfos
end

function LogicFrameBehaviour:start()
    self.currentFrameIdx = 0
end

function LogicFrameBehaviour:update()
    self.currentFrameIdx = self.currentFrameIdx + 1
    --self._frameIdxInfos[#self._frameIdxInfos+1] = {}
end

---@param info FrameIdxInfo
function LogicFrameBehaviour:updateKeyFrameIdxInfoAtFrameIdx(frameIdx, info)
    --info.idx = frameIdx
    --if frameIdx >= #self._frameIdxInfos then
    --    error("error:" .. frameIdx)
    --end
    --local frames = self._frameIdxInfos[frameIdx]
    --local updateState = false
    --for _, keyframe in ipairs(frames) do
    --    -- if keyframe:equalInfo(info) then
    --    --     updateState = true
    --    --     keyframe.params = info.params
    --    --     break
    --    -- end
    --end
    --
    --if not updateState then
    --    frames[#frames+1] = info
    --end

    -- LuaClass.FightSortUtil:sortKeyFrames(frames)
end

return LogicFrameBehaviour
```

存放每一帧的信息和当前世界进行了多少帧。

## 输入行为

```lua
local LuaClass = LuaClass
local super = LuaClass.ISimulativeBehaviour
---@class InputBehaviour
local InputBehaviour = class("InputBehaviour", super)

function InputBehaviour:update()
    if LuaClass.Input.GetKeyDown(LuaClass.KeyCode.A) then
        printJax("GetKeyDown A")
    end

    if LuaClass.Input.GetKeyUp(LuaClass.KeyCode.A) then
        printJax("GetKeyUp A")
    end
end

return InputBehaviour
```

## 备份行为

```lua
local LuaClass = LuaClass
local super = LuaClass.ISimulativeBehaviour
---@class BackupBehaviour:ISimulativeBehaviour
local BackupBehaviour = class("BackupBehaviour", super)

function BackupBehaviour:ctor()
    ---@type table<int,EntityWorldFrameData>
    self._entityWorldFrameDataDict = {}
    ---@type int[]
    self.queueFrameCache = {}
end

function BackupBehaviour:getEntityWorldFrameByFrameIdx(frameIdx)
    return self._entityWorldFrameDataDict[frameIdx]
end

---@param data EntityWorldFrameData
function BackupBehaviour:setEntityWorldFrameByFrameIdx(frameIdx, data)
    self.queueFrameCache[#self.queueFrameCache+1] = frameIdx
    if self._entityWorldFrameDataDict[frameIdx] then
        self._entityWorldFrameDataDict[frameIdx]:clear()
    end
    self._entityWorldFrameDataDict[frameIdx] = data
end

function BackupBehaviour:getEntityWorldFrameData()
    return self._entityWorldFrameDataDict
end

function BackupBehaviour:sendKeyFrame(idx)
    if #App.keyFrameSender.keyFrameCollection.keyFrames > 0 then
        -- printJax("BackupBehaviour:sendKeyFrame", idx)
        App.fightNetMgr:requestSyncClientKeyframes(idx, App.keyFrameSender:getKeyFrameCollection())
        App.keyFrameSender:clearFrameCommand()
    end
end

function BackupBehaviour:update()
    ---@type LogicFrameBehaviour
    local logic = self.sim:getBehaviour(LuaClass.LogicFrameBehaviour)
    local frameIdx = logic.currentFrameIdx
    
    --self:setEntityWorldFrameByFrameIdx(frameIdx, self.sim.entityWorld:snapshot())

    self:sendKeyFrame(frameIdx)
end

return BackupBehaviour
```

## 游戏入口类

```lua
-- @Author: Jax
-- @Date:   2020-04-24 12:20:37
-- @Last Modified by:   zhangzhiqiang
-- @Last Modified time: 2020-06-30 11:45:38
local LuaClass = LuaClass
local super = LuaClass.BaseLuaComponent
---@class GameStartup:BaseLuaComponent
local GameStartup = class("GameStartup", super)

local EcsEntityExtensions = LuaClass.EcsEntityExtensions
local EcsWorldExtensions = LuaClass.EcsWorldExtensions
local EcsWorldExtensionsPhysics = LuaClass.EcsWorldExtensionsPhysics
local EcsWorldExtensionsCoroutine = LuaClass.EcsWorldExtensionsCoroutine
local a = LuaClass.RoleAndBombInfo
local b = LuaClass.SpineInfo

function GameStartup:Awake(chapterId, stageIndex)

    CS.Main.Instance:GC()
    self.gameObject:Layer("Map")
    App.gamestartup = self

    App.fightNetMgr:init()

    local sim = LuaClass.Simulation()
    sim:addBehaviour(LuaClass.LogicFrameBehaviour())
    sim:addBehaviour(LuaClass.BackupBehaviour())
    sim:addBehaviour(LuaClass.RollbackBehaviour())
    sim:addBehaviour(LuaClass.EntitasBehaviour())
    sim:addBehaviour(LuaClass.InputBehaviour())

    ---@type EntitasBehaviour
    local entitasBehaviour = sim:getBehaviour(LuaClass.EntitasBehaviour)
    --entitasBehaviour:addSystem(LuaClass.SupportSystem(sim.entityWorld))
    entitasBehaviour:addSystem(LuaClass.FrameCommandSystem(sim.entityWorld))
                    :addSystem(LuaClass.FrameClockSystem(sim.entityWorld))
                    :addSystem(LuaClass.TimeSystem(sim.entityWorld))

                    :addSystem(LuaClass.ChapterInitSystem(sim.entityWorld))
                    :addSystem(LuaClass.LoadStageSystem(sim.entityWorld))
                    :addSystem(LuaClass.FightCreateSystem(sim.entityWorld))
                    :addSystem(LuaClass.FightInitSystem(sim.entityWorld))
                    :addSystem(LuaClass.CreatePlayerSystem(sim.entityWorld))
                    :addSystem(LuaClass.FightStartSystem(sim.entityWorld))
                    :addSystem(LuaClass.FightJudgeResultSystem(sim.entityWorld))
                    :addSystem(LuaClass.TurnStartSystem(sim.entityWorld))
                    :addSystem(LuaClass.TurnEndSystem(sim.entityWorld))

                    :addSystem(LuaClass.MoveSystem(sim.entityWorld))
                    :addSystem(LuaClass.NoCollideWithSystem(sim.entityWorld))
                    :addSystem(LuaClass.PhysicsBodySwitchSystem(sim.entityWorld))
                    :addSystem(LuaClass.PhysicsSystem(sim.entityWorld))
                    :addSystem(LuaClass.CollisionSystem(sim.entityWorld))
                    --:addSystem(LuaClass.IdleSystem(sim.entityWorld))

                    :addSystem(LuaClass.LaunchPlayerSystem(sim.entityWorld))
                    :addSystem(LuaClass.PlayerStateSystem(sim.entityWorld))
                    :addSystem(LuaClass.NpcStateSystem(sim.entityWorld))

                    :addSystem(LuaClass.AISystem(sim.entityWorld))
                    :addSystem(LuaClass.AimSystem(sim.entityWorld))

                    :addSystem(LuaClass.BombStateSystem(sim.entityWorld))

                    :addSystem(LuaClass.ForbidOperateSystem(sim.entityWorld))

                    :addSystem(LuaClass.SkillTriggerSystem(sim.entityWorld))
                    :addSystem(LuaClass.SkillSystem(sim.entityWorld))
                    :addSystem(LuaClass.BuffSystem(sim.entityWorld))
                    :addSystem(LuaClass.MotionSystem(sim.entityWorld))
                    :addSystem(LuaClass.LaunchBombSystem(sim.entityWorld))
                    :addSystem(LuaClass.TrackSystem(sim.entityWorld))
                    :addSystem(LuaClass.LaserSystem(sim.entityWorld))

                    :addSystem(LuaClass.BreakObjectSystem(sim.entityWorld))
                    :addSystem(LuaClass.RotationSystem(sim.entityWorld))


                    :addSystem(LuaClass.ItemSystem(sim.entityWorld))

                    :addSystem(LuaClass.DestroyedSystem(sim.entityWorld))
                    :addSystem(LuaClass.DrawOutlineSystem(sim.entityWorld))
                    :oneFrame(LuaClass.LoadStageEvent)
                    :oneFrame(LuaClass.FightInitEvent)
                    :oneFrame(LuaClass.KeyFrameCollectionEvent)
                    :oneFrame(LuaClass.TriggerEvent)
                    :oneFrame(LuaClass.ChapterInitEvent)
                    :oneFrame(LuaClass.FightCreateEvent)

    App.simulationMgr:addSimulation(sim)
    App.simulationMgr.clientSim = sim
    self.sim = sim
    self.entityWorld = sim.entityWorld

    self:addLuaComponent(LuaClass.EntitySpawnerController, sim.entityWorld)
    self:addLuaComponent(LuaClass.MapMoveController, sim.entityWorld)
    self:addLuaComponent(LuaClass.ObjectSortingController, sim.entityWorld)
    self:addLuaComponent(LuaClass.FightUiController, sim.entityWorld)
    self:addLuaComponent(LuaClass.FightCameraController, sim.entityWorld)

    chapterId = chapterId or LuaClass.GameConfigDatatable:getNumberById(2,1)
    stageIndex = stageIndex or 1
    self.entityWorld:newEntityWith(LuaClass.ChapterInitEvent, chapterId, 1)

    App.outlineMgr:addEventListener(self.entityWorld)

    App.simulationMgr:start()
end

function GameStartup:FixedUpdate()
    if self._isDisposed then
        return
    end
    App.simulationMgr:update()
end

function GameStartup:OnDestroy()
    App.gamestartup = nil
    self._isDisposed = true
    App.simulationMgr:destroySim(self.sim)
    super.OnDestroy(self)
    CS.Main.Instance:GC()
end

return GameStartup

```

在我们的游戏入口类中，除了大量的逻辑帧初始化和处理方式，还包含表现成的管理类处理：

* EntitySpawnerController
* MapMoveController
* ObjectSortingController
* FightUiController
* FightCameraController

其中Uicontroller主要负责控制ui层，spawnerController，主要用来连接逻辑层和表现层的。

逻辑层告知表现层是通过事件触发的，操作成和逻辑层的交互是通过FrameCommand来交互的。

## 总结

在我们的战斗中，一共可以说包含两个层级，一个是跑在逻辑层的，一帧一帧的进行更新，一个是跑在表现层的，通过逻辑层的事件来驱动。

