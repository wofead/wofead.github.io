---
layout:     post
title:      场景基类
subtitle:   BaseView
date:       2019-8-5
author:     Jow
header-img: img/lua-home-bg-o.jpg
catalog: 	 true 
tags:
    - DDT
    - BaseView

---

### 目录
1. 场景基类
2. 点击界面自动pop
3. 界面退出和进入事件的监听
4. 场景的进入
5. 锁状态下的覆盖
6. 添加场景到顶层
7. 设置全屏
8. 焦化

## 场景基类
在DDT中，所有的view界面都会继承一个场景基类，叫做BaseView，这个基类基本存储着一个场景的所有共同信息。baseview继承的是cocos的MENode组件。
一个基本的场景一般包含以下的属性：
* self._isFullScreen = self._isFullScreen or false:全屏
* self.isBGBlur = self.isBGBlur or false --背景焦化
* self.viewPath = viewPath：界面的ui地址 
* self.showCount  = 1：？？？
* self.showCount  = 1：是否可用
* if self.doAct == nil then self.doAct = true end   ---关闭时，是否刷新Top窗口
* if self.isBackGroundOpacity == nil then self.isBackGroundOpacity = true end --背景透明度
* self.isEntering = false：正在进入
* self.isCovered = false：是否被覆盖

关于viewpath的加载使用：
```lua
view = MEDirector:createMEModule(viewPath, self.isUISingleton)
if view and not view:getParent() then --添加到场景中
    self:addChild(view, 1)
end
```


一般在cocos堆ui建立索引可以使用createUISearcher函数，将自身self传递过去就行了。
```lua
function createUISearcher(node)
    local mt = { }
    mt.__index = function(table, key)
        local childNode = node:getChildByName(key)
        table[key] = childNode
        return childNode
    end

    local result = { }
    setmetatable(result, mt)
    return result
end
```

## 点击界面自动pop
notAutoPop是在类的构建的时候传递过来的参数。
```lua
if not notAutoPop then 
    applyNormalScaleBtn(view, function() 
        AlertManager:popByType(self.type)
    end, nil, nil, nil, false)
    view.clickAudioPath = nil

    local bg = self.view:getChildByName("bg")
    if bg then
        bg:setTouchEnabled(true)
        bg:addMEListener(MEWIDGET_CLICK, function ()
            -- do nothing
        end)
    end
end
```

## 界面退出事件的监听
```lua
 local node = CCNode:create():AddTo(self)
node:addMEListener(MEWIDGET_EXIT, function ()
    if not self.isSetEnable then
        if not Configs.isOnGuide then
            MEDirector:setTouchEnabled(true)
        end
    end

    self:onExit()
end)
node:addMEListener(MEWIDGET_ENTER, function() self:onEnter() end)
```

