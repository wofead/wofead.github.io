---
layout:     post
title:      DDT常用业务的书写
subtitle:   Action
date:       2019-5-22
author:     Jow
header-img: img/about-bg-walle.jpg
catalog: 	 true 
tags:
    - DDT
    - module

---

### 目录
1. item的抽取和设计
2. Entity的设计
3. 主界面的设计
4. 副界面的设计
5. tween
6. list
7. 购买的判断PurchaseHelper
8. 新手
9. 通用物品类
10. 通用排行榜数据获取
11. timer
12. 字体颜色
13. baseview
14. 时间处理
15. 坐标转换

> 成功的人千篇一律，直率、随风一往无前而已。

## item的抽取和设计

1. 在设计item的时候注意锚点的设置，这样才能精确的将item嵌入到正确的位置上。
2. 在将item抽取出来的时候注意要将其从父容器中移除，这样才不会报重复被多个父容器包含的错。
3. 在require一个文件的时候注意不要忘记在类文件的尾端return这个类，否者得不到这个类。
4. 当把item作为list的item时，主要设置容器的大小，否者在创建list的时候由于得不到容器大小会报错。

## Entity的设计 ##

Entity是一个模块的数据管理类，保存着和服务器交互的数据，所以在模块的初始化的时候就要初始化entity类，可以将其初始化放到init中，通过require然后调用其init函数，在init函数中添加监听函数。

```lua
SocketManager:addProto(s2c...., self, self.func)  --监听服务器发送 过来的数据
NetworkManager:send(c2s..., { 发送的msg })  --像服务器发送数据
 self:dispatchEvent(StarVoiceEntity.CUR_STAR_UPDATE, data) --派发事件
 StarVoiceEntity:addEventListener(StarVoiceEntity.CUR_STAR_UPDATE, self, function(data) --监听事件
        self:updateCurStar(data)
    end)
StarVoiceEntity:removeEventListener(StarVoiceEntity.CUR_STAR_UPDATE, self) --移除事件
```

## 主界面的设计 ##

1. 调用class类集成baseView类
2. 通过AlertManager:registerView(type,cname) 注册界面
3. 构造界面，通过调用父类的ctor  传入自身，ui地址进行ui的构建
4. 注册监听事件
5. 初始化UI
6. 通用物品类

## 副界面的设计 ##

1. 通过调用MEDirector:createMEModule(ui地址,boolean) 来进行UI的组合。
2. self.ui = createUISearcher(self) 来进行ui界面的索引
3. 最后return 本身用来被调用

```lua
local StarVoiceFindView = class("StarVoiceFindView", function()
    local view = MEDirector:createMEModule("LuaScript.ui.Style6.starVoiceFindUI", true)
    local findView = view:getChildByName("findView")
    findView:removeFromParent()
    return findView
end)
```

## tween ##
```lua
if self.selectTween ~= nil then
        MEDirector:killTween(self.selectTween) --释放缓动动画
        self.selectTween = nil
    end
self.selectTween = {
        target = effectSelect,  --缓动目标
        {
            repeated = 1, --循环次数 1为一次  -1为无限循环
            { duration = self.tweenTime[self.playAniTimes] / 2, alpha = 0 }, --第一次缓动
            { duration = self.tweenTime[self.playAniTimes] / 2, alpha = 1, onComplete = function() --duration:时间  alpha:透明度  onComplete:缓动执行完成的回调，
                if not isGetFlag then
                    effectSelect:setVisible(false)
                    effectSelect = nil
                end
            end }--第二个缓动
        }
    }
MEDirector:toTween(self.starTween) --执行缓动动画
--x:x轴 y:y轴，ease = { type = MEEaseType.EASE_BOUNCE_OUT, rate = 2 }缓动类型
 self.starImg:Tween {
        repeated = -1,
        { duration = 0.8, y = pos.y + 10 },
        { duration = 0.8, y = pos.y },
    } --直接给目标添加缓动动画
```

## list ##

第一种ui使用的是scrollView调用ListGridView.ToCreate

第二种向一个容器中添加一个创建好的list

