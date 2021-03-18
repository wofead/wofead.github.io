---
layout:     post
title:      DDT activity
subtitle:   DDT
date:       2019-8-29
author:     Jow
header-img: img/post-bg-infinity.jpg
catalog: 	 true 
tags:
    - DDT

---

### 目录
1. ActivityGroupEntity
2. ActivityBottomTabController
3. activityEntity
4. 活动开发总结


> study and think. then you will get more.


## ActivityGroupEntity
一个新的活动的添加，首先需要在ActivityGroupEntity中进行注册，包含是否开启，是否需要提醒以及界面的获取。策划需要在mainbtn表中，配置组为6的一级菜单，然后新开一个组，让这个组属于这个一级菜单。这里所有的界面都是通过btnname来进行获取的，即把btnname作为键值，来进行数据的获取。

在activity中一共用两个页签管理器，一个是下面的页签管理器，一个是左侧的页签管理器，下面的ActivityBottomTabController类通过调用ActivityGroupEntity类中的getOpenList函数来判断这个组是否有活动开启，开启的话就打开这个活动。AssemblyEntry类用来管理和添加左边item。这个就是组配置中getView返回的界面，就是左侧的item。
```lua
-- 国庆阵营对抗活动：组的配置
nationalDayActBtn = {
    isOpen = function()
        return activityGroupEntity:isGroupOpen(ActivityGroupType.nationalDayActBtn)
    end,
    isRemine = function()
        return activityGroupEntity:checkGroupRemine(ActivityGroupType.nationalDayActBtn)
    end,
    getView = function()
        return AssemblyEntry:create(ActivityGroupType.nationalDayActBtn)
    end
},
-- 里面具体的活动
nationalDayScoreBtn = {
    isOpen = function()
        return ActivityEntity.nationalDayPersonalScore ~= nil
    end,
    isRemine = function()
        return false
    end,
    getView = function()
        return NationalDayScoreView:create()
    end
},
```

## ActivityBottomTabController
在bottom管理类中，注意区分是否存在二级界面。将一级界面和二级界面进行分开加载。

## activityEntity
我们还需要在activityentity中缓存一份我们的活动。这个缓存的数据是通过activityid在activity表中取到的activity数据。

## 活动开发总结
```lua
RanklistProcessor:prepareDataDirect(RangeType.allArea, self.nationalDayCampData.rankType, function(...)
        self:dispatchEvent(NationDayEvent.TASK_SCORE_RANK_UPDATE)
    end)

myRankData = RanklistProcessor:getOwnedRank(RangeType.allArea, self.nationalDayCampData.rankType)
```
1. 排行榜：获取玩家的排行榜，是根据排行榜管理类来获取的。通过RanklistProcessor类中的prepareDataDirect函数，加上排行榜类型，以及type来获取排行榜信息。
2. 排行榜我的数据：一样的通过ranklistProcessor类中函数来获取，这次是通过getOwnRank来获取数据。返回的数据类型使用rankdata这个类封装的。
3. 界面的创建，可以直接return一个MEImage:create(),然后再ctor中createMEModule("uipath")，来将view添加到界面中。一般情况下，监听一下MEWIDGET_EXIT函数，界面退出的时候调用。
4. 空的物品item创建，直接在创建的时候传入一个空的info，然后再setData的时候，将info作为参数传过去，这个一般用于list。
5. list创建：ListGridView.ToCreate(self.ui.listWinReward, { scrollType = 1, interval = 10 }, createFunc, tempfunc)
6. timer的创建MEDirector:addTimer(1000,remainTime,completeCallback,eachDelayCall),删除的时候使用removeTimer。