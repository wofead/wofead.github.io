---
layout:     post
title:      c++一个类或者一个对象占用的字节数
subtitle:   c++
date:       2019-8-28
author:     Jow
header-img: img/post-bg-infinity.jpg
catalog: 	 true 
tags:
    - c++

---

### 目录
1. ActivityGroupEntity
2. ActivityBottomTabController


> study and think. then you will get more.


## ActivityGroupEntity
一个新的活动的添加，首先需要在ActivityGroupEntity中进行注册，包含是否开启，是否需要提醒以及界面的获取。策划需要在mainbtn表中，配置组为6的一级菜单，然后新开一个组，让这个组属于这个二级菜单。这里所有的界面都是通过btnname来进行获取的，即把btnname作为键值，来进行数据的获取。

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