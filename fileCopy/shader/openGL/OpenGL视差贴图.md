---
layout:     post
title:      视差贴图
subtitle:   opengl
date:       2019-9-17
author:     Jow
header-img: img/about-bg-walle.jpg
catalog: 	 true 
tags:
    - OpenGL

---

### 目录
1. 视差贴图
2. 陡峭视差贴图（Steep Parallax Map）
3. 视差遮蔽贴图（Parallax Occlusion Mapping）


> Always study， then you will get you want.

> 如何使用视差贴图得到更好的表面凹凸效果？

## 视差贴图
视差贴图，本质上，是一种位移贴图（Displacement Mapping）。它的原理，是根据视线方向，对当前看到的像素位置进行一定的坐标偏移，显示偏移过后的像素颜色。这样，就能有产生这种凹凸落差很大的视觉效果。

> 所谓切线空间，就是以表面法向量为z轴，纹理坐标UV为系统x轴和y轴所组成的空间。这个空间存在的唯一目的是在这个空间中进行法线贴图、视差贴图等等的工作十分方便。

## 陡峭视差贴图
陡峭视差贴图的原理，是设定一个采样层数，每一层都对相应纹理采样，当采样到小于当前层深度的纹理坐标时，返回此纹理坐标。

## 视差遮蔽贴图
视差遮蔽贴图的原理，就是取与交点相邻的两个层的层深度和采样深度，计算出两个深度值的权重，根据权重采样两个纹理坐标之间某个位置的纹理坐标。

[https://www.jianshu.com/p/98c137baf855](https://www.jianshu.com/p/98c137baf855 "视差贴图")