```lua
--1
local createFunc = function()
        local item = StarVoiceRankItem:new()
        return item
    end

    local tempfunc = function(widget, data)
        widget:setData(data)
    end

    local rankItemList = ListGridView.ToCreate(self.ui.listRank,
            { scrollAmount = 1,
              scrollType = 2,
              interval = 10, --水平
              verInterval = 10, -- 垂直间的间距
            },
            createFunc, tempfunc)
    rankItemList:letItWork(true)
    self.rankItemList = rankItemList
--2
ListGridView:create({width = size.width , height = size.height , scrollAmount = 1 , scrollType = 1 ,interval = 6 , verInterval = 6})
list:setPosition(ccp(pos.x - anchorPoint.x * size.width, pos.y + (1- anchorPoint.y) * size.height - 20))
    self.giftScrollView:getParent():addChild(list)
    list:setTemplate(createFunc , tempfunc)
    list:setDataInfo()
    list:letItWork()
```

第二种将listGride添加到一个容器中。

```lua
local orderedMap = self:mGetSortedWeapons()
local scrollView = self.ui.svWeapon
scrollView:removeAllChildren()

local scrollViewSize = scrollView:getSize()
local midX = scrollViewSize.width / 2
local xOffset = ITEM_SIZE.width + GAP.width
local yOffset = -(ITEM_SIZE.height + GAP.height)
local container = scrollView
local contentList = {} -- 列表子元件列表
local heightCount = 0
local weaponNodeList = {}
local valueList = orderedMap:getValues()
local weaponList = valueList[index]
table.sort(weaponList, weaponSortCb)
self.weaponNodeList = weaponNodeList
local titleNode = self:mCreateTitleBar(index)
local titleSize = titleNode:getSize()
container:addChild(titleNode)
table.insert(contentList, titleNode)
titleNode:setPosition(ccp(midX, -titleSize.height / 2 + heightCount))
heightCount = heightCount - titleSize.height
local numCount = 0
for m, n in ipairs(weaponList) do
    local beginTime = ActivityUtils:stringToTimeAllowZero(n.weaponCfg.BeginShowTime)
    local curTime = ServerTimeEntity:getServerTime()
    if n.weaponCfg.IsWeapon ~= 1 or beginTime <= curTime then
        numCount = numCount + 1
        if numCount > LINE_NUM then
            numCount = numCount - LINE_NUM
            heightCount = heightCount + yOffset
        end
        local weaponNode = WeaponItemNode:new()
        weaponNode:setTriggerDetailCb(
                function(tData, infoType)
                    if tData == nil then
                        return
                    end
                    AlertManager:pushByType(ViewType.WEAPON_INFO, tData, self, infoType)
                end)
        table.insert(weaponNodeList, weaponNode)
        weaponNode:Name("weaponNode" .. n:getID())
        weaponNode:setData(n)
        table.insert(contentList, weaponNode)
        container:addChild(weaponNode)
        weaponNode:setPositionX(16 + ITEM_SIZE.width / 2 + xOffset * (numCount - 1))
        weaponNode:setPositionY(-ITEM_SIZE.height / 2 + heightCount - 5)

        applyNormalScaleBtn(
                weaponNode,
                handler(self.mTriggerWeaponNode, self))
        weaponNode:setLongTouchEnabled(true)
        weaponNode:setLongTouchSec(Configs.longTouchSce)
        weaponNode:addMEListener(MEWIDGET_LONGTOUCH, handler(self.mLongTouchWeaponNode, self))
    end
end

heightCount = heightCount + yOffset
if 1 == #valueList and #weaponList <= LINE_NUM then
    -- 尾部数量不超过一行时的修正
    heightCount = heightCount + (-100)
end

local moveUpDistance = math.abs(heightCount) + 150 -- 增加列表底部的间隙
moveUpDistance = math.max(scrollViewSize.height, moveUpDistance)
for i, v in ipairs(contentList) do
    v:setPositionY(v:getPositionY() + moveUpDistance) -- 全部往上移动至y坐标均为正数
end
scrollView:setInnerContainerSize(ccs(scrollViewSize.width, moveUpDistance))
```



## PurchaseHelper ##

通过调用PurchaseHelper来判断购买消耗是否足够。

## 新手 ##

继承BaseOrderedGuide类。

```lua
local cbList = self:mCreateOrderedCbList()
local stepAdder = self:mGetStepAdder()
cbList:add(
            function() 
                startCmd(CmdName.enter1v1PvpRoom)
                cbList:callNext()
            end) --添加函数
```

