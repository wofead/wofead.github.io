



# 资源工作流程-贴图资源

[toc]

## 图像资源（Texture）

图像资源又经常被称作贴图、图片，是游戏中绝大部分图像渲染的数据源。图像资源一般由图像处理软件（比如 Photoshop，Windows 上自带的画图）制作而成并输出成 Cocos Creator 可以使用的文件格式，目前包括 **JPG** 和 **PNG** 两种。

## Texture 属性

| 属性              | 功能说明                                                     |
| ----------------- | ------------------------------------------------------------ |
| Type              | 包括 Raw 和 Sprite 两种模式。**Raw** 模式表示只会生成贴图资源，**Sprite** 模式表示还会生成 SpriteFrame 子资源。 |
| Premultiply Alpha | 是否开启 Alpha 预乘，勾选之后会将 RGB 通道预先乘以 Alpha 通道。 |
| Wrap Mode         | 寻址模式，包括 **Clamp（钳位）**、**Repeat（重复）** 两种寻址模式 |
| Filter Mode       | 过滤方式，包括 **Point（邻近点采样）**、**Bilinear（双线性过滤）**、**Trilinear（三线性过滤）** 三种过滤方式。 |
| genMipmaps        | 是否开启自动生成 mipmap                                      |
| packable          | 是否允许贴图参与合图                                         |

## Premultiply Alpha

Texture 的 Premultiply Alpha 属性勾选与否表示是否开启 Alpha 预乘，两种状态分别表示：

- **Premultiply Alpha（预乘 Alpha）**：表示 RGB 在存储的时候预先将 Alpha 通道与 RGB 相乘，比如透明度为 50% 的红色，RGB 为（255, 0, 0），预乘之后存储的颜色值为（127，0，0，0.5）。
- **Non-Premultiply Alpha（非预乘 Alpha）**：表示 RGB 不会预先与 Alpha 通道相乘，那么上面所述的透明度为 50% 的红色，存储的颜色值则为（255，0，0，0.5）。

那什么情况下需要使用 Premultiply Alpha？在图形渲染中透明图像通过 Alpha Blending 进行颜色混合，一般的颜色混合计算公式为：

**结果颜色 =（源颜色值 \* 源 alpha 值）+ 目标颜色值 \* (1 - 源 alpha 值)**

`result = source.RGB * source.A + dest.RGB * (1 - source.A);`

即颜色混合函数的设置为 `gl.blendFunc(gl.SRC_ALPHA, gl.ONE_MINUS_SRC_ALPHA)`。

当使用 Alpha 预乘之后，上述计算方式则简化为：

**结果颜色 = 源颜色值 + 目标颜色值 \* (1 - 源 alpha 值)**

`result = source.RGB + dest.RGB * (1 - source.A);`

对应的颜色混合函数设置为 `gl.blendFunc(gl.ONE, gl.ONE_MINUS_SRC_ALPHA)`。

但是使用 Alpha 预乘并不仅仅是为了简化上述计算提高效率，还是因为 Non-Premultiply Alpha 的纹理图像不能正确的进行线性插值计算。

假设有两个相邻顶点的颜色，一个是顶点颜色为透明度 100% 的红色（255，0，0，1.0），另一个是顶点颜色为透明度 10% 的绿色（0，255，0，0.1），那么当图像缩放时这两个顶点之间的颜色就是对它们进行线性插值的结果。如果是 Non-Premultiply Alpha，那么结果为：

`(255, 0, 0, 1.0) * 0.5 + (0, 255, 0, 0.1) * (1 - 0.5) = (127, 127, 0, 0.55)`

如果使用了 Premultiply Alpha，绿色存储的颜色值变为（0，25，0，0.1），再与红色进行线性插值的结果为：

`(255, 0, 0, 1.0）* 0.5 +（0, 25, 0, 0.1）*（1 - 0.5）=（127, 12, 0, 0.55）`

## 寻址模式

一般来说，纹理坐标 UV 的取值范围为 [0，1]，当传递的顶点数据中的纹理坐标取值超出 [0，1] 的范围时，就可以通过不同的寻址模式来控制超出范围的纹理坐标如何进行纹理映射，目前 Texture 提供两种寻址模式：

* **钳位寻址模式（Clamp）**
  将纹理坐标截取在 0 到 1 之间，只复制一遍 [0，1] 的纹理坐标。对于 [0，1] 之外的内容，将使用边缘的纹理坐标内容进行延伸。

