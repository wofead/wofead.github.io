# 基本图像渲染

# Sprite 组件参考

Sprite（精灵）是 2D 游戏中最常见的显示图像的方式，在节点上添加 Sprite 组件，就可以在场景中显示项目资源中的图片。

点击 **属性检查器** 下面的 **添加组件** 按钮，然后从 **渲染组件** 中选择 **Sprite**，即可添加 Sprite 组件到节点上。

## Sprite 属性

| 属性             | 功能说明                                                     |
| ---------------- | ------------------------------------------------------------ |
| Atlas            | Sprite 显示图片资源所属的 [Atlas 图集资源](https://docs.cocos.com/creator/manual/zh/asset-workflow/atlas.html)。（Atlas 后面的 **选择** 按钮，该功能暂时不可用，我们会尽快优化） |
| Sprite Frame     | 渲染 Sprite 使用的 [SpriteFrame 图片资源](https://docs.cocos.com/creator/manual/zh/asset-workflow/sprite.html)。（Sprite Frame 后面的 **编辑** 按钮用于编辑图像资源的九宫格切分，详情请参考 [使用 Sprite 编辑器制作九宫格图像](https://docs.cocos.com/creator/manual/zh/ui/sliced-sprite.html)） |
| Type             | 渲染模式，包括普通（Simple）、九宫格（Sliced）、平铺（Tiled）、填充（Filled）和网格（Mesh）渲染五种模式 |
| Size Mode        | 指定 Sprite 的尺寸 `Trimmed` 表示会使用原始图片资源裁剪透明像素后的尺寸 `Raw` 表示会使用原始图片未经裁剪的尺寸 `Custom` 表示会使用自定义尺寸。当用户手动修改过 `Size` 属性后，`Size Mode` 会被自动设置为 `Custom`，除非再次指定为前两种尺寸。 |
| Trim             | 勾选后将在渲染时去除原始图像周围的透明像素区域，该项仅在 Type 设置为 Simple 时生效。详情请参考 [图像资源的自动剪裁](https://docs.cocos.com/creator/manual/zh/asset-workflow/trim.html) |
| Src Blend Factor | 当前图像混合模式                                             |
| Dst Blend Factor | 背景图像混合模式，和上面的属性共同作用，可以将前景和背景 Sprite 用不同的方式混合渲染，效果预览可以参考 [glBlendFunc Tool](http://www.andersriggelsen.dk/glblendfunc.php)。 |

添加 Sprite 组件之后，通过从 **资源管理器** 中拖拽 Texture 或 SpriteFrame 类型的资源到 `Sprite Frame` 属性引用中，就可以通过 Sprite 组件显示资源图像。

如果拖拽的 SpriteFrame 资源是包含在一个 Atlas 图集资源中的，那么 Sprite 的 `Atlas` 属性也会被一起设置。

## 渲染模式

Sprite 组件支持五种渲染模式：

- `普通模式（Simple）`：根据原始图片资源渲染 Sprite，一般在这个模式下我们不会手动修改节点的尺寸，来保证场景中显示的图像和美术人员生产的图片比例一致。
- `九宫格模式（Sliced）`：图像将被分割成九宫格，并按照一定规则进行缩放以适应可随意设置的尺寸(`size`)。通常用于 UI 元素，或将可以无限放大而不影响图像质量的图片制作成九宫格图来节省游戏资源空间。详细信息请阅读 [使用 Sprite 编辑器制作九宫格图像](https://docs.cocos.com/creator/manual/zh/ui/sliced-sprite.html#-) 一节。
- `平铺模式（Tiled）`：图像将会根据 Sprite 的尺寸重复平铺显示。如果 SpriteFrame 包含 [九宫格配置](https://docs.cocos.com/creator/manual/zh/ui/sliced-sprite.html)，平铺时将保持周围宽度不变，而其余部分重复。

- `填充模式（Filled）`：根据原点和填充模式的设置，按照一定的方向和比例绘制原始图片的一部分。经常用于进度条的动态展示。
- `网格模式（Mesh）`：必须使用 **TexturePacker 4.x** 以上版本并且设置 ploygon 算法打包出的 plist 文件才能够使用该模式。

### 填充模式（Filled）

`Type` 属性选择填充模式后，会出现一组新的属性可供配置，让我们依次介绍它们的作用。

| 属性        | 功能说明                                                     |
| ----------- | ------------------------------------------------------------ |
| Fill Type   | 填充类型选择，有 `HORIZONTAL`（横向填充）、`VERTICAL`（纵向填充）和 `RADIAL`（扇形填充）三种。 |
| Fill Start  | 填充起始位置的标准化数值（从 0 ~ 1，表示填充总量的百分比），选择横向填充时，`Fill Start` 设为 0，就会从图像最左边开始填充 |
| Fill Range  | 填充范围的标准化数值（同样从 0 ~ 1），设为 1，就会填充最多整个原始图像的范围。 |
| Fill Center | 填充中心点，只有选择了 `RADIAL` 类型才会出现这个属性。决定了扇形填充时会环绕 Sprite 上的哪个点，所用的坐标系和 [Anchor 锚点](https://docs.cocos.com/creator/manual/zh/content-workflow/transform.html#-anchor-) 是一样的。 |

#### Fill Range 填充范围补充说明

在 `HORIZONTAL` 和 `VERTICAL` 这两种填充类型下，`Fill Start` 设置的数值将影响填充总量，如果 `Fill Start` 设为 0.5，那么即使 `Fill Range` 设为 1.0，实际填充的范围也仍然只有 Sprite 总大小的一半。

而 `RADIAL` 类型中 `Fill Start` 只决定开始填充的方向，`Fill Start` 为 0 时，从 x 轴正方向开始填充。`Fill Range` 决定填充总量，值为 1 时将填充整个圆形。`Fill Range` 为正值时逆时针填充，为负值时顺时针填充。

## Label 属性

| 属性             | 功能说明                                                     |
| :--------------- | :----------------------------------------------------------- |
| String           | 文本内容字符串。                                             |
| Horizontal Align | 文本的水平对齐方式。可选值有 LEFT，CENTER 和 RIGHT。         |
| Vertical Align   | 文本的垂直对齐方式。可选值有 TOP，CENTER 和 BOTTOM。         |
| Font Size        | 文本字体大小。                                               |
| Line Height      | 文本的行高。                                                 |
| SpacingX         | 文本字符之间的间距。（使用 BMFont 位图字体时生效）           |
| Overflow         | 文本的排版方式，目前支持 CLAMP，SHRINK 和 RESIZE_HEIGHT。详情见下方的 [Label 排版](https://docs.cocos.com/creator/manual/zh/components/label.html#label-排版)。 |
| Enable Wrap Text | 是否开启文本换行。（在排版方式设为 CLAMP、SHRINK 时生效）    |
| Font             | 指定文本渲染需要的字体文件，如果使用系统字体，则此属性可以为空。 |
| Font Family      | 文字字体名字。(使用系统字体时生效)                           |
| Enable Bold      | 是否启用黑体。(使用系统字体或 TTF 字体时生效)                |
| Enable Italic    | 是否启用斜体。(使用系统字体或 TTF 字体时生效)                |
| Enable Underline | 是否启用下划线。(使用系统字体或 TTF 字体时生效)              |
| Underline Height | 下划线的高度。                                               |
| Cache Mode       | 文本缓存类型包括 **NONE**、**BITMAP**、**CHAR** 三种。仅对系统字体或 TTF 字体有效，BMFont 字体无需进行这个优化。详情见下方的 [文本缓存类型](https://docs.cocos.com/creator/manual/zh/components/label.html#文本缓存类型（cache-mode）)。 |
| Use System Font  | 是否使用系统字体。                                           |
| Src Blend Factor | 混合文本图片时，源图片的取值模式。可参考 [BlendFactor API](https://docs.cocos.com/creator/api/zh/enums/BlendFactor.html) |
| Dst Blend Factor | 混合显示两张图片时，目标图片的取值模式。可参考 [BlendFactor API](https://docs.cocos.com/creator/api/zh/enums/BlendFactor.html) |
| Materials        | 材质资源，详情请参考文档 [Material](https://docs.cocos.com/creator/manual/zh/render/material.html)。 |

## Label 排版

| 属性          | 功能说明                                                     |
| :------------ | :----------------------------------------------------------- |
| CLAMP         | 文字尺寸不会根据 Bounding Box 的大小进行缩放，Wrap Text 关闭的情况下，按照正常文字排列，超出 Bounding Box 的部分将不会显示。Wrap Text 开启的情况下，会试图将本行超出范围的文字换行到下一行。如果纵向空间也不够时，也会隐藏无法完整显示的文字。 |
| SHRINK        | 文字尺寸会根据 Bounding Box 大小进行自动缩放（不会自动放大，最大显示 Font Size 规定的尺寸）。 Wrap Text 开启时，当宽度不足时会优先将文字换到下一行，如果换行后还无法完整显示，则会将文字进行自动适配 Bounding Box 的大小。 Wrap Text 关闭时，则直接按照当前文字进行排版，如果超出边界则会进行自动缩放。 **注意**：这个模式在文本刷新的时候可能会占用较多 CPU 资源。 |
| RESIZE_HEIGHT | 文本的 Bounding Box 会根据文字排版进行适配，这个状态下用户无法手动修改文本的高度，文本的高度由内部算法自动计算出来。 |

## 文本缓存类型（Cache Mode）

| 属性   | 功能说明                                                     |
| :----- | :----------------------------------------------------------- |
| NONE   | 默认值，Label 中的整段文本将生成一张位图。                   |
| BITMAP | 选择后，Label 中的整段文本仍将生成一张位图，但是会尽量参与 [动态合图](https://docs.cocos.com/creator/manual/zh/advanced-topics/dynamic-atlas.html)。只要满足动态合图的要求，就会和动态合图中的其它 Sprite 或者 Label 合并 Draw Call。由于动态合图会占用更多内存，**该模式只能用于文本不常更新的 Label**。 **补充**：和 NONE 模式一样，BITMAP 模式会强制给每个 Label 组件生成一张位图，不论文本内容是否等同。如果场景中有大量相同文本的 Label，建议使用 CHAR 模式以复用内存空间。 |
| CHAR   | 原理类似 BMFont，Label 将以“字”为单位将文本缓存到全局共享的位图中，相同字体样式和字号的每个字符将在全局共享一份缓存。能支持文本的频繁修改，对性能和内存最友好。不过目前该模式还存在如下限制，我们将在后续的版本中进行优化： 1、**该模式只能用于字体样式和字号固定（通过记录字体的 fontSize、fontFamily、color、outline 为关键信息，以此进行字符的重复使用，其他有使用特殊自定义文本格式的需要注意），并且不会频繁出现巨量未使用过的字符的 Label**。这是为了节约缓存，因为全局共享的位图尺寸为 2048*2048，只有场景切换时才会清除，一旦位图被占满后新出现的字符将无法渲染。 2、不能参与动态合图（同样启用 CHAR 模式的多个 Label 在渲染顺序不被打断的情况下仍然能合并 Draw Call） |

> **注意**：
>
> 1. Cache Mode 对所有平台都有优化效果。
> 2. BITMAP 模式取代了原先的 Batch As Bitmap 选项，旧项目如启用了 Batch As Bitmap 将自动迁移至该选项。
> 3. 使用缓存模式时不能剔除 **项目 -> 项目设置 -> 模块设置** 面板中的 **RenderTexture** 模块。

## LabelOutline 组件参考

LabelOutline 组件将为所在节点上的 Label 组件添加描边效果，只能用于系统字体或者 TTF 字体。

![label-outline](https://docs.cocos.com/creator/manual/zh/components/label/label-outline.png)

点击 **属性检查器** 下面的 **添加组件** 按钮，然后从 **渲染组件** 中选择 **LabelOutline**，即可添加 LabelOutline 组件到节点上。

LabelOutline 组件的脚本接口请参考 [LabelOutline API](https://docs.cocos.com/creator/api/zh/classes/LabelOutline.html)。

## LabelOutline 属性

| 属性  | 功能说明   |
| ----- | ---------- |
| Color | 描边的颜色 |
| Width | 描边的宽度 |

## LabelShadow 组件参考

LabelShadow 组件可以为 Label 组件添加阴影效果，但只能用于系统字体或者 TTF 字体。

![label-shadow](https://docs.cocos.com/creator/manual/zh/components/label/label-shadow.png)

点击 **属性检查器** 下面的 **添加组件** 按钮，然后从 **渲染组件** 中选择 **LabelShadow**，即可添加 LabelShadow 组件到节点上。

LabelShadow 组件在 Label 组件的 Cache Mode 属性设置为 CHAR 时不生效，除了原生平台，但是原生平台也只有在使用 TTF 字体时是生效的。

描边脚本接口请参考 [LabelShadow API](https://docs.cocos.com/creator/api/zh/classes/LabelShadow.html)。

## LabelShadow 属性

| 属性   | 功能说明         |
| ------ | ---------------- |
| Color  | 阴影的颜色       |
| Offset | 字体与阴影的偏移 |
| Blur   | 阴影的模糊程度   |



## 系统文本的混合模式说明

对于 Label 组件，**Src Blend Factor** 常用的设置主要有两种，包括 **SRC_ALPHA** 和 **ONE**。引擎系统文本的实现是先将文本绘制到 Canvas，然后再生成图片给 Label 组件使用，这里涉及到一个文本透明度的处理问题。

- 当使用 **SRC_ALPHA** 模式时，可以通过顶点数据将透明度传递到 Shader 中，然后在 Shader 中进行像素透明度的计算，因此文本的透明度就不需要在绘制到 Canvas 时处理。在这种模式下，Label 节点透明度变化时，就不需要频繁的调用 updateRenderData 进行 Canvas 的重新绘制，可以减少 API 调用以及频繁重绘造成的性能消耗。
- 当使用 **ONE** 模式时，文本图片的透明度需要做预乘处理，所以在 Canvas 绘制时就需要进行透明度的处理。在这种模式下，Label 的节点透明度变化时就需要频繁的调用 updateRenderData，进行文本内容的重绘。

需要注意的是不同的混合模式，会影响与其他节点的动态合批，例如：

- 当 **Src Blend Factor** 选择 **ONE** 模式，**Cache Mode** 选择 **BITMAP** 缓存模式，则使用的是 **动态图集**，可能会导致动态合批失效。
- 若 **Cache Mode** 选择 **CHAR** 缓存模式，**Src Blend Factor** 会默认使用 **SRC_ALPHA** 模式，因为是全局共用同一张字符图集，无法进行不同的模式兼容。

对于 **原生平台**，在 **SRC_ALPHA** 模式下，为了消除文本的黑边问题，在文本图片数据返回时，需要做反预乘处理。
对于使用大量文本节点或者使用 **SHRINK** 模式的大段文本内容来说，做反预乘操作会有不少的性能消耗，开发者需要依据不同的使用场景以及文本内容进行合理的选择，以便在不同的平台能够减少重绘带来的性能消耗。具体的使用场景说明如下：

1. 如果 **Cache Mode** 选择 **CHAR** 缓存模式，只能使用 **SRC_ALPHA**。
2. 如果只是发布 **Web** 平台，推荐使用默认的 **SRC_ALPHA** 模式。因为 **ONE** 模式下，透明度变化会造成频繁的重绘。另外使用 **BITMAP** 缓存模式以及 **CHAR** 缓存模式也无法生效。
3. 如果需要发布 **Native** 平台，并且文本使用了 **SHRINK** 等会频繁重绘的排版模式，界面创建时会因为文本频繁的反预乘操作导致性能消耗比较明显，可以选择使用 **ONE** 模式避免反预乘带来的卡顿。

# Mask（遮罩）组件参考

Mask 用于规定子节点可渲染的范围，带有 Mask 组件的节点会使用该节点的约束框（也就是 **属性检查器** 中 Node 组件的 **Size** 规定的范围）创建一个渲染遮罩，该节点的所有子节点都会依据这个遮罩进行裁剪，遮罩范围外的将不会渲染。

点击 **属性检查器** 下面的 **添加组件** 按钮，然后从 **渲染组件** 中选择 **Mask**，即可添加 Mask 组件到节点上。注意该组件不能添加到有其他渲染组件（如 **Sprite**、**Label** 等）的节点上。

遮罩的脚本接口请参考 [Mask API](https://docs.cocos.com/creator/api/zh/classes/Mask.html)。

## Mask 属性

| 属性            | 功能说明                                                     |
| --------------- | ------------------------------------------------------------ |
| Type            | 遮罩类型。包括 **RECT**、**ELLIPSE**、**IMAGE_STENCIL** 三种类型，详情可查看 [Type API](https://docs.cocos.com/creator/api/zh/enums/Mask.Type.html) |
| Inverted        | 布尔值，反向遮罩                                             |
| Alpha Threshold | Alpha 阈值，该属性为浮点类型，仅在 Type 设为 **IMAGE_STENCIL** 时才生效。 只有当模板像素的 alpha 值大于该值时，才会绘制内容。 该属性的取值范围是 0 ~ 1，1 表示完全禁用。 |
| Sprite Frame    | 遮罩所需要的贴图，只在遮罩类型设为 **IMAGE_STENCIL** 时生效  |
| Segements       | 椭圆遮罩的曲线细分数，只在遮罩类型设为 **ELLIPSE** 时生效    |

**注意**：节点添加了 Mask 组件之后，所有在该节点下的子节点，在渲染的时候都会受 Mask 影响。

### `Mask.Type` 枚举

模块: [cc](https://docs.cocos.com/creator/api/zh/modules/cc.html)

遮罩组件类型

### 索引

- `RECT`
- `ELLIPSE`
- `IMAGE_STENCIL`

### Details

##### RECT

> 使用矩形作为遮罩

| meta   | description                                                  |
| ------ | ------------------------------------------------------------ |
| 类型   | [Number](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Number) |
| 定义于 | [cocos2d/core/components/CCMask.js:62](https://github.com/cocos-creator/engine/blob/76f37f407b386c997979b56dd0d3e99ac2c02cc4/cocos2d/core/components/CCMask.js#L62) |

##### ELLIPSE

> 使用椭圆作为遮罩

| meta   | description                                                  |
| ------ | ------------------------------------------------------------ |
| 类型   | [Number](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Number) |
| 定义于 | [cocos2d/core/components/CCMask.js:68](https://github.com/cocos-creator/engine/blob/76f37f407b386c997979b56dd0d3e99ac2c02cc4/cocos2d/core/components/CCMask.js#L68) |

##### IMAGE_STENCIL

> 使用图像模版作为遮罩

| meta   | description                                                  |
| ------ | ------------------------------------------------------------ |
| 类型   | [Number](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Number) |
| 定义于 | [cocos2d/core/components/CCMask.js:74](https://github.com/cocos-creator/engine/blob/76f37f407b386c997979b56dd0d3e99ac2c02cc4/cocos2d/core/components/CCMask.js#L74) |

# MotionStreak（拖尾）组件参考

MotionStreak（拖尾）是运动轨迹，用于在游戏对象的运动轨迹上实现拖尾渐隐效果。

![img](https://docs.cocos.com/creator/manual/zh/components/motion-streak/motionstreak.png)

点击 **属性检查器** 下方的 **添加组件** 按钮，然后从 **其他组件** 中选择 **MotionStreak**，即可添加 MotionStreak 组件到节点上。

![add motionStreak](https://docs.cocos.com/creator/manual/zh/components/motion-streak/add-motion-streak.png)

拖尾的脚本接口请参考 [MotionStreak API](https://docs.cocos.com/creator/api/zh/classes/MotionStreak.html)。

## MotionStreak 属性

| 属性     | 功能说明                                                     |
| -------- | ------------------------------------------------------------ |
| fadeTime | 拖尾的渐隐时间，以秒为单位。                                 |
| minSeg   | 拖尾之间的最小距离。                                         |
| stroke   | 拖尾的宽度。                                                 |
| texture  | 拖尾的贴图。                                                 |
| fastMode | 是否启用快速模式。当启用快速模式，新的点会被更快地添加，但精度较低。 |

# Graphics 组件参考

Graphics 组件提供了一系列绘画接口，这些接口参考了 canvas 的绘画接口来进行实现。

![img](https://docs.cocos.com/creator/manual/zh/render/graphics/graphics/graphics.png)

在 **层级管理器** 中选中一个节点，然后点击 **属性检查器** 下方的 **添加组件** 按钮，从 **渲染组件** 中选择 **Graphics**，即可添加 Graphics 组件到节点上。

## 绘图属性

| 属性                                                         | 功能说明                                 |
| ------------------------------------------------------------ | ---------------------------------------- |
| [lineCap](https://docs.cocos.com/creator/manual/zh/render/graphics/lineCap.html) | 设置或返回线条的结束端点样式             |
| [lineJoin](https://docs.cocos.com/creator/manual/zh/render/graphics/lineJoin.html) | 设置或返回两条线相交时，所创建的拐角类型 |
| [lineWidth](https://docs.cocos.com/creator/manual/zh/render/graphics/lineWidth.html) | 设置或返回当前的线条宽度                 |
| [miterLimit](https://docs.cocos.com/creator/manual/zh/render/graphics/miterLimit.html) | 设置或返回最大斜接长度                   |
| [strokeColor](https://docs.cocos.com/creator/manual/zh/render/graphics/strokeColor.html) | 设置或返回笔触的颜色                     |
| [fillColor](https://docs.cocos.com/creator/manual/zh/render/graphics/fillColor.html) | 设置或返回填充绘画的颜色                 |

## 绘图接口

### 路径

| 方法                                                         | 功能说明                                               |
| ------------------------------------------------------------ | ------------------------------------------------------ |
| [moveTo](https://docs.cocos.com/creator/manual/zh/render/graphics/moveTo.html) (x, y) | 把路径移动到画布中的指定点，不创建线条                 |
| [lineTo](https://docs.cocos.com/creator/manual/zh/render/graphics/lineTo.html) (x, y) | 添加一个新点，然后在画布中创建从该点到最后指定点的线条 |
| [bezierCurveTo](https://docs.cocos.com/creator/manual/zh/render/graphics/bezierCurveTo.html) (c1x, c1y, c2x, c2y, x, y) | 创建三次方贝塞尔曲线                                   |
| [quadraticCurveTo](https://docs.cocos.com/creator/manual/zh/render/graphics/quadraticCurveTo.html) (cx, cy, x, y) | 创建二次贝塞尔曲线                                     |
| [arc](https://docs.cocos.com/creator/manual/zh/render/graphics/arc.html) (cx, cy, r, a0, a1, counterclockwise) | 创建弧/曲线（用于创建圆形或部分圆）                    |
| [ellipse](https://docs.cocos.com/creator/manual/zh/render/graphics/ellipse.html) (cx, cy, rx, ry) | 创建椭圆                                               |
| [circle](https://docs.cocos.com/creator/manual/zh/render/graphics/circle.html) (cx, cy, r) | 创建圆形                                               |
| [rect](https://docs.cocos.com/creator/manual/zh/render/graphics/rect.html) (x, y, w, h) | 创建矩形                                               |
| [close](https://docs.cocos.com/creator/manual/zh/render/graphics/close.html) () | 创建从当前点回到起始点的路径                           |
| [stroke](https://docs.cocos.com/creator/manual/zh/render/graphics/stroke.html) () | 绘制已定义的路径                                       |
| [fill](https://docs.cocos.com/creator/manual/zh/render/graphics/fill.html) () | 填充当前绘图（路径）                                   |
| [clear](https://docs.cocos.com/creator/manual/zh/render/graphics/clear.html) () | 清除所有路径                                           |