## 场景的进入
```lua
self:timeOut(function()
    MEDirector:clearLuaBegin()      
end, 5)
self:mLock()
-- 添加到舞台才会触发隔帧调用
nextFrameCall(
    function() 
        -- 隔帧检查没有进入动作的话直接解锁
        if self.actionTarget == nil then
            self:mReleaseLock()
        end
        MEDirector:dispatchGlobalEventWith(ViewEvent.addToStage, self:getType(), self) -- 发送添加到舞台的事件
    end, 
    self)


-- 锁定面板 -->
-- 锁定面板不能点击，超时释放
function BaseView:mLock(lockTime)
    if self.isLock then return end
    self.isLock = true
    self:mGetLockCover():setVisible(true)
    self.lockPanelTimeOutAction = self:timeOut(
        function() 
            self:mReleaseLock() 
        end, lockTime or LOCK_TIME)
end
-- 释放锁定
function BaseView:mReleaseLock()
    if self.isLock == false then return end
    self.isLock = false
    self:mGetLockCover():setVisible(false)
    if self.lockPanelTimeOutAction ~= nil then
        self:stopAction(self.lockPanelTimeOutAction)
        self.lockPanelTimeOutAction = nil
    end
end

function BaseView:ShowCall(call)
    MEDirector:setFPS(Configs.BestFPS)
    local packCall = function()
        MEDirector:setFPS(Configs.uiFPS)

        self.isAddToSceneShow = true

        if call then call() end
        self:onShowComplete()
    end

    if self.showAction == ViewShowAction.Action1 then
        self:ShowAction1(self.actionTarget, packCall)
    elseif self.showAction == ViewShowAction.Action2 then
        self:ShowAction2(self.actionTarget, packCall)
    else
        packCall()
    end
end

function BaseView:addToSceneShow()
    if self.showCount == nil or self.showCount == 0 then return end
    
    if self.showCount > 0 then
        self.showCount = self.showCount - 1
    end

    if self.actionTarget then
        self.actionTarget:setScale(0)
    end
    if not Configs.isOnGuide then
        MEDirector:setTouchEnabled(false)
    end
    self.isSetEnable = false
    self.isEntering = true
    self:timeOut(function ()
        MEDirector:setFPS(Configs.BestFPS)
        self:ShowCall(function ()
            if not Configs.isOnGuide then
                MEDirector:setTouchEnabled(true)
            end
            self.isEntering = false 
            self.isSetEnable = true
            self.isShowing = false
            MEDirector:setFPS(Configs.uiFPS)
            self:mReleaseLock()
            self:mOnEnterActionComplete()
            MEDirector:dispatchGlobalEventWith(ViewEvent.enterComplete, self:getType(), self)
        end)
    end)
end
```

## 锁状态下的覆盖
```lua
-- 若UI中没有覆盖节点lockCover则动态创建
function BaseView:mGetLockCover()
    local lockCover = self.lockCover
    if lockCover == nil then
        lockCover = MEPanel:create()
        -- lockCover:setBackGroundColorType(ME_LAYOUT_COLOR_SOLID)
        lockCover:setTouchEnabled(true)
        -- lockCover:setColor(ccc3(0, 0, 0))
        lockCover:setAlpha(0)
        lockCover:setSize(Configs.designSize)
        self:addChild(lockCover, 100000)
        self.lockCover = lockCover
    end
    return lockCover
end
```

## 添加场景到顶层
```lua
function BaseView:addToTop(order, pat)
    order = order or 9998
    self:updateBGBlur(1)      
    if not pat then 
        pat = SceneManager:mainScene()
    end                 
    pat:addChild(self, order)
end
```

## 设置全屏
function BaseView:setFullScreen(isfull)
    self._isFullScreen = isfull

    CCNode:create(): Size(Configs.designSize)
                : AddTo(self, -9999)
                : setTouchEnabled(true)
end

## 焦化
```lua
function BaseView:updateBGBlur(dt)
    self.elapse = self.elapse or 0
    self.elapse = self.elapse + (dt or 0)

    local isBGBlur = self.isBGBlur
    -- isBGBlur = false
    if isBGBlur and self.elapse >= 0.033 and not self.bgBlurImg then 
        self.elapse = 0

        if false then 
            if not self.bgBlurImg then 
                self.bgBlurImg = MEImage:create()
                self.bgBlurImg:setSize(ccp(Configs.designSize.width + 2, Configs.designSize.height + 2))
                self.bgBlurImg:setFlipY(true)
                self.bgBlurImg:setPosition(ccp(Configs.designSize.width / 2, Configs.designSize.height / 2))
                self:addChild(self.bgBlurImg, -1)
            end

            local tex = self:genBGBlur()
            self:setBGBlurTexture(tex)
        else 
            if not self.bgBlurImg then 
                self.bgBlurImg = BlurSprite:create(3, 0, 0, 3)
                self.bgBlurImg:setPosition(ccp(Configs.designSize.width / 2, Configs.designSize.height / 2))
                self:addChild(self.bgBlurImg, -1)
                self.bgBlurImg:setColor(ccc3(222, 222, 222))
            else 
                self.bgBlurImg:reset()
            end
            -- self.bgBlurImg:setTouchEnabled(true)
        end
    end
end
```