* **重复寻址模式（Repeat）**

  对于超出 [0，1] 范围的纹理坐标，使用 [0，1] 的纹理坐标内容进行不断重复。

## 过滤方式

当 Texture 的原始大小与屏幕映射的纹理图像尺寸不一致时，通过不同的纹理过滤方式进行纹理单元到像素的映射会产生不同的效果。Texture 有三种过滤方式：

* **邻近点采样（Point）**

  使用中心位置距离采样点最近的纹理单元颜色值作为该采样点的颜色值，不考虑其他相邻像素的影响。优点是算法简单，计算量较小。缺点是当图像放大之后重新采样的颜色值不连续，会有明显的马赛克和锯齿。

* **双线性过滤（Bilinear）**
  使用距离采样点最近的 **2 x 2** 的纹理单元矩阵进行采样，取四个纹理单元颜色值的平均值作为采样点的颜色，像素之间的颜色值过渡更加平滑，但是计算量相比邻近点采样也稍大。

* **三线性过滤（Trilinear）**
  基于双线性过滤，对像素大小与纹理单元大小最接近的两层 Mipmap Level 分别进行双线性过滤，然后再对得到的结果进行线性插值计算采样点的颜色值。最终的采样结果相比邻近点采样和双线性过滤是最好的，但是计算量也最大。
  **注意**：当前引擎版本中三线性过滤与双线性过滤效果一致。

## genMipmaps

为了加快 3D 场景渲染速度和减少图像锯齿，贴图被处理成由一系列被预先计算和优化过的图片组成的序列，这样的贴图被称为 mipmap。mipmap 中每一个层级的小图都是原图的一个特定比例的缩小细节的复制品，当贴图被缩小或者只需要从远距离观看时，mipmap就会转换到适当的层级。

当贴图过滤方式设置为三线性过滤（trilinear filtering）时，会在两个相近的层级之间插值。因为渲染远距离物体时，mipmap 贴图比原图小，提高了显卡采样过程中的缓存命中率，所以渲染的速度得到了提升。同时因为 mipmap 的小图精度较低，从而减少了摩尔纹现象，可以减少画面上的锯齿。另外因为额外生成了一些小图，所以 mipmap 需要额外占用约三分之一的内存空间。

## packable

