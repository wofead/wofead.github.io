# FairyGUI And Unity 物品掉落实现

[toc]

## 掉落的item流程

1. 算出掉落的item数量
2. 创建item逻辑层实体和表现层实体
3. 销毁逻辑层实体和表现层实体

关于如何掉落的数量，按照策划的需求来算，然后派发事件告诉逻辑层创建实体，要先算出位置和速度。

```lua
local velocity = TSVector2(FP(speed) * LuaClass.TSMath.Cos(radian), FP(speed) * LuaClass.TSMath.Sin(radian))
local itemEntity = world:createItem(TSVector2(self.transform.position.x, self.transform.position.y), res, velocity)
world:notifyEvent(LuaClass.EntityEvent.CreateItem, itemEntity.id)
```

创建逻辑层实体:

1. 添加item组件
2. 添加资源组件
3. 添加类型组件
4. 创建body
5. 设置body属性（pos，rotation，collisionCategories，collidewith，设置速度）
6. 添加transform和velocitycomponent

```lua
local entity = self:newEntity()
entity:set(LuaClass.ItemComponent)

entity:set(LuaClass.ResPathComponent, res)
entity:set(LuaClass.CategoryComponent, LuaClass.FightEnum.ObjectCategory.Item)
local data = {}
data.Resource = res
local body = self:createBody(data)
local pos = LuaClass.TSVector2(position.x, position.y + FP(2))
body.Position = pos
body.Rotation = LuaClass.FP.Zero
body.CollisionCategories = ObjectCategory:value(ObjectCategory.Item)
body.CollidesWith = ObjectCategory:allOf({ ObjectCategory.Ground, ObjectCategory.AirWall, ObjectCategory.Bound })
self:_addPhysicsBodyComponent(entity, body)
self:_addColliderComponent(entity, body)
body:SetLinearVelocity(velocity)
entity:set(LuaClass.TransformComponent, position, FP(0))
entity:set(LuaClass.VelocityComponent, velocity)
```



然后通过itemsystem来销毁item。

```lua
    if LuaClass.TSMath.Abs(entity.velocity.value.x) < vel.x and LuaClass.TSMath.Abs(entity.velocity.value.y) < vel.y and not entity.item.velIsZero then --判断item是否静止了
    entity.item.timeCondition.time = 0
    entity.item.timeCondition.duration = 500
    entity.item.velIsZero = true
end
if (entity.item.timeCondition:check() or entity.item.isOut) and not entity.item.isFlaying then 
    local body = entity.physicsBody.value --告知itemrenderitem开始飞了
    body.IgnoreGravity = true
    body.CollidesWith = ObjectCategory:value(ObjectCategory.None)
    entity.item.isFlaying = true
end
```



## Item在地图层飞

```lua
 body.IgnoreGravity = true --忽略重力
 body.CollidesWith = ObjectCategory:value(ObjectCategory.None) --不和任何东西发生碰撞
```

在itemrender中判断如何飞

```lua
local fightCamera = App:getFightCamera()
local uiPos = self.chapterInfo.tokenUiPos[entity.resPath.value]
LuaClass.BombManMathUtil.GlobalUIPosToWorld(uiPos, fightCamera, vec)
local body = entity.physicsBody.value
local dis = LuaClass.TSVector2(FP(vec.x + 0.8), FP(vec.y - 0.7)) - entity.transform.position
if LuaClass.TSMath.Abs(dis.x) < FP(0.5) and LuaClass.TSMath.Abs(dis.y) < FP(0.5) then
    entity:set(LuaClass.DestroyedComponent)
    self.world.eventDispather:dispatchEvent(LuaClass.EntityEvent.GetToken, entity.transform.position, entity.resPath.value)
end
local radian = LuaClass.TSMath.Atan2(dis.y, dis.x)
local velocity = TSVector2(FP(30) * LuaClass.TSMath.Cos(radian), FP(30) * LuaClass.TSMath.Sin(radian))
body:SetLinearVelocity(velocity)
if not self.setFlyTrigger then
    self.animator:SetTrigger("fly")
else
    self.setFlyTrigger = true
end
```

将ui坐标根据fight camera算出其在map层的坐标，然后得到矢量的dis，在算速度，直到dis在误差为0.5的范围内。



## item在UI层飞

item在ui层，飞的可以是gcomponent，里面是icon或者里面放一个holder，然后使用wrapper，设置unityobject。

```lua
local prefab = LuaClass.AssetManager.Load(res)
local go = LuaClass.GameObject.Instantiate(prefab)
go.transform:LocalIdentity()
go.transform.localScale = LuaClass.Vector3(100, 100, 100)
go.transform.localPosition = LuaClass.Vector3(80, -70, 0)
self.ui._holderGraph:SetNativeObject(LuaClass.GuiGoWrapper(go))
```

然后使用缓动动画来：

```lua
local position = LuaClass.Vector2()
local vector = LuaClass.Vector3(pos.x:AsFloat(), pos.y:AsFloat(), 0)
LuaClass.BombManMathUtil.WorldPosToUILocal(vector, self.ui, App:getFightCamera(), position)
local tokenItem = App.uiPoolMgr:popComponent(LuaClass.UiConstant.FIGHT_VIEW.moduleConfig, "NewFight3/Component/RewardInfo/TokenItem")
self.ui:AddChild(tokenItem)
LuaClass.TokenItemView.AddToView(tokenItem, "GameAssets/Prefabs/UI/Item.prefab")
tokenItem:SetXY(position.x, position.y)
tokenItem.visible = true
local rewardPos = self.rewardNode.position
local tokenPos = tokenItem.position
local time = math.sqrt((rewardPos.x - tokenPos.x) ^ 2 + (rewardPos.y - tokenPos.y) ^ 2)
local tweener = LuaClass.GuiGTween.To(tokenPos, rewardPos, 2):SetTarget(tokenItem):OnUpdate(
function(tweener)
tokenItem:SetXY(tweener.value.x, tweener.value.y)
end)            :OnComplete(
function(tweener)
App.uiPoolMgr:pushComponent(tokenItem)
table.removebyvalue(self.tokenTweener, tweener)
self:addToken(tweener.userData)
end
)                       :SetUserData(res)
table.insert(self.tokenTweener, tweener)
```

