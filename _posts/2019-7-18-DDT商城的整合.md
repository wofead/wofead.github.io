---
layout:     post
title:      DDT商城的整合
subtitle:   data structure
date:       2019-7-18
author:     Jow
header-img: img/lua-home-bg-o.jpg
catalog: 	 true 
tags:
    - DDT
    - 商城

---

### 目录
1. GroupEntity类的构建
2. 公共属性的总结
3. 界面的获取
4. 界面的提醒
5. 界面的控制
6. 开关的设置
7. 总结

> I love my famly, when i was in trouble, they always tell me how to deal this problem. Sometimes i really feel sad, because i don't know how to live can make happy. But it will make me relax by calling them. So meaning of you live in the world not only due to yourself, everyone is not a alone island, there also is the sea around it. It's what our come here.

实际上，如果时间充足的情况下，可以全重写的，但是代价太高了，自己实在是hold不住，就这样，已经让自己精疲力竭了。自己已经在这上面花了2周的时间了，但是实际上只完成了一半的工作，这中间穿插着自己低效的工作，也因为自己生病的原因吧，但是这些都不应该是借口，自己最近的工作态度真的很成问题。

## GroupEntity类的构建
首先声明一级菜单和二级菜单，其实这里使用键值对应该是最好的方式，但这里我才用的是数组的方式，应该是有一些不妥的。
```lua
StoreGroupEntity.RIGHT_TAB = {
    ONSALE = 1,
    GREEDY = 2,
    PROP = 3,
    CULTIVATE = 4,
    MODE = 5
}

StoreGroupEntity.BOTTOM_TAB = {
    WEAPON = 1,
    PET = 2,
    Orb = 3,
    Fashion = 4,
    Union = 5,
    Ancient = 6,
    Athletic = 7,
    Rank = 8,
    Salvo = 9,
    Warrior = 10,
}
```
接下来就是每个商店的配置了，把这些商店按照一定的属性进行配置。

## 公共属性的总结
这里为了能够通用的访问和控制每一个商店界面，我们对商店进行了配置。其中的属性包含：
* index：商店的索引，通过这个数据来在界面最获取这条商店数据
* refresh：刷新方式，每次进这个界面信息就刷新一次，还是第一次进这个界面刷新，还是不刷新
* itenMainMenuIndex:用来向服务器请求刷新那个商店的index值
* isOpen：是个表，里面放的是判断这个商店是否开启的多重条件
* remind：是否提示的条件
* viewType：之前跳转的界面

```lua
ONSALE = {
        index = StoreGroupEntity.RIGHT_TAB.ONSALE,
        refresh = 2,
        itemMainMenuIndex = StoreNewEntity.ItemMainMenuIndex.ONSALE,
        isOpen = {
            otherCondition = function()
                return StoreNewEntity:checkOnsaleOpen()
            end
        },
        remind = function()
            return StoreNewEntity:IsRemindHotShopTip()
        end,
        viewType = ViewType.STORE,
    },
WEAPON = {
        index = StoreGroupEntity.BOTTOM_TAB.WEAPON,
        parent = StoreGroupEntity.RIGHT_TAB.CULTIVATE,
        refresh = 1,
        isOpen = {
            functionId = FUNCTIONID.weapon_shop,
            switchType = SwitchType.WEAPON_GAIN
        },
        remind = function()
            return WeaponEntity:getMakeCanFree()
        end,
        viewType = ViewType.WEAPON_MAKE,
        view = function()
            return WeaponMakeView:new()
        end
    },
```
接下来就是对数据的处理函数，包含：
* 判断一个商店是否开启
* 这个组是否有商店开启
* 红点的开启
* 获取数据的相关函数

## 界面的获取
因为是整合过来的，这也就导致他们之前可能是独立的界面，这个时候就需要对这些界面进行简单的重构，让它们能够作为整合界面的一部分来使用。
```lua
function StoreGroupEntity:createView(btnData)
    if nil == btnData then
        return
    end
    if btnData.viewType == ViewType.STORE then
        --因为本身就已经存在了，不需要新建
        return nil
    end
    return btnData.view()
end
```

## 界面的提醒
每个界面都存在一个remind，直接调用remind就行了，如果是对组的判断的话，需要对组里面的每个成员进行调用判断。

## 界面的控制
通过在mainview中缓存界面，如果这个界面已经缓存起来了，那么直接调用就行了，否则进行新的界面创建和缓存。

## 开关的设置
开关方面通过isOpen来控制。

## 总结
在涉及到resBar的时候，可以采用同样缓存的方式来进行存储，然后到了相关的界面就进行显示，其他的隐藏。
