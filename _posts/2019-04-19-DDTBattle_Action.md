---
layout:     post
title:      DDT Battle
subtitle:   Action
date:       2019-4-19
author:     Jow
header-img: img/post-bg-2015.jpg
catalog: 	 true 
tags:
    - DDT
    - Battle
    - Action

---

### 目录
1. 问题的解决

> 镜头的锁定bug，不知道为什么镜头一直被锁定，当玩家死亡，然后镜头就一直被锁定

## 问题的解决
这个问题是因为，LivingGhostCamaraAction这个行为的生成却没有完成导致的，解决这个问题的方法就是查找那个地方在不该产生这个行为的时候产生了这个行为。

最终发现在Living类中，dispear函数中 ，假死的时候不应该产生这个行为但是还是产生了。