1. stepAdder:addDetectScene(cbList, ViewType.WEAPON_BROWSER, 4) ：参数2监听界面的打开
2. self:mReachGoal()完成目标条件 即这个新手已经完成
3. self:mFinish()新手完成且结束新手引导
4. MEDirector:lockScene()锁定场景  屏蔽所有监听事件

## 通用物品类 ##

```lua
local itemInfo = BagCell.CreateInfoByTempID(data.Reward, data.RewardNum)
    local item = BagCell:create({ info = itemInfo, isNeedCount = true, isNoQualityIcon = true, isSmall = 3, isNoBG = true })
    item:setCountPosScale(ccp(20, -25), 0.6)
    item:enableTips()
    self.ui.imgItemIcon:addChild(item)
```

## 排行榜数据获取 ##

```lua
local rangeType, rankType = RangeType.allArea, RankType.starVoiceLeaderboard
    if not RanklistProcessor:getDataIsPrepared(rangeType, rankType) then
        RanklistProcessor:prepareData(rangeType, rankType, function(...)
            if isValid(self) then
                self:updateRankList()
            end
        end)
    else
        self:updateRankList()
    end
local rankDataList = RanklistProcessor:getRankList(RangeType.allArea, RankType.starVoiceLeaderboard)
local newRankData = {}
local newRankDataList = {}
for i, rankData in ipairs(rankDataList) do
    newRankData = {}
    newRankData.rankData = rankData
    newRankData.rankRewardData = StarVoiceEntity:getRankTempByRank(rankData:getRanking()).rewards
    newRankData.extraCond = StarVoiceEntity:getRankTempByRank(rankData:getRanking()).extraCond
    table.insert(newRankDataList, newRankData)
end

local rankData = RanklistProcessor:getOwnedRank(RangeType.allArea, RankType.starVoiceLeaderboard)

ui.txtMyName:setText(rankData:getName())
ui.imgMyIcon:setURL(rankData:getHeadIcon())
ui.imgIconFrame:setTexture(TemplateManager:getHeadFramePathById(rankData:getHeadFrame()))
ui.txtMyRankNum:setText(rankData:getRanking())
ui.txtMyStarNum:setText(rankData:getPoint())
local rankData = StarVoiceEntity:getRankTempByRank(rankData:getRanking())
if not self.myRankListReward then
    self:createMyRewardList(rankData.rewards)
else
    self.myRankListReward:setDataInfo(rankData.rewards, true)
end
```

## timer ##

```lua
function WeaponMakeView:manageTimer(remainTime)
    if self.timer then
        MEDirector:removeTimer(self.timer)
        self.timer = nil
    end
    local count = remainTime
    if not self.timer then
        self.timer = MEDirector:addTimer(1000, remainTime, function()
            self:canFreeGet(true)
            self.canFree = true
        end, function()
            count = count - 1
            local hour, min, sec = TimeUtil:translateSecToHour(count)
            self.ui.txtTime:setText(string.format(LanguageConfig.PET_MAKE_TIME, TimeUtil:numberFormat(hour), TimeUtil:numberFormat(min), TimeUtil:numberFormat(sec)))
        end)
    end
end
```

## 字体颜色 ##

```lua
Text(QualityType.WEAPON_QUALITY_TXT[self.lowGet[type]]):FontColor(QualityType.color[self.lowGet[type]]):Stroke(QualityType.strokeColor[self.lowGet[type]], 2)
```

## baseview ##
1. setBGBlur(true) 背景模糊，要放在ctor的前面
2. ctor(self, "LuaScript.ui.Style6.starVoiceMainUI", true)  参数3，为真的时候点击屏幕不自动pop，否则自动pop
3. onExit() 退出的时候调用
4. refresh() 重新得焦的时候调用 call when pop to top
5. mLock(lockTime) 锁定面板不能点击，超时释放

## 时间处理 ##

timeutil 时间处理类

## 坐标转换 ##

带上AR表明根据锚点来转换

不带AR以左下角来转换

```lua

local selectPos = imgSelect:convertToWorldSpaceAR(ccp(0, 0))
local starPos = imgGet:convertToWorldSpaceAR(ccp(0, 0))
selectPos = selectedImg:getParent():convertToNodeSpaceAR(selectPos)
starPos = starImg:getParent():convertToNodeSpaceAR(starPos)
selectedImg:setPosition(selectPos)
starImg:setPosition(starPos)
```