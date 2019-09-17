---
layout:     post
title:      法线贴图
subtitle:   opengl
date:       2019-9-17
author:     Jow
header-img: img/about-bg-walle.jpg
catalog: 	 true 
tags:
    - OpenGL

---

### 目录
1. 法线贴图


> Always study， then you will get you want.

> 如何使用法线贴图给物体添加更多的细节？

## 法线贴图
法线贴图，顾名思义，就是将每一个片元(像素)的法线值保存成一张图片。法向量的xyz坐标对应图的rgb值。这样会存在一个问题，就是我们的法线xyz的取值范围是[-1,1]，而rgb的值的取值范围[0,1]。所以我们需要转换一下： rgb = (xyz + 1) / 2.

物体的法线在世界空间的不同位置是不同的，我们如何才能用一张图表示物体的法线，使它能够在所有的情况下都适用呢?法线贴图是贴在物体表面的，我们在一个固定的坐标系中生成法线贴图，再通过一个转换矩阵将法线转换到世界空间中，这样就可以了。

一般法线贴图的坐标系统称作TBN坐标系，对应我们熟悉的xyz轴。N轴表示的是图元三角形表面的法向量。还有两个轴是切线轴(tangent)和副切线轴(bitangent)。通常我们采用的是和表面纹理坐标一致的轴作为T轴和B轴，即U轴对应T轴，V轴对应B轴，这样我们就可以用UV值来计算转换矩阵了。

> TBN坐标系的存在意义就是简化计算变换矩阵的，所以我们的TBN轴就必须按照约定俗成的规范来。

[https://www.jianshu.com/p/eea4d8582499](https://www.jianshu.com/p/eea4d8582499 "法线贴图")