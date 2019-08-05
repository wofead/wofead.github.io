---
layout:     post
title:      界面的管理
subtitle:   AlertManager
date:       2019-8-5
author:     Jow
header-img: img/lua-home-bg-o.jpg
catalog: 	 true 
tags:
    - DDT
    - AlertManager

---

### 目录
1. 界面的管理
2. pushByType(nType, ...)
3. popByType(nType)


## 界面的管理
在DDT中，界面的管理使用的是统一的AlertManager类来进行加载和移除的，通过调用push函数来加载一个界面，通过pop函数来关闭一个界面。

## pushByType(nType, ...)
打开一个界面的步骤：
1. 判断这个界面是否可打开，这个原因可能是这个界面出现了问题，等等，这个控制在SwitchManager管理器中，它有三种状态open，grey，hide。我们将nType传过去，进行判断。
2. 进行GruopEntry的判断，这个如要是用来判断类似于商城整合类的，因为之前这个界面有可能是独立，这个时候如果是组里的话，要走另一套流程。
3. 需要服务器控制才能打开的界面，如果这个界面需要服务器判断之后才能打开，这个时候就要先给服务器发送消息，等待其回执之后才能决定是否打开界面（我觉得这个是多余的，上面就可以实现这个功能，不过也有可能是这个界面的打开需要有初始数据，这个时候要等数据来了再进入这个界面）
4. 接下里判断是否是能够重复打开的界面，如果是就要新建新的界面，这里需要注意一点，为了防止上一个界面被点穿，在这里获取上一个界面，并且将其这设置为被覆盖。并派发这个界面已经关闭的事件。
5. 接下来获取这个界面，如果这个类型的界面已经注册过了，那就创建这个界面，否则报错，打日志。然后进行放点穿和返回这个界面。在这里我们一样要把新常见的界面进行存储管理。为了防止我们打开了太多的界面，我们进行界面数量打开的限制，如果界面打开太多的话，我们就进行pop出最先打开的那个界面。调用底层类baseView:addToSceneShow()函数添加到场景中

```lua
if self:checkIsCanRepeatExist(nType) then
        -- todo: Already Exists
        local isExists, ui = self:checkExists(nType)
        if isExists then
            --防止点穿之前上面的界面
            self.alertList = self.alertList or MEArray:new()
            local per       = self.alertList:back()
            if not tolua.isnull(per) then
                per:cover(ui:isFullScreen())
                per:setIsCovered(true)
                MEDirector:dispatchGlobalEventWith(ViewEvent.leaveComplete, per:getType(), per)
            end

            self.alertList:removeObject(ui)
            self.alertList:pushBack(ui)
            ui:ZO(ui:ZO())
            ui:setVisible(true and self.unlock)
            ChatManager:refreshChatCmpState(nType)
            return ui
        end
    end
    local viewClass = self.registerViews[nType]
    if viewClass then
        local path = viewClass.strRequirePath --每次都那新的文件重新加载，这个一般用在开发模式，这样修改了文件能够立即加载新的
        if path and viewClass == require(path) then -- if require not return self will return it's super, which will cause error.
            self.registerViews[nType] = nil
            package.loaded[path] = nil
            require_debug(path)
            viewClass = self.registerViews[nType] or viewClass
        end
        ui = viewClass:create(...)
    else
        -- class not found
        print(string.format("Class with type:%d did not register.", nType))
        return
    end

    ui:setVisible(self.unlock)
    self.alertList = self.alertList or MEArray:new()
    local per       = self.alertList:back()--弹出最后一个界面
    if not tolua.isnull(per) then
        per:cover(ui:isFullScreen())
        per:setIsCovered(true)
        MEDirector:dispatchGlobalEventWith(ViewEvent.leaveComplete, per:getType(), per)
    end

    self.alertList:pushBack(ui)
    self:restrictViewCount(nType)

    if ui:isFullScreen() and not tolua.isnull(SceneManager:currentScene()) then 
        SceneManager:currentScene():setVisible(false)
    end

    ui.isAddToSceneShow = false
    ui:addToSceneShow() --添加到场景中
```

## popByType(nType)
在pop一个界面之前首先判断这个界面是否是不需要pop的界面，或者被锁定不能pop的界面，然后再对其进行pop。
在pop一个界面的时候我们需要处理几个细节问题：
1. 判断是否存在pop的这个界面
2. 而pop之前先show之前的界面，并且默认刷新这个界面，除非这个界面已经表示不需要刷新了。
3. 关闭这个界面的时候派发关闭了这个界面的函数，并且执行回调函数。
4. 显示最上面的界面，解除被cover，并且调用refresh()函数。并派发进入场景的函数。

```lua
function AlertManager:popByType(nType)
    if nType == ViewType.MAIN_SCENE then return end
    print_("AlertManager:popByType------------------- ", nType, self.unlock)
    if not self.unlock then
        table.insert(self.needPopList, nType)
        return
    end
    self.alertList = self.alertList or MEArray:new()
    local needDelay = false
    local doAct = true
    local popSuccess = false -- 是否真的有弹出窗口
    for ui in self.alertList:riterator() do 
        if not tolua.isnull(ui) and ui.type == nType then 
            needDelay = true
            self:popByView(ui, function()
                doAct = ui.doAct
                self:showTop(doAct)
            end)
            popSuccess = true
            break 
        end
    end
    
    if not popSuccess then return end
    
    if not needDelay then
        self:showTop(doAct)
    end
end

-- 显示当前栈顶的窗口
function AlertManager:showTop(doAct)
    doAct = doAct ~= false
    self.alertList = self.alertList or MEArray:new()
    local current = self.alertList:back()  
    if current == nil then
        if SceneManager:currentScene() then
            MEDirector:dispatchGlobalEventWith(ViewEvent.enterComplete, SceneManager:currentScene():getType(), SceneManager:currentScene())
        end
    end
    if BattleManager.isSimulation and current and (current.type == ViewType.ZONEMAIN or current.type == ViewType.STAR_SHOW) then
        return
    end
    self:showUI(current, doAct)
end

function AlertManager:showUI(current, doAct)
    while current do
        if not tolua.isnull(current) then 
            current:setVisible(true and self.unlock)
            current:setIsCovered(false)
            current:refresh()
            ChatManager:refreshChatCmpState(current.type)
            if current.type == ViewType.PLAYER_EQUIP then 
                local view = self.alertList:objectAt(self.alertList:size() - 1)
                view:setVisible(true and self.unlock)
                view:refresh()
                ChatManager:refreshChatCmpState(view.type)
            end

            if not current:getIsEntering() then
                MEDirector:dispatchGlobalEventWith(ViewEvent.enterComplete, current:getType(), current)
            end
            
            break 
        end
        self.alertList:popBack()
        current = self.alertList:back()  
    end
    
    if current and doAct then
        current:addToSceneShow()
    end

    local currentScene = SceneManager:currentScene()
    if not currentScene then return end

    if current and current:isFullScreen() then 
        currentScene:setVisible(false)
    else 
        currentScene:setVisible(true)
        currentScene:refresh()
        currentScene:setScale(1)
    end
end
```




