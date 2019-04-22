---
layout:     post
title:      DDT Battle
subtitle:   翼精灵的添加
date:       2019-4-12
author:     Jow
header-img: img/post-bg-2015.jpg
catalog: 	 true 
tags:
    - DDT
    - Battle

---

### 目录
1. 翼精灵的添加

> 今天开始改战斗相关的bug了，框架和逻辑还是挺复杂的，自己从改bug开始，不断的进行模块和功能的分析，发现自己的疑问，并解答自己的疑问

## BattleManager

## RoomManager

## RoomMatchView


## FightMultiPlayer
从fightMultiPlayer这个开始入手，在这个类中，有一个loadPlayer的方法。这个方法获取到战斗中玩家的信息，进行玩家的创建。然后进行展示。

## GamePlayer
游戏中的玩家类，

## Living
玩家和怪物基类，

## Avarta
玩家所有形象的展示类


## 技能相关
BattleManager这个战斗管理类
1. onMsgCreateGame 接收到游戏相关的msg
2. RoomManager:freshRoomPlayerFightInfo   刷新玩家的信息,在里面存储一份 ourTeam 和 enemyTeam 