如果引擎开启了 [动态合图](https://docs.cocos.com/creator/manual/zh/advanced-topics/dynamic-atlas.html) 功能，动态合图会自动将合适的贴图在开始场景时动态合并到一张大图上来减少 drawcall。但是将贴图合并到大图中会修改原始贴图的 uv 坐标，如果在自定义 effect 中使用了贴图的 uv 坐标，这时 effect 中的 uv 计算将会出错，需要将贴图的 packable 属性设置为 **false** 来避免贴图被打包到动态合图中。

## 动态合图

降低 DrawCall 是提升游戏渲染效率一个非常直接有效的办法，而两个 DrawCall 是否可以合并为一个 DrawCall 的其中一个重要因素就是这两个 DrawCall 是否使用了同一张贴图。

Cocos Creator 提供了在项目构建时的静态合图方法 —— **自动合图**（Auto Atlas）。但是当项目日益壮大的时候贴图会变得非常多，很难将贴图打包到一张大贴图中，这时静态合图就比较难以满足降低 DrawCall 的需求。所以 Cocos Creator 在 v2.0 中加入了 **动态合图**（Dynamic Atlas）的功能，它能在项目运行时动态的将贴图合并到一张大贴图中。当渲染一张贴图的时候，动态合图系统会自动检测这张贴图是否已经被合并到了图集（图片集合）中，如果没有，并且此贴图又符合动态合图的条件，就会将此贴图合并到图集中。

动态合图是按照 **渲染顺序** 来选取要将哪些贴图合并到一张大图中的，这样就能确保相邻的 DrawCall 能合并为一个 DrawCall（又称“合批”）。

### 启用、禁用动态合图

Cocos Creator 在初始化过程中，会根据不同的平台设置不同的 [CLEANUP_IMAGE_CACHE](https://docs.cocos.com/creator/api/zh/classes/macro.html#cleanupimagecache) 参数，当禁用 `CLEANUP_IMAGE_CACHE` 时，动态合图就会默认开启。

启用动态合图会占用额外的内存，不同平台占用的内存大小不一样。目前在小游戏和原生平台上默认会禁用动态合图，但如果你的项目内存空间仍有富余的话建议开启。

**若希望强制开启动态合图**，请在代码中加入：

```typescript
cc.macro.CLEANUP_IMAGE_CACHE = false;
cc.dynamicAtlasManager.enabled = true;
```

> **注意**：这些代码请写在项目脚本中的最外层，不要写在 `onLoad` / `start` 等类函数中，才能确保在项目加载过程中即时生效。否则如果在部分贴图缓存已经释放的情况下才启用动态图集，可能会导致报错。

**若希望强制禁用动态合图**，可以直接剔除“Dynamic Atlas”模块以减小引擎包体，或者通过代码控制：

```typescript
cc.dynamicAtlasManager.enabled = false;
```

### 贴图限制

动态合图系统限制了能够进行合图的贴图大小，默认只有贴图宽高都小于 **512** 的贴图才可以进入到动态合图系统。用户可以根据需求修改这个限制：

```typescript
cc.dynamicAtlasManager.maxFrameSize = 512;
```

## 支持定制的渲染组件

目前动态合图系统只支持 Sprite、Label 等部分内建渲染组件的合图。如果希望加入其它类型渲染组件的支持，可以自己定制渲染组件。需要将相应的 SpriteFrame 添加到动态合图系统中：

```typescript
cc.dynamicAtlasManager.insertSpriteFrame(spriteFrame);
```

当 SpriteFrame 被添加到动态合图后，SpriteFrame 的贴图被替换为 `动态合图系统中的大图`，SpriteFrame 中的 uv 也会按照大图中的坐标进行重新计算。

**注意**：在场景加载前，动态合图系统会进行重置，SpriteFrame 贴图的引用和 uv 都会恢复到初始值。

### 调试

如果希望看到动态合图的效果，那么可以开启调试来看到最终生成的大图，这些大图会添加到一个 ScrollView 展示出来。

```typescript
// 开启调试
cc.dynamicAtlasManager.showDebug(true);
// 关闭调试
cc.dynamicAtlasManager.showDebug(false);
```



## Texture 和 SpriteFrame 资源类型

在 **资源管理器** 中，图像资源的左边会显示一个和文件夹类似的三角图标，点击就可以展开看到它的子资源（sub asset），每个图像资源导入后编辑器会自动在它下面创建同名的 SpriteFrame 资源。

SpriteFrame 是核心渲染组件 **Sprite** 所使用的资源，设置或替换 **Sprite** 组件中的 `spriteFrame` 属性，就可以切换显示的图像。**Sprite** 组件的设置方式请参考 [Sprite 组件参考](https://docs.cocos.com/creator/manual/zh/components/sprite.html)。

# Sprite 组件参考

Sprite（精灵）是 2D 游戏中最常见的显示图像的方式，在节点上添加 Sprite 组件，就可以在场景中显示项目资源中的图片。

### Sprite属性

| 属性             | 功能说明                                                     |
| ---------------- | ------------------------------------------------------------ |
| Atlas            | Sprite 显示图片资源所属的 [Atlas 图集资源](https://docs.cocos.com/creator/manual/zh/asset-workflow/atlas.html)。（Atlas 后面的 **选择** 按钮，该功能暂时不可用，我们会尽快优化） |
| Sprite Frame     | 渲染 Sprite 使用的 [SpriteFrame 图片资源](https://docs.cocos.com/creator/manual/zh/asset-workflow/sprite.html)。（Sprite Frame 后面的 **编辑** 按钮用于编辑图像资源的九宫格切分，详情请参考 [使用 Sprite 编辑器制作九宫格图像](https://docs.cocos.com/creator/manual/zh/ui/sliced-sprite.html)） |
| Type             | 渲染模式，包括普通（Simple）、九宫格（Sliced）、平铺（Tiled）、填充（Filled）和网格（Mesh）渲染五种模式 |
| Size Mode        | 指定 Sprite 的尺寸 `Trimmed` 表示会使用原始图片资源裁剪透明像素后的尺寸 `Raw` 表示会使用原始图片未经裁剪的尺寸 `Custom` 表示会使用自定义尺寸。当用户手动修改过 `Size` 属性后，`Size Mode` 会被自动设置为 `Custom`，除非再次指定为前两种尺寸。 |
| Trim             | 勾选后将在渲染时去除原始图像周围的透明像素区域，该项仅在 Type 设置为 Simple 时生效。详情请参考 [图像资源的自动剪裁](https://docs.cocos.com/creator/manual/zh/asset-workflow/trim.html)。导入图像资源后生成的 SpriteFrame 会进行自动剪裁，去除原始图片周围的透明像素区域。 |
| Src Blend Factor | 当前图像混合模式                                             |
| Dst Blend Factor | 背景图像混合模式，和上面的属性共同作用，可以将前景和背景 Sprite 用不同的方式混合渲染，效果预览可以参考 [glBlendFunc Tool](http://www.andersriggelsen.dk/glblendfunc.php)。 |

和图片裁剪相关的 **Sprite** 组件设置有以下两个：

- `Trim` 勾选后将在渲染 Sprite 图像时去除图像周围的透明像素，我们将看到刚好能把图像包裹住的约束框。取消勾选，Sprite 节点的约束框会包括透明像素的部分。只在 Type 设置为 Simple 时生效。

- `Size Mode` 用来将节点的尺寸设置为原图或原图裁剪透明像素后的大小，通常用于在序列帧动画中保证图像显示为正确的尺寸。有以下几种选择：

  * `TRIMMED` 选择这个选项，会将节点的尺寸（size）设置为原始图片裁剪掉透明像素后的大小。
  * `RAW` 选择这个，会将节点尺寸设置为原始图片包括透明像素的大小。
  * `CUSTOM` 自定义尺寸，开发者在使用 **矩形变换工具** 拖拽改变节点的尺寸，或通过修改 `Size` 属性，或在脚本中修改 `width` 或 `height` 后，都会自动将 `Size Mode` 设为 `CUSTOM`。表示开发者将自己决定节点的尺寸，而不需要考虑原始图片的大小。

  

  **若要动态更换 SpriteFrame 则需要先动态加载图片资源，然后再进行替换**。

### 渲染模式

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

在 `HORIZONTAL` 和 `VERTICAL` 这两种填充类型下，`Fill Start` 设置的数值将影响填充总量，如果 `Fill Start` 设为 0.5，那么即使 `Fill Range` 设为 1.0，实际填充的范围也仍然只有 Sprite 总大小的一半。

而 `RADIAL` 类型中 `Fill Start` 只决定开始填充的方向，`Fill Start` 为 0 时，从 x 轴正方向开始填充。`Fill Range` 决定填充总量，值为 1 时将填充整个圆形。`Fill Range` 为正值时逆时针填充，为负值时顺时针填充。

## 加载资源

### 动态加载resources

通常我们会把项目中需要动态加载的资源放在 `resources` 目录下，配合 `cc.resources.load` 等接口动态加载。你只要传入相对 resources 的路径即可，并且路径的结尾处 **不能** 包含文件扩展名。

```typescript
// 加载 Prefab
cc.resources.load("test assets/prefab", function (err, prefab) {
    var newNode = cc.instantiate(prefab);
    cc.director.getScene().addChild(newNode);
});

// 加载 AnimationClip
var self = this;
cc.resources.load("test assets/anim", function (err, clip) {
    self.node.getComponent(cc.Animation).addClip(clip, "anim");
});
```

所有需要通过脚本动态加载的资源，都必须放置在 `resources` 文件夹或它的子文件夹下。`resources` 文件夹需要在 **assets 根目录** 下手动创建。

> **resources** 文件夹中的资源，可以引用文件夹外部的其它资源，同样也可以被外部场景或资源所引用。项目构建时，除了在 **构建发布** 面板中勾选的场景外，**resources** 文件夹中的所有资源，包括它们关联依赖的 **resources** 文件夹外部的资源，都会被导出。
>
> 如果一份资源仅仅是被 **resources** 中的其它资源所依赖，而不需要直接被 `cc.resources.load` 调用，那么 **请不要** 放在 resources 文件夹中。否则会增大 `config.json` 的大小，并且项目中无用的资源，将无法在构建的过程中自动剔除。同时在构建过程中，JSON 的自动合并策略也将受到影响，无法尽可能合并零碎的 JSON。

Creator 相比之前的 Cocos2d-JS，资源动态加载的时候都是 **异步** 的，需要在回调函数中获得载入的资源。这么做是因为 Creator 除了场景关联的资源，没有另外的资源预加载列表，动态加载的资源是真正的动态加载。

### 加载SpriteFrame

图片设置为 Sprite 后，将会在 **资源管理器** 中生成一个对应的 SpriteFrame。但如果直接加载 `test assets/image`，得到的类型将会是 `cc.Texture2D`。你必须指定第二个参数为资源的类型，才能加载到图片生成的 `cc.SpriteFrame`：

```typescript
// 加载 SpriteFrame
var self = this;
cc.resources.load("test assets/image", cc.SpriteFrame, function (err, spriteFrame) {
    self.node.getComponent(cc.Sprite).spriteFrame = spriteFrame;
});
```

> 如果指定了类型参数，就会在路径下查找指定类型的资源。当你在同一个路径下同时包含了多个重名资源（例如同时包含 player.clip 和 player.psd），或者需要获取 “子资源”（例如获取 Texture2D 生成的 SpriteFrame），就需要声明类型。

### 加载图集中的 SpriteFrame

对从 TexturePacker 等第三方工具导入的图集而言，如果要加载其中的 SpriteFrame，则只能先加载图集，再获取其中的 SpriteFrame。这是一种特殊情况。

```typescript
// 加载 SpriteAtlas（图集），并且获取其中的一个 SpriteFrame
// 注意 atlas 资源文件（plist）通常会和一个同名的图片文件（png）放在一个目录下, 所以需要在第二个参数指定资源类型
cc.resources.load("test assets/sheep", cc.SpriteAtlas, function (err, atlas) {
    var frame = atlas.getSpriteFrame('sheep_down_0');
    sprite.spriteFrame = frame;
});
```

### 资源释放

`cc.resources.load` 加载进来的单个资源如果需要释放，可以调用 `cc.resources.release`，`release` 可以传入和 `cc.resources.load` 相同的路径和类型参数。

```typescript
cc.resources.release("test assets/image", cc.SpriteFrame);
cc.resources.release("test assets/anim");
```

此外，你也可以使用 `cc.assetManager.releaseAsset` 来释放特定的 Asset 实例。

```typescript
cc.assetManager.releaseAsset(spriteFrame);
```

### 资源的批量加载

`cc.resources.loadDir` 可以加载相同路径下的多个资源：

```typescript
// 加载 test assets 目录下所有资源
cc.resources.loadDir("test assets", function (err, assets) {
    // ...
});

// 加载 test assets 目录下所有 SpriteFrame，并且获取它们的路径
cc.resources.loadDir("test assets", cc.SpriteFrame, function (err, assets) {
    // ...
});
```

### 预加载资源

从 v2.4 开始，除了场景能够预加载之外，其他资源也可以预加载。预加载的加载参数与正常加载时一样，不过预加载只会去下载必要的资源，并不会进行资源的反序列化和初始化工作，所以性能消耗更小，适合游戏运行中使用。

`cc.resources` 提供了 `preload` 和 `preloadDir` 用于预加载资源。

```typescript
cc.resources.preload('test assets/image', cc.SpriteFrame);

// wait for while
cc.resources.load('test assets/image', cc.SpriteFrame, function (err, spriteFrame) {
    self.node.getComponent(cc.Sprite).spriteFrame = spriteFrame;
});
```

开发者可以使用预加载相关接口提前加载资源，不需要等到预加载结束即可使用正常加载接口进行加载，正常加载接口会直接复用预加载过程中已经下载好的内容，缩短加载时间。

为了尽可能缩短下载时间，很多游戏都会使用预加载。Asset Manager 中的大部分加载接口包括 `load`、`loadDir`、`loadScene` 都有其对应的预加载版本。加载接口与预加载接口所用的参数是完全一样的，两者的区别在于：

1. 预加载只会下载资源，不会对资源进行解析和初始化操作。
2. 预加载在加载过程中会受到更多限制，例如最大下载并发数会更小。
3. 预加载的下载优先级更低，当多个资源在等待下载时，预加载的资源会放在最后下载。
4. 因为预加载没有做任何解析操作，所以当所有的预加载完成时，不会返回任何可用资源。

**因为预加载没有去解析资源，所以需要在预加载完成后配合加载接口进行资源的解析和初始化，来完成资源加载。**

**注意**：加载不需要等到预加载完成后再调用，开发者可以在任何时候进行加载。正常加载接口会直接复用预加载过程中已经下载好的内容，缩短加载时间。

### 加载远程资源和设备资源

在目前的 Cocos Creator 中，我们支持加载远程贴图资源，这对于加载用户头像等需要向服务器请求的贴图很友好，需要注意的是，这需要开发者直接调用 `cc.assetManager.loadRemote` 方法。同时，如果开发者用其他方式下载了资源到本地设备存储中，也需要用同样的 API 来加载，上文中的 `cc.resources.load` 等 API 只适用于应用包内的资源和热更新的本地资源。下面是这个 API 的用法：

```typescript
// 远程 url 带图片后缀名
var remoteUrl = "http://unknown.org/someres.png";
cc.assetManager.loadRemote(remoteUrl, function (err, texture) {
    // Use texture to create sprite frame
});

// 远程 url 不带图片后缀名，此时必须指定远程图片文件的类型
remoteUrl = "http://unknown.org/emoji?id=124982374";
cc.assetManager.loadRemote(remoteUrl, {ext: '.png'}, function () {
    // Use texture to create sprite frame
});

// 用绝对路径加载设备存储内的资源，比如相册
var absolutePath = "/dara/data/some/path/to/image.png"
cc.assetManager.loadRemote(absolutePath, function () {
    // Use texture to create sprite frame
});

// 远程音频
remoteUrl = "http://unknown.org/sound.mp3";
cc.assetManager.loadRemote(remoteUrl, function (err, audioClip) {
    // play audio clip
});

// 远程文本
remoteUrl = "http://unknown.org/skill.txt";
cc.assetManager.loadRemote(remoteUrl, function (err, textAsset) {
    // use string to do something
});
```

目前的此类手动资源加载还有一些限制，对开发者影响比较大的是：

1. 这种加载方式只支持图片、声音、文本等原生资源类型，不支持 SpriteFrame、SpriteAtlas、Tilemap 等资源的直接加载和解析。（如需远程加载所有资源，可使用 [Asset Bundle](https://docs.cocos.com/creator/manual/zh/scripting/asset-bundle.html#加载-asset-bundle-中的资源))
2. Web 端的远程加载受到浏览器的 [CORS 跨域策略限制](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)，如果对方服务器禁止跨域访问，那么会加载失败，而且由于 WebGL 安全策略的限制，即便对方服务器允许 http 请求成功之后也无法渲染。

### 资源的依赖和释放

在加载完资源之后，所有的资源都会临时被缓存到 `cc.assetManager` 中，以避免重复加载资源时发送无意义的 http 请求，当然，缓存的内容都会占用内存，有些资源可能开发者不再需要了，想要释放它们，这里介绍一下在做资源释放时需要注意的事项。

**首先最为重要的一点就是：资源之间是互相依赖的。**

Prefab 资源中的 Node 包含 Sprite 组件，Sprite 组件依赖于 SpriteFrame，SpriteFrame 资源依赖于 Texture 资源，而 Prefab，SpriteFrame 和 Texture 资源都被 cc.assetManager 缓存起来了。这样做的好处是，有可能有另一个 SpriteAtlas 资源依赖于同样的一个 SpriteFrame 和 Texture，那么当你手动加载这个 SpriteAtlas 的时候，就不需要再重新请求贴图资源了，cc.assetManager 会自动使用缓存中的资源。

**接下来要介绍问题的另一个核心：JavaScript 中无法跟踪对象引用。**

在 JavaScript 这种脚本语言中，由于其弱类型特性，以及为了代码的便利，往往是不包含内存管理功能的，所有对象的内存都由垃圾回收机制来管理。这就导致 JS 层逻辑永远不知道一个对象会在什么时候被释放，这意味着引擎无法通过类似引用计数的机制来管理外部对象对资源的引用，也无法严谨地统计资源是否不再被需要了。

在 v2.4 之前，Creator 很长时间里选择让开发者控制所有资源的释放，包括资源本身和它的依赖项，你必须手动获取资源所有的依赖项并选择需要释放的依赖项，例如如下形式：

```typescript
// 直接释放某个贴图
cc.loader.release(texture);
// 释放一个 prefab 以及所有它依赖的资源
var deps = cc.loader.getDependsRecursively('prefabs/sample');
cc.loader.release(deps);
// 如果在这个 prefab 中有一些和场景其他部分共享的资源，你不希望它们被释放，可以将这个资源从依赖列表中删除
var deps = cc.loader.getDependsRecursively('prefabs/sample');
var index = deps.indexOf(texture2d._uuid);
if (index !== -1)
    deps.splice(index, 1);
cc.loader.release(deps);
```

这种方案给予了开发者最大的控制权力，对于小型项目来说工作良好，但随着 Creator 的发展，项目的规模不断提升，场景所引用的资源不断增加，而其他场景可能也复用了这些资源，这会造成释放资源的复杂度越来越高，开发者需要掌握所有资源的使用非常困难。为了解决这个痛点，Asset Manager 提供了一套基于引用计数的资源释放机制，让开发者可以简单高效地释放资源，不用担心项目规模的急剧膨胀。

这一套方案所做的工作是通过 AssetManager 加载资源时，对资源的依赖资源进行分析记录，并增加引用。而在通过 AssetManager 释放资源时，拿到记录的依赖资源，取消引用，并根据依赖资源的引用数，尝试自动去释放依赖资源。所以这个方案引擎只对资源的静态引用进行了分析，也就是说如果开发者在游戏运行过程中动态加载了资源并设置给场景或其他资源，则这些动态加载出来的资源引擎是没有记录的，这些资源需要开发者进行管理。每一个资源对象都提供了两个方法 `addRef`，`decRef`，你可以使用这两个接口来对动态资源的引用进行控制，比如说：

```typescript
cc.resources.load('image', cc.SpriteFrame, (err, spriteFrame) => {
    this.spriteFrame = spriteFrame;
    spriteFrame.addRef();
});
```

因为 texture 是动态加载进来的，而不是一开始就被组件所引用，所以这个 texture 是没有记录的，他的引用计数是 0，为了避免这个 texture 被其他地方误释放，开发者需要手动执行 `addRef` 操作为其增加一个引用。而在你不再需要使用这个资源时，你需要执行 `decRef` 为其减少一个引用：

```typescript
this.spriteFrame.decRef();
this.spriteFrame = null;
```

**最后一个值得关注的要点：JavaScript 的垃圾回收是延迟的。**

想象一种情况，当你释放了 cc.assetManager 对某个资源的引用之后，由于考虑不周的原因，游戏逻辑再次请求了这个资源。此时垃圾回收还没有开始（垃圾回收的时机不可控），当出现这个情况时，意味着这个资源还存在内存中，但是 cc.assetManager 已经访问不到了，所以会重新加载它。这造成这个资源在内存中有两份同样的拷贝，浪费了内存。如果只是一个资源还好，但是如果类似的资源很多，甚至不止一次被重复加载，这对于内存的压力是有可能很高的。如果观察到游戏使用的内存曲线有这样的异常，请仔细检查游戏逻辑，避免释放近期内将要复用的资源，如果没有的话，垃圾回收机制是会正常回收这些内存的。

## 图集资源

图集（Atlas）也称作 Sprite Sheet，是游戏开发中常见的一种美术资源。图集是通过专门的工具将多张图片合并成一张大图，并通过 **plist** 等格式的文件索引的资源。可供 Cocos Creator 使用的图集资源由 **plist** 和 **png** 文件组成。

### 为什么要使用图集资源

在游戏中使用多张图片合成的图集作为美术资源，有以下优势：

* 合成图集时会去除每张图片周围的空白区域，加上可以在整体上实施各种优化算法，合成图集后可以大大减少游戏包体和内存占用
* 多个 Sprite 如果渲染的是来自同一张图集的图片时，这些 Sprite 可以使用同一个渲染批次来处理，大大减少 CPU 的运算时间，提高运行效率。

### 制作图集资源

1. 要生成图集，首先您应该准备好一组原始图片
2. 接下来可以使用专门的软件生成图集，我们推荐的图集制作软件包括：TexturePacher和Zwoptex。使用这些软件生成图集时请选择 cocos2d-x 格式的 plist 文件。最终得到的图集文件是同名的 **plist** 和 **png**。

将上面所示的 **plist** 和 **png** 文件同时拖拽到 **资源管理器** 中，就可以生成可以在编辑器和脚本中使用的图集资源了。

### Atlas 和 SpriteFrame

导入图集资源后，我们可以看到类型为 `Atlas` 的图集资源，点击左边的三角图标展开，展开后可以看到图集资源里包含了很多类型为 `SpriteFrame` 的子资源，每个子资源都是可以单独使用和引用的图片。

### 碎图转图集

在项目原型阶段或生产初期，美术资源的内容和结构变化都会比较频繁，我们通常会直接使用碎图（也就是多个单独的图片）来搭建场景和制作 UI。在之后为了优化性能和节约包体，需要将碎图合并成图集。Creator 提供了自动图集功能，可以在发布项目时无缝地将生产阶段的碎图合并成图集，并且自动更新资源索引。

# 自动图集资源 (Auto Atlas)

**自动图集资源** 作为 Cocos Creator 自带的合图功能，可以将指定的一系列碎图打包成一张大图，具体作用和 Texture Packer 的功能很相近。

## 创建自动图集资源

在 **资源管理器** 中右键，可以在如下菜单中找到 **新建 -> 自动图集配置** 的子菜单，点击菜单将会新建一个类似 **AutoAtlas.pac** 的资源。

**自动图集资源** 将会以当前文件夹下的所有 **SpriteFrame** 作为碎图资源，以后会增加其他的选择碎图资源的方式。如果碎图资源 **SpriteFrame** 有配置过，在打包后重新生成的 **SpriteFrame** 中将会保留这些配置。

## 配置自动图集资源

在资源管理器中选中一个 **自动图集资源** 后，**属性检查器** 面板将会显示 **自动图集资源** 的所有可配置项。

| 属性               | 功能说明                                                     |
| ------------------ | ------------------------------------------------------------ |
| 最大宽度           | 单张图集最大宽度                                             |
| 最大高度           | 单张图集最大高度                                             |
| 间距               | 图集中碎图之间的间距                                         |
| 允许旋转           | 是否允许旋转碎图                                             |
| 输出大小为正方形   | 是否强制将图集长宽大小设置成正方形                           |
| 输出大小为二次幂   | 是否将图集长宽大小设置为二次方倍数                           |
| 算法               | 图集打包策略，可选的策略有 [BestShortSideFit、BestLongSideFit、BestAreaFit、BottomLeftRule、ContactPointRule] |
| 扩边               | 在碎图的边框外扩展出一像素外框，并复制相邻碎图像素到外框中。该功能也称作 “Extrude”。 |
| 不包含未被引用资源 | 在预览中，此选项不会生效，构建后此选项才会生效               |

配置完成后，如需预览，请点击 **预览** 按钮来预览打包的结果，结果将会展示在 **属性检查器** 下面的区域。

* **Packed Textures**：显示打包后的图集图片以及图片相关的信息，如果会生成的图片有多张，则会往下在 **属性检查器** 中列出来。
* **Unpacked Textures**：显示不能打包进图集的碎图资源，造成的原因有可能是这些碎图资源的大小比图集资源的大小还大导致的，这时候可能需要调整下图集的配置或者碎图的大小了。

### 生成图集

预览项目或者在 Cocos Creator 中使用碎图的时候都是直接使用的碎图资源，在 **构建项目** 这一步才会真正生成图集到项目中。

> **注意**：如果碎图开启了 Alpha 预乘，那么在生成图集时会失效。若需要使用预乘功能，可在图集上勾选 Premultiply Alpha。

为什么会有 SpriteFrame 这种资源？Texture 是保存在 GPU 缓冲中的一张纹理，是原始的图像资源。而 SpriteFrame 包含两部分内容：记录了 Texture 及其相关属性的 Texture2D 对象和纹理的矩形区域，对于相同的 Texture 可以进行不同的纹理矩形区域设置，然后根据 Sprite 的填充类型，如 SIMPLE、SLICED、TILED 等进行不同的顶点数据填充，从而满足 Texture 填充图像精灵的多样化需求。而 SpriteFrame 记录的纹理矩形区域数据又可以在资源的属性检查器中根据需求自由定义，这样的设置让资源的开发更为高效和便利。除了每个文件会产生一个 SpriteFrame 的图像资源（Texture）之外，我们还有包含多个 SpriteFrame 的图集资源（Atlas）类型。参考 [图集资源（Atlas）文档](https://docs.cocos.com/creator/manual/zh/asset-workflow/atlas.html) 来了解更多信息。

## 九宫格切分

![sliced](https://docs.cocos.com/creator/manual/zh/ui/sliced-sprite/editing.png)

https://docs.cocos.com/creator/manual/zh/asset-workflow/sprite.html

https://docs.cocos.com/creator/manual/zh/scripting/asset-bundle.html#%E5%8A%A0%E8%BD%BD-asset-bundle-%E4%B8%AD%E7%9A%84%E8%B5%84%E6%BA%90

