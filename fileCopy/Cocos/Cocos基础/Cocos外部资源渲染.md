# 外部资源渲染

[toc]

包括以下渲染组件：

- [ParticleSystem 组件参考](https://docs.cocos.com/creator/manual/zh/components/particle-system.html)
- [TiledMap 组件参考](https://docs.cocos.com/creator/manual/zh/components/tiledmap.html)
- [Spine 组件参考](https://docs.cocos.com/creator/manual/zh/components/spine.html)
- [DragonBones 组件参考](https://docs.cocos.com/creator/manual/zh/components/dragonbones.html)
- [VideoPlayer 组件参考](https://docs.cocos.com/creator/manual/zh/components/videoplayer.html)
- [WebView 组件参考](https://docs.cocos.com/creator/manual/zh/components/webview.html)

# ParticleSystem 组件参考

## 概述

该组件是用来读取 [粒子资源](https://docs.cocos.com/creator/manual/zh/asset-workflow/particle.html) 数据，并且对其进行一系列例如播放、暂停、销毁等操作。

## 创建方式

ParticleSystem 组件可通过编辑器和脚本两种方式创建，如下所示

### 1、通过编辑器创建

点击 **属性检查器** 下方的 **添加组件** 按钮，然后从 **渲染组件** 中选择 **ParticleSystem**，即可添加 ParticleSystem 组件到节点上。

### 2、通过脚本创建

```js
  // 创建一个节点
  var node = new cc.Node();
  // 将节点添加到场景中
  cc.director.getScene().addChild(node);
  // 添加粒子组件到 Node 上
  var particleSystem = node.addComponent(cc.ParticleSystem);
  // 接下去就可以对 particleSystem 这个对象进行一系列操作了
```

## ParticleSystem 属性

| 属性                  | 功能说明                                                     |
| --------------------- | ------------------------------------------------------------ |
| Preview               | 在编辑器模式下预览粒子，启用后选中粒子时，粒子将自动播放     |
| Play On Load          | 如果设置为 true 运行时会自动发射粒子                         |
| Auto Remove On Finish | 粒子播放完毕后自动销毁所在的节点                             |
| File                  | Plist 格式的粒子配置文件                                     |
| Custom                | 是否自定义粒子属性。开启该属性后可自定义以下部分粒子属性     |
| Sprite Frame          | 自定义的粒子贴图                                             |
| Duration              | 发射器生存时间，单位秒，-1 表示持续发射                      |
| Emission Rate         | 每秒发射的粒子数目                                           |
| Life                  | 粒子的运行时间及变化范围                                     |
| Total Particle        | 粒子最大数量                                                 |
| Start Color           | 粒子初始颜色                                                 |
| Start Color Var       | 粒子初始颜色变化范围                                         |
| End Color             | 粒子结束颜色                                                 |
| End Color Var         | 粒子结束颜色变化范围                                         |
| Angle                 | 粒子角度及变化范围                                           |
| Start Size            | 粒子的初始大小及变化范围                                     |
| End Size              | 粒子结束时的大小及变化范围                                   |
| Start Spin            | 粒子开始自旋角度及变化范围                                   |
| End Spin              | 粒子结束自旋角度及变化范围                                   |
| Source Pos            | 发射器位置                                                   |
| Pos Var               | 发射器位置的变化范围。（横向和纵向）                         |
| Position Type         | 粒子位置类型，包括 **FREE**、**RELATIVE**、**GROUPED** 三种。详情可参考 [PositionType API](https://docs.cocos.com/creator/api/zh/enums/ParticleSystem.PositionType.html) |
| Emitter Mode          | 发射器类型，包括 **GRAVITY**、**RADIUS** 两种。详情可参考 [EmitterMode API](https://docs.cocos.com/creator/api/zh/enums/ParticleSystem.EmitterMode.html) |
| Gravity               | 重力。仅在 Emitter Mode 设为 **GRAVITY** 时生效              |
| Speed                 | 速度及变化范围。仅在 Emitter Mode 设为 **GRAVITY** 时生效    |
| Tangential Accel      | 每个粒子的切向加速度及变化范围，即垂直于重力方向的加速度。仅在 Emitter Mode 设为 `GRAVITY` 时生效 |
| Radial Accel          | 粒子径向加速度及变化范围，即平行于重力方向的加速度。仅在 Emitter Mode 设为 **GRAVITY** 时生效 |
| Rotation Is Dir       | 每个粒子的旋转是否等于其方向。仅在 Emitter Mode 设为 **GRAVITY** 时生效 |
| Start Radius          | 初始半径及变化范围，表示粒子发射时相对发射器的距离。仅在 Emitter Mode 设为 **RADIUS** 时生效 |
| End Radius            | 结束半径。仅在 Emitter Mode 设为 **RADIUS** 时生效           |
| Rotate Per S          | 粒子每秒围绕起始点的旋转角度及变化范围。仅在 Emitter Mode 设为 **RADIUS** 时生效 |
| Src Blend Factor      | 混合显示两张图片时，原图片的取值模式。可参考 [BlendFactor API](https://docs.cocos.com/creator/api/zh/enums/BlendFactor.html) |
| Dst Blend Factor      | 混合显示两张图片时，目标图片的取值模式。可参考 [BlendFactor API](https://docs.cocos.com/creator/api/zh/enums/BlendFactor.html) |

# TiledMap 组件参考

TiledMap（地图）用于在游戏中显示 TMX 格式的地图。

点击 **属性检查器** 下方的 **添加组件** 按钮，然后从 **渲染组件** 中选择 **TiledMap**，即可添加 TiledMap 组件到节点上。

## TiledMap 属性

| 属性      | 功能说明                 |
| --------- | ------------------------ |
| Tmx Asset | 指定 .tmx 格式的地图资源 |

## 详细说明

- 添加 TiledMap 组件之后，从 **资源管理器** 中拖拽一个 **.tmx** 格式的地图资源到 Tmx Asset 属性上就可以在场景中看到地图的显示了。
- 在 TiledMap 组件中添加了 Tmx Asset 属性后，会在节点中自动添加与地图中的 Layer 对应的节点。这些节点都添加了 TiledLayer 组件。**请勿删除这些 Layer 节点中的 TiledLayer 组件**。

- TiledMap 组件不支持 `mapLoaded` 回调，在 `start` 函数中可正常使用 TiledMap 组件。

## TiledLayer 与节点遮挡

TiledLayer 组件会将添加到地图层的节点坐标转化为地图块行列坐标。当按行列顺序渲染地图层中的地图块时，如果该地图块的行列中存在节点，那么将会中断渲染地图块转而渲染节点。当地图块中的节点渲染完毕后，会继续渲染地图块。以此实现节点与地图层相互遮挡关系。

> **注意**：该遮挡关系只与节点的坐标有关，与节点的大小无关。

下面通过一个范例来介绍 TiledLayer 如何与节点相互遮挡。

1. 在场景中新建一个节点并添加 TiledMap 组件，设置好 TiledMap 组件属性后会自动生成带有 TiledLayer 组件的节点（即地图层）。

2. 创建 [预制资源](https://docs.cocos.com/creator/manual/zh/asset-workflow/prefab.html) 以便在场景中实例化出多个节点。

3. 在 **资源管理器** 中新建一个 JavaScript 脚本，编写组件脚本。脚本代码如下：

   ```js
    cc.Class({
        extends: cc.Component,
   
        properties: {
            // 用于实例化节点的预制体
            prefab:{
                type: cc.Prefab,
                default: null,
            },
   
            // TiledLayer 组件
            tiledLayer: {
                type: cc.TiledLayer,
                default: null,
            },
        },
   
        start () {
            // 开发者可根据需求设置节点位置
            let posArr = [cc.v2(-249, 96), cc.v2(-150, 76), cc.v2(-60, 54), cc.v2(-248, -144), cc.v2(-89, -34)];
            for (let i = 0; i < posArr.length; i++) {
                let shieldNode = cc.instantiate(this.prefab);
                // 可任意设置节点位置，这里仅作为示范
                shieldNode.x = posArr[i].x;
                shieldNode.y = posArr[i].y;
                // 调用 TiledLayer 组件的 addUserNode 方法，可将节点添加到对应的地图层中，并与地图层产生相互遮挡关系。
                this.tiledLayer.addUserNode(shieldNode); 
            }
        },
    });
   ```

4. 将脚本组件挂载到 Canvas 节点上，即将脚本拖拽到 Canvas 节点的 **属性检查器** 中。再将 **层级管理器** 中自动生成的带有 TiledLayer 组件的节点以及 **资源管理器** 中的预制资源拖拽至脚本组件对应的属性框中，然后保存场景。

5. 点击编辑器上方的预览按钮，即可看到节点与地图层相互遮挡的效果。关于代码可参考 example-case 中的 **ShieldNode** 范例（[GitHub](https://github.com/cocos-creator/example-cases/tree/v2.4.3/assets/cases/tiledmap) | [Gitee](https://gitee.com/mirrors_cocos-creator/example-cases/tree/v2.4.3/assets/cases/tiledmap)）。

   ![img](https://docs.cocos.com/creator/manual/zh/components/tiledmap/shieldNode.png)

若想移除地图层中的节点，调用 TiledLayer 的 `removeUserNode` 方法即可。

## TiledMap 关闭裁剪

```js
cc.macro.ENABLE_TILEDMAP_CULLING = false;
```

如果需要旋转地图或者把地图置于 3D 相机中，则需要关闭裁剪。另外，如果地图块不是非常多，如小于 5000 块，那么关闭裁剪还能减少 CPU 的运算负担，GPU 直接使用缓存进行渲染。



## TiledTile 组件参考

TiledTile 组件可以单独对某一个地图块进行操作。

![tiledtile-component](https://docs.cocos.com/creator/manual/zh/components/tiledtile/tiledtile-component.png)

## 创建方式

### 1、通过编辑器创建

在创建 [TiledMap 组件](https://docs.cocos.com/creator/manual/zh/components/tiledmap.html) 过程中 **自动生成** 的 Layer 节点下创建一个空节点。然后选中该空节点，点击 **属性检查器** 下方的 **添加组件 -> 渲染组件 -> TiledTile**，即可添加 TiledTile 组件到节点上。再通过设置 TiledTile 组件上的属性来操作地图块。

![img](https://docs.cocos.com/creator/manual/zh/components/tiledtile/add_tiledtile.png)

相关 TiledTile 脚本接口请参考 [TiledTile API](https://docs.cocos.com/creator/api/zh/classes/TiledTile.html)

### 2、通过代码创建

在代码中设置地图块有两种方式。当你在某个 Layer 节点中设置了 TiledTile 之后，该 Layer 节点原先所在位置的 TiledTile 将会被取代。

#### 通过对一个节点添加 TiledTile 组件创建

```js
// 创建一个新节点
var node = new cc.Node();
// 然后把该节点的父节点设置为任意的 layer 节点
node.parent = this.layer.node;  
// 最后添加 TiledTile 组件到该节点上，并返回 TiledTile 对象，就可以对 TiledTile 对象进行一系列操作
var tiledTile = node.addComponent(cc.TiledTile);
```

#### 通过 getTiledTileAt 获取 TiledTile

```js
// 获取 layer 上横向坐标为 0，纵向坐标为 0 的 TiledTile 对象，就可以对 TiledTile 对象进行一系列操作
var tiledTile = this.layer.getTiledTileAt(0, 0);
```

Layer 脚本接口相关请参考 [TiledLayer API](https://docs.cocos.com/creator/api/zh/classes/TiledLayer.html)

## TiledTile 属性

| 属性  | 功能说明                                                     |
| ----- | ------------------------------------------------------------ |
| X     | 指定 TiledTile 的横向坐标，以地图块为单位                    |
| Y     | 指定 TiledTile 的纵向坐标，以地图块为单位                    |
| Gid   | 指定 TiledTile 的 gid 值，来切换 TiledTile 的样式            |
| Layer | 获取 TiledTile 属于哪一个 TiledLayer (从 v2.0.1 开始移除该属性 ) |

TiledTile 可以控制指定的地图块，以及将节点的位移、旋转和缩放等应用到地图块。用户可以通过更改 TiledTile 的 gid 属性来更换地图块样式。

**注意**: 只能使用地图中现有地图块的 gid 来切换地图块的样式，无法通过自定义 Sprite Frame 来切换地图块的样式。

## 可作用到 TiledTile 上的节点属性

| 属性     | 功能说明                                    |
| -------- | ------------------------------------------- |
| Position | 可对指定的 TiledTile 进行 **平移** 操作     |
| Rotation | 可对指定的 TiledTile 进行 **旋转** 操作     |
| Scale    | 可对指定的 TiledTile 进行 **缩放** 操作     |
| Color    | 可对指定的 TiledTile 进行更改 **颜色** 操作 |
| Opacity  | 可对指定的 TiledTile 调整 **不透明度**      |
| Skew     | 可对指定的 TiledTile 调整 **倾斜角度**      |

# Spine 组件参考

Spine 组件支持 Spine 导出的数据格式，并对骨骼动画（Spine）资源进行渲染和播放。

选中节点，点击 **属性检查器** 下方的 **添加组件 -> 渲染组件 -> Spine Skeleton** 按钮，即可添加 Spine 组件到节点上。

## Spine 换装

下面通过一个范例介绍 Spine 如何换装。我们将会通过替换插槽的 attachment 对象，将绿色框中的手臂替换为红色框中的手臂。

![spine-cloth](https://docs.cocos.com/creator/manual/zh/components/spine/cloth0.png)

1. 首先在 **层级管理器** 中新建一个空节点，重命名为 goblingirl。在 **属性检查器** 中添加 Spine 组件，并将资源拖拽至 Spine 组件的 Skeleton Data 属性框中，然后将 Default Skin 属性设置成红色框中用于替换的皮肤。可更改 Spine 组件的 Animation 属性用于设置开发者想要播放的动画。

   ![spine-cloth](https://docs.cocos.com/creator/manual/zh/components/spine/cloth1.png)

2. 再次新建一个空节点并重命名为 goblin，添加 Spine 组件，并将资源拖拽至 Spine 组件的 Skeleton Data 属性框中。然后将 Default Skin 属性设置成绿色框中想要替换的皮肤，如下图所示：

   ![spine-cloth](https://docs.cocos.com/creator/manual/zh/components/spine/cloth2.png)

3. 在 **资源管理器** 中新建一个 JavaScript 脚本，编写组件脚本。脚本代码如下：

   ```js
    cc.Class({
        extends: cc.Component,
   
        properties: {
            goblin: {
                type: sp.Skeleton,
                default: null,
            },
            goblingirl: {
                type: sp.Skeleton,
                default: null,
            }
        },
   
        start () {
            let parts = ["left-arm", "left-hand", "left-shoulder"];
            for (let i = 0; i < parts.length; i++) {
                let goblin = this.goblin.findSlot(parts[i]);
                let goblingirl = this.goblingirl.findSlot(parts[i]);
                let attachment = goblingirl.getAttachment();
                goblin.setAttachment(attachment);
            }
        }
    });
   ```

4. 然后将脚本组件挂载到 Canvas 节点或者其他节点上，即将脚本拖拽到节点的 **属性检查器** 中。再将 **层级管理器** 中的 goblingirl 节点和 goblin 节点分别拖拽到脚本组件对应的属性框中，并保存场景。

   ![spine-cloth](https://docs.cocos.com/creator/manual/zh/components/spine/spine-js.png)

5. 点击编辑器上方的预览按钮，可以看到绿色框中的手臂已经被替换。

   ![spine-cloth](https://docs.cocos.com/creator/manual/zh/components/spine/cloth3.png)

## Spine 顶点效果

顶点效果只有当 Spine 处于 REALTIME 模式时有效，下面通过一个范例介绍 Spine 如何设置顶点效果。

1. 首先在 **层级管理器** 中新建一个空节点并重命名。然后在 **属性检查器** 中添加 Spine 组件，并将资源拖拽至 Spine 组件的 Skeleton Data 属性框中，设置好 Spine 组件属性。

2. 在 **资源管理器** 中新建一个 JavaScript 脚本，编写组件脚本。脚本代码如下：

   ```js
    cc.Class({
        extends: cc.Component,
   
        properties: {
            skeleton: {
                type: sp.Skeleton,
                default: null,
            }
        },
   
        start () {
            this._jitterEffect = new sp.VertexEffectDelegate();
            // 设置好抖动参数。
            this._jitterEffect.initJitter(20, 20);
            // 调用 Spine 组件的 setVertexEffectDelegate 方法设置效果。
            this.skeleton.setVertexEffectDelegate(this._jitterEffect);
        }
    });
   ```

3. 然后将脚本组件挂载到 Canvas 节点或者其他节点上，即将脚本拖拽到节点的 **属性检查器** 中。再将 **层级管理器** 中的节点拖拽到脚本组件对应的属性框中，并保存场景。

4. 点击编辑器上方的预览按钮，即可看到 Spine 动画的顶点抖动的效果。关于代码可参考 **SpineMesh** 范例（[GitHub](https://github.com/cocos-creator/example-cases/tree/v2.4.3/assets/cases/spine) | [Gitee](https://gitee.com/mirrors_cocos-creator/example-cases/tree/v2.4.3/assets/cases/spine)）。

## Spine 挂点

在使用骨骼动画时，经常需要在骨骼动画的某个部位上挂载节点，以实现节点与骨骼动画联动的效果。我们可以通过使用编辑器和脚本两种方式来实现 Spine 挂点，下面用一个范例来介绍 Spine 如何使用挂点将星星挂在龙的尾巴上，并随着龙的尾巴一起晃动。

![img](https://docs.cocos.com/creator/manual/zh/components/spine/attach0.png)

### 通过编辑器实现 Spine 挂点

1. 首先在 **层级管理器** 中新建一个空节点并重命名。选中该节点然后在 **属性检查器** 中添加 Spine 组件，并将资源拖拽至 Spine 组件的 Skeleton Data 属性框中，设置好 Spine 组件属性。然后点击 Spine 组件下方的 **生成挂点** 按钮。

   ![img](https://docs.cocos.com/creator/manual/zh/components/spine/attach1.png)

2. 点击 **生成挂点** 按钮后，**层级管理器** 中 Spine 组件所在节点的下方，会以节点树的形式生成所有骨骼。

   ![img](https://docs.cocos.com/creator/manual/zh/components/spine/attach2.png)

3. 在 **层级管理器** 中选中目标骨骼节点（龙的尾巴）作为父节点，创建一个 Sprite 节点为子节点。

   ![img](https://docs.cocos.com/creator/manual/zh/components/spine/attach3.png)

   即可看到在 **场景编辑器** 中龙的尾巴上已经挂了一个 Sprite。

   ![img](https://docs.cocos.com/creator/manual/zh/components/spine/attach4.png)

4. 最后将星星资源拖拽到 Sprite 组件的 Sprite Frame 属性上。保存场景，点击编辑器上方的预览按钮，即可看到星星挂在龙的尾巴上，并随着龙的尾巴一起晃动。具体可参考 example-case 中的 **SpineAttach** 范例（[GitHub](https://github.com/cocos-creator/example-cases/tree/v2.4.3/assets/cases/spine) | [Gitee](https://gitee.com/mirrors_cocos-creator/example-cases/tree/v2.4.3/assets/cases/spine)）。

> **注意**：Spine 挂点完成后，即可删除 **层级管理器** 中无用的骨骼节点，以减少运行时的计算开销。注意目标骨骼节点的父节点都不可删。

### 通过脚本实现 Spine 挂点

1. 跟通过编辑器实现的步骤类似，首先先创建一个挂有 Spine 组件的节点，并设置好 Spine 组件的属性。

2. 创建要挂载到骨骼动画上的星星预制资源，预制资源相关可参考 [Prefab](https://docs.cocos.com/creator/manual/zh/asset-workflow/prefab.html) 文档。

3. 在 **资源管理器** 中新建一个 JavaScript 脚本，编写组件脚本。脚本代码如下：

   ```js
    cc.Class({
        extends: cc.Component,
   
        properties: {
            skeleton: {
                type: sp.Skeleton,
                default: null,
            },
            // 将要添加到骨骼动画上的预制体
            targetPrefab: {
                type: cc.Prefab,
                default: null,
            },
            // 目标骨骼名称
            boneName: "",
        },
   
        onLoad () {
            this.generateSomeNodes();
        },
   
        generateAllNodes () {
            // 取得挂点工具
            let attachUtil = this.skeleton.attachUtil;
            attachUtil.generateAllAttachedNodes();
            // 因为同名骨骼可能不止一个，所以需要返回数组
            let boneNodes = attachUtil.getAttachedNodes(this.boneName);
            // 取第一个骨骼作为挂点
            let boneNode = boneNodes[0];
            boneNode.addChild(cc.instantiate(this.targetPrefab));
        },
   
        destroyAllNodes () {
            let attachUtil = this.skeleton.attachUtil;
            attachUtil.destroyAllAttachedNodes();
        },
   
        // 生成指定骨骼名称节点树的方法
        generateSomeNodes () {
            let attachUtil = this.skeleton.attachUtil;
            let boneNodes = attachUtil.generateAttachedNodes(this.boneName);
            let boneNode = boneNodes[0];
            boneNode.addChild(cc.instantiate(this.targetPrefab));
        },
   
        // 销毁指定骨骼名称节点的方法
        destroySomeNodes () {
            let attachUtil = this.skeleton.attachUtil;
            attachUtil.destroyAttachedNodes(this.boneName);
        }
    });
   ```

4. 将脚本挂载到 Canvas 节点或者其他节点上，即将脚本拖拽到节点的 **属性检查器** 中。然后再将对应的节点或者资源拖拽到脚本组件的属性框中，并保存场景。

   ![img](https://docs.cocos.com/creator/manual/zh/components/spine/attach_script.png)

   若不知道目标骨骼的名称，可点击 Spine 组件中的 **生成挂点** 按钮，然后在 **层级管理器** 中 Spine 节点下生成的骨骼节点树中查找。查找完成后再删除 Spine 节点下的骨骼节点树即可。

## Spine 碰撞检测

通过 Spine 挂点功能可以对骨骼动画的某个部位做碰撞检测，Spine 如何实现挂点请参考前面 Spine 挂点部分章节。下面通过一个范例来介绍 Spine 如何实现碰撞检测，通过判断人物脚与地面接触与否来实现当人物跑动时，动态地改变地面颜色。

![collider](https://docs.cocos.com/creator/manual/zh/components/spine/collider0.png)

1. 与通过编辑器实现 Spine 挂点的前两个步骤一样，创建好 Spine 节点后，点击 Spine 组件中的 **生成挂点** 按钮。

2. 然后在 **层级管理器** Spine 节点下的骨骼节点树中选中目标骨骼节点（人物的脚）作为父节点，再创建一个空节点（重命名为 FrontFootCollider）作为子节点。

   ![collider](https://docs.cocos.com/creator/manual/zh/components/spine/collider1.png)

3. 在 **层级管理器** 中选中 FrontFootCollider 节点，在 **属性检查器** 中点击 **添加组件 -> 碰撞组件 -> Polygon Collider**，然后设置好碰撞组件参数。该节点便会随着骨骼动画一起运动，从而碰撞组件的包围盒也会实时地与骨骼动画保持同步。

   ![collider](https://docs.cocos.com/creator/manual/zh/components/spine/collider2.png)

4. 在 **层级管理器** 中创建一个 Sprite 节点作为地面。选中该节点，然后在 **属性检查器** 中设置好位置大小等属性，并添加 **BoxCollider** 碰撞组件。

5. 在 **资源管理器** 中新建一个 JavaScript 脚本，然后将脚本挂载到地面节点上。脚本代码可参考：

   ```js
    cc.Class({
        extends: cc.Component,
   
        properties: {
   
        },
   
        start () {
            cc.director.getCollisionManager().enabled = true;
            cc.director.getCollisionManager().enabledDebugDraw = true;
            this.stayCount = 0;
        },
   
        onCollisionEnter (other, self) {
            this.stayCount++;
        },
   
        onCollisionExit (other, self) {
            this.stayCount--;
        },
   
        update () {
            if (this.stayCount > 0) {
                this.node.color = cc.color(0, 200, 200);
            } else {
                this.node.color = cc.color(255, 255, 255);
            }
        }
    });
   ```

6. 设置碰撞组件所在节点的分组，添加分组的方法请参考文档 [碰撞分组管理](https://docs.cocos.com/creator/manual/zh/physics/collision/collision-group.html)。

   ![collider](https://docs.cocos.com/creator/manual/zh/components/spine/collider_foot.png)

   ![collider](https://docs.cocos.com/creator/manual/zh/components/spine/collider_ground.png)

7. 点击编辑器上方的预览按钮，即可看到效果。具体可参考 example-case 中的 **SpineCollider** 范例（[GitHub](https://github.com/cocos-creator/example-cases/tree/v2.4.3/assets/cases/spine) | [Gitee](https://gitee.com/mirrors_cocos-creator/example-cases/tree/v2.4.3/assets/cases/spine)）。

> **注意**：由于挂点的实现机制导致基于挂点的碰撞检测，存在延迟一帧的问题。

# DragonBones 组件参考

DragonBones 组件可以对骨骼动画（DragonBones）资源进行渲染和播放。

在 **层级管理器** 中选中需要添加 DragonBones 组件的节点，然后点击 **属性检查器** 下方的 **添加组件 -> 渲染组件 -> DragonBones** 按钮，即可添加 DragonBones 组件到节点上。

- DragonBones 组件在脚本中的操作请参考 example-cases 范例中的 **DragonBones**（[GitHub](https://github.com/cocos-creator/example-cases/tree/v2.4.3/assets/cases/dragonbones) | [Gitee](https://gitee.com/mirrors_cocos-creator/example-cases/tree/v2.4.3/assets/cases/dragonbones)）。
- DragonBones 相关的脚本接口请参考 [DragonBones API](https://docs.cocos.com/creator/api/zh/modules/dragonBones.html)。

## DragonBones 属性

| 属性                 | 功能说明                                                     |
| :------------------- | :----------------------------------------------------------- |
| Dragon Asset         | 骨骼信息数据，包含了骨骼信息（绑定骨骼动作，slots，渲染顺序，attachments，皮肤等等）和动画，但不持有任何状态。 多个 ArmatureDisplay 可以共用相同的骨骼数据。 可拖拽 DragonBones 导出的骨骼资源到这里 |
| Dragon Atlas Asset   | 骨骼数据所需的 Atlas Texture 数据。可拖拽 DragonBones 导出的 Atlas 资源到这里 |
| Armature             | 当前使用的 Armature 名称                                     |
| Animation            | 当前播放的动画名称                                           |
| Animation Cache Mode | 渲染模式，默认 `REALTIME` 模式。（v2.0.9 中新增） 1. **REALTIME** 模式，实时运算，支持 DragonBones 所有的功能。 2. **SHARED_CACHE** 模式，将骨骼动画及贴图数据进行缓存并共享，相当于预烘焙骨骼动画。拥有较高性能，但不支持动作融合、动作叠加、骨骼嵌套，只支持动作开始和结束事件。至于内存方面，当创建 N(N>=3) 个相同骨骼、相同动作的动画时，会呈现内存优势。N 值越大，优势越明显。综上 `SHARED_CACHE` 模式适用于场景动画、特效、副本怪物、NPC 等，能极大提高帧率和降低内存。 3. **PRIVATE_CACHE** 模式，与 `SHARED_CACHE` 类似，但不共享动画及贴图数据，所以在内存方面没有优势，仅存在性能优势。当想利用缓存模式的高性能，但又存在换装的需求，因此不能共享贴图数据时，那么 `PRIVATE_CACHE` 就适合你。 |
| Time Scale           | 当前骨骼中所有动画的时间缩放率                               |
| Play Times           | 播放默认动画的循环次数。 **-1** 表示使用配置文件中的默认值； **0** 表示无限循环； **>0** 表示循环次数 |
| Premultiplied Alpha  | 图片是否启用贴图预乘，默认为 True。（v2.0.7 中新增） 当图片的透明区域出现色块时需要关闭该项。 当图片的半透明区域颜色变黑时需要启用该项 |
| Debug Bones          | 是否显示 bone 的 debug 信息                                  |
| Enable Batch         | 是否开启动画合批，默认关闭。（v2.0.9 中新增） 开启时，能减少 drawcall，适用于大量且简单动画同时播放的情况。 关闭时，drawcall 会上升，但能减少 cpu 的运算负担，适用于复杂的动画。 |

**注意**：当使用 DragonBones 组件时，**属性检查器** 中 Node 组件上的 **Anchor** 与 **Size** 属性是无效的。

## DragonBones 换装

下面通过一个范例介绍 DragonBones 如何换装。通过替换插槽的显示对象，将下图绿色框中的武器替换为红色框中的刀。此方法适用于 **v2.0.10** 或者 **v2.1.1** 及以上版本。

![dragonbones-cloth](https://docs.cocos.com/creator/manual/zh/components/dragonbones/cloth.png)

1. 首先在 **层级管理器** 中新建一个空节点，重命名为 knife。然后在 **属性检查器** 中添加 DragonBones 组件。并将红色框中的刀的资源拖拽至 DragonBones 组件的属性框中，如下图所示：

   ![dragonbones-cloth](https://docs.cocos.com/creator/manual/zh/components/dragonbones/cloth2.png)

2. 再次新建一个空节点并重命名为 robot，然后在 **属性检查器** 中添加 DragonBones 组件，并将机器人的资源拖拽至 DragonBones 组件的属性框中，如下图所示。可更改 DragonBones 组件的 Animation 属性用于设置开发者想要播放的动画。

   ![dragonbones-cloth](https://docs.cocos.com/creator/manual/zh/components/dragonbones/cloth3.png)

3. 在 **资源管理器** 中新建一个 JavaScript 脚本，编写组件脚本。脚本代码如下：

   ```js
    cc.Class({
        extends: cc.Component,
   
        properties: {
            robot: {
                type: dragonBones.ArmatureDisplay,
                default: null,
            },
            knife: {
                type: dragonBones.ArmatureDisplay,
                default: null,
            }
        },
   
        start () {
            let robotArmature = this.robot.armature();
            let robotSlot = robotArmature.getSlot("weapon_hand_r");
            let factory = dragonBones.CCFactory.getInstance();
            factory.replaceSlotDisplay(
                this.knife.getArmatureKey(), 
                "weapon", 
                "weapon_r", 
                "weapon_1004c_r", 
                robotSlot
            );
        },
    });
   ```

4. 然后将脚本组件挂载到 Canvas 节点上，即将脚本拖拽到 Canvas 节点的 **属性检查器** 中。再将 **层级管理器** 中的 robot 节点和 knife 节点分别拖拽到脚本组件对应的属性框中，并保存场景。

   ![dragonbones-cloth](https://docs.cocos.com/creator/manual/zh/components/dragonbones/dragonbone_jscomponent.png)

5. 点击编辑器上方的预览按钮，可以看到机器人右手的刀已经被替换。

   ![dragonbones-cloth](https://docs.cocos.com/creator/manual/zh/components/dragonbones/cloth4.png)

# VideoPlayer 组件参考

VideoPlayer 是一种视频播放组件，可通过该组件播放本地和远程视频。

**播放本地视频**：

![videoplayer](https://docs.cocos.com/creator/manual/zh/components/videoplayer/videoplayer.png)

**播放远程视频**：

![videoplayer-remote](https://docs.cocos.com/creator/manual/zh/components/videoplayer/videoplayer-remote.png)

点击 **属性检查器** 下面的 **添加组件** 按钮，然后从 **UI 组件** 中选择 **VideoPlayer**，即可添加 VideoPlayer 组件到节点上。

VideoPlayer 的脚本接口请参考 [VideoPlayer API](https://docs.cocos.com/creator/api/zh/classes/VideoPlayer.html)。

## VideoPlayer 属性

| 属性               | 功能说明                                                     |
| ------------------ | ------------------------------------------------------------ |
| Resource Type      | 视频来源的类型，目前支持本地（LOCAL）视频和远程（REMOTE）视频 URL |
| Clip               | 当 Resource Type 为 LOCAL 时显示的字段，拖拽本地视频的资源到此处来使用 |
| Remote URL         | 当 Resource Type 为 REMOTE 时显示的字段，填入远程视频的 URL  |
| Current Time       | 指定从哪个时间点开始播放视频                                 |
| Volume             | 视频的音量（0.0 ~ 1.0）                                      |
| Mute               | 是否静音视频。静音时设置音量为 0，取消静音时恢复原来的音量   |
| Keep Aspect Ratio  | 是否保持视频原来的宽高比                                     |
| Is Fullscreen      | 是否全屏播放视频                                             |
| Stay On Bottom     | 永远在游戏视图最底层（该属性仅在 Web 平台生效）              |
| Video Player Event | 视频播放回调函数，该回调函数会在特定情况被触发，比如播放中，暂时，停止和完成播放。详情见下方的 **VideoPlayer 事件** 章节或者 [VideoPlayerEvent API](https://docs.cocos.com/creator/api/zh/classes/VideoPlayer.html#videoplayerevent)。 |

**注意**：在 **Video Player Event** 属性的 **cc.Node** 中，应该填入的是一个挂载有用户脚本组件的节点，在用户脚本中便可以根据用户需要使用相关的 VideoPlayer 事件。

## VideoPlayer 事件

### VideoPlayerEvent 事件

| 属性            | 功能说明                                                     |
| --------------- | ------------------------------------------------------------ |
| target          | 带有脚本组件的节点。                                         |
| component       | 脚本组件名称。                                               |
| handler         | 指定一个回调函数，当视频开始播放后，暂停时或者结束时都会调用该函数，该函数会传一个事件类型参数进来。 |
| customEventData | 用户指定任意的字符串作为事件回调的最后一个参数传入。         |

详情可参考 API 文档 [Component.EventHandler 类型](https://docs.cocos.com/creator/api/zh/classes/Component.EventHandler.html)

### 事件回调参数

| 名称          | 功能说明                                                     |
| ------------- | ------------------------------------------------------------ |
| PLAYING       | 表示视频正在播放中。                                         |
| PAUSED        | 表示视频暂停播放。                                           |
| STOPPED       | 表示视频已经停止播放。                                       |
| COMPLETED     | 表示视频播放完成。                                           |
| META_LOADED   | 表示视频的元信息已加载完成，你可以调用 getDuration 来获取视频总时长。 |
| CLICKED       | 表示视频被用户点击了。（只支持 Web 平台）                    |
| READY_TO_PLAY | 表示视频准备好了，可以开始播放了。                           |

> **注意**：在 iOS 平台的全屏模式下，点击视频无法发送 CLICKED 事件。如果需要让 iOS 全屏播放并正确接受 CLICKED 事件，可以使用 Widget 组件把视频控件撑满。

详情可参考 [VideoPlayer 事件](https://docs.cocos.com/creator/api/zh/classes/VideoPlayer.html#事件) 或者参考引擎自带的 example-cases 范例中的 **09_videoplayer**（[GitHub](https://github.com/cocos-creator/example-cases/tree/v2.4.3/assets/cases/02_ui/09_videoplayer) | [Gitee](https://gitee.com/mirrors_cocos-creator/example-cases/tree/v2.4.3/assets/cases/02_ui/09_videoplayer)）。

## 详细说明

此控件支持的视频格式由所运行系统的视频播放器决定，为了让所有支持的平台都能正确播放视频，推荐使用 **mp4** 格式的视频。

### 通过脚本代码添加回调

#### 方法一

这种方法添加的事件回调和使用编辑器添加的事件回调是一样的。通过代码添加，首先你需要构造一个 `cc.Component.EventHandler` 对象，然后设置好对应的 `target`、`component`、`handler` 和 `customEventData` 参数。

```js
var videoPlayerEventHandler = new cc.Component.EventHandler();
videoPlayerEventHandler.target = this.node; //这个 node 节点是你的事件处理代码组件所属的节点
videoPlayerEventHandler.component = "cc.MyComponent"
videoPlayerEventHandler.handler = "callback";
videoPlayerEventHandler.customEventData = "foobar";

videoPlayer.videoPlayerEvent.push(videoPlayerEventHandler);

//here is your component file
cc.Class({
    name: 'cc.MyComponent'
    extends: cc.Component,

    properties: {
    },

    //注意参数的顺序和类型是固定的
    callback: function(videoplayer, eventType, customEventData) {
        //这里 videoplayer 是一个 VideoPlayer 组件对象实例
        // 这里的 eventType === cc.VideoPlayer.EventType enum 里面的值
        //这里的 customEventData 参数就等于你之前设置的 "foobar"
    }
});
```

#### 方法二

通过 `videoplayer.node.on('ready-to-play', ...)` 的方式来添加

```js
//假设我们在一个组件的 onLoad 方法里面添加事件处理回调，在 callback 函数中进行事件处理:
cc.Class({
    extends: cc.Component,

    properties: {
       videoplayer: cc.VideoPlayer
    },

    onLoad: function () {
       this.videoplayer.node.on('ready-to-play', this.callback, this);
    },

    callback: function (videoplayer) {
        // 这里的 videoplayer 表示的是 VideoPlayer 组件
        // 做任何你想要对 videoplayer 执行的操作
        // 需要注意的是，使用这种方式注册的事件，无法传递 customEventData
    },
});
```

同样的，用户也可以注册 `meta-loaded`、`clicked`、`playing` 等事件，这些事件的回调函数的参数与 `ready-to-play` 的参数一致。

**注意**：由于 VideoPlayer 是特殊的组件，所以它无法监听节点上的 **触摸** 和 **鼠标** 事件。

关于完整的 VideoPlayer 的事件列表，可以参考 [VideoPlayer API](https://docs.cocos.com/creator/api/zh/classes/VideoPlayer.html)。

## 如何实现 UI 在 VideoPlayer 上渲染

可通过以下三个步骤实现 UI 在 VideoPlayer 上显示：

1. 开启 `cc.macro.ENABLE_TRANSPARENT_CANVAS = true`（设置 Canvas 背景支持 alpha 通道）。

2. 在 **属性检查器** 中设置摄像机的 `backgroundColor` 属性透明度为 **0**。

3. 在 **属性检查器** 中勾选 VideoPlayer 组件上的 **stayOnBottom** 属性。

   ![img](https://docs.cocos.com/creator/manual/zh/components/videoplayer/stayonbutton.png)

**注意**：

- 该功能仅支持 **Web** 平台。
- 各个浏览器具体效果无法保证一致，跟浏览器是否支持与限制有关。
- 开启 **stayOnBottom** 后，将无法正常监听 `VideoPlayerEvent` 中的 `clicked` 事件。

# WebView 组件参考

WebView 是一种显示网页的组件，该组件让你可以在游戏里面集成一个小的浏览器。由于不同平台对于 WebView 组件的授权、API、控制方式都不同，还没有形成统一的标准，所以目前只支持 Web、iOS 和 Android 平台。

![webview](https://docs.cocos.com/creator/manual/zh/components/webview/webview.png)

点击 **属性检查器** 下方的 **添加组件** 按钮，然后从 **UI 组件** 中选择 **WebView**，即可添加 WebView 组件到节点上。

## WebView 属性

| 属性           | 功能说明                                                     |
| -------------- | ------------------------------------------------------------ |
| Url            | 指定一个 URL 地址，这个地址以 http 或者 https 开头，请填写一个有效的 URL 地址。 |
| Webview Events | WebView 的回调事件，当 webview 在加载网页过程中，加载网页结束后或者加载网页出错时会调用此函数。 |

> **注意**：在 **Webview Events** 属性的 **cc.Node** 中，应该填入的是一个挂载有用户脚本组件的节点，在用户脚本中便可以根据用户需要使用相关的 WebView 事件。

## WebView 事件

### WebViewEvents 事件

| 属性            | 功能说明                                                     |
| --------------- | ------------------------------------------------------------ |
| Target          | 带有脚本组件的节点。                                         |
| Component       | 脚本组件名称。                                               |
| Handler         | 指定一个回调函数，当网页加载过程中、加载完成后或者加载出错时会被调用，该函数会传一个事件类型参数进来。详情见下方的 **WebView 事件回调参数** 部分 |
| CustomEventData | 用户指定任意的字符串作为事件回调的最后一个参数传入。         |

### WebView 事件回调参数

| 名称    | 功能说明                 |
| ------- | ------------------------ |
| LOADING | 表示网页正在加载过程中。 |
| LOADED  | 表示网页加载已经完毕。   |
| ERROR   | 表示网页加载出错了。     |

## 详细说明

目前此组件只支持 Web（PC 和手机）、iOS 和 Android 平台（v2.0.0～2.0.6 版本不支持），Mac 和 Windows 平台暂时还不支持，如果在场景中使用此组件，那么在 PC 的模拟器里面预览的时候可能看不到效果。

**注意**:

- WebView 组件暂时不支持加载指定 HTML 文件或者执行 Javascript 脚本。
- 如果开发者在项目中未使用到 WebView 相关功能，请确保在 **项目 -> 项目设置 -> 模块设置** 中剔除 WebView 模块，以提高 iOS 的 App Store 机审成功率。如果开发者确实需要使用 WebView（或者添加的第三方 SDK 自带了 WebView），并因此 iOS 的 App Store 机审不通过，仍可尝试通过邮件进行申诉。

### 通过脚本代码添加回调

#### 方法一

这种方法添加的事件回调和使用编辑器添加的事件回调是一样的，通过代码添加，你需要首先构造一个 `cc.Component.EventHandler` 对象，然后设置好对应的 `target`、`component`、`handler` 和 `customEventData` 参数。

```js
//here is your component file
cc.Class({
    name: 'cc.MyComponent',
    extends: cc.Component,
    properties: {
       webview: cc.WebView,
    },

    onLoad: function() {
        var webviewEventHandler = new cc.Component.EventHandler();
        webviewEventHandler.target = this.node; //这个 node 节点是你的事件处理代码组件所属的节点
        webviewEventHandler.component = "cc.MyComponent";
        webviewEventHandler.handler = "callback";
        webviewEventHandler.customEventData = "foobar";

        this.webview.webviewEvents.push(webviewEventHandler);
    },

    //注意参数的顺序和类型是固定的
    callback: function(webview, eventType, customEventData) {
        //这里 webview 是一个 WebView 组件对象实例
        // 这里的 eventType === cc.WebView.EventType enum 里面的值
        //这里的 customEventData 参数就等于你之前设置的 "foobar"
    }
});
```

#### 方法二

通过 `webview.node.on('loaded', ...)` 的方式来添加

```js
//假设我们在一个组件的 onLoad 方法里面添加事件处理回调，在 callback 函数中进行事件处理:

cc.Class({
    extends: cc.Component,
    properties: {
        webview: cc.WebView,
    },

    onLoad: function () {
        this.webview.node.on('loaded', this.callback, this);
    },

    callback: function (event) {
        //这里的 event 是一个 EventCustom 对象，你可以通过 event.detail 获取 WebView 组件
        var webview = event.detail;
        //do whatever you want with webview
        //另外，注意这种方式注册的事件，也无法传递 customEventData
    }
});
```

同样的，你也可以注册 `loading`、`error` 事件，这些事件的回调函数的参数与 `loaded` 的参数一致。

## 如何与 WebView 内部页面进行交互

### 调用 WebView 内部页面

```js
cc.Class({
    extends: cc.Component,
    properties: {
        webview: cc.WebView,
    },

    onLoad: function () {
        // 这里的 Test 是你 webView 内部页面代码里定义的全局函数
        this.webview.evaluateJS('Test()');
    }
});
```

#### 注意: Web 平台上的跨域问题需要自行解决

### WebView 内部页面调用外部的代码

目前 Android 与 iOS 用的机制是，通过截获 URL 的跳转，判断 URL 前缀的关键字是否与之相同，如果相同则进行回调。

1. 通过 `setJavascriptInterfaceScheme` 设置 URL 前缀关键字
2. 通过 `setOnJSCallback` 设置回调函数，函数参数为 URL

```js
cc.Class({
    extends: cc.Component,
    properties: {
        webview: cc.WebView,
    },
    // onLoad 中设置会导致 API 绑定失效，所以请在 start 中设置 webview 回调。
    start: function () {
        // 这里是与内部页面约定的关键字，请不要使用大写字符，会导致 location 无法正确识别。
        var scheme = "testkey";

        function jsCallback (target, url) {
            // 这里的返回值是内部页面的 URL 数值，需要自行解析自己需要的数据。
            var str = url.replace(scheme + '://', ''); // str === 'a=1&b=2'
            // webview target
            console.log(target);
        }

        this.webview.setJavascriptInterfaceScheme(scheme);
        this.webview.setOnJSCallback(jsCallback);
    }
});
```

因此当你需要通过内部页面交互 WebView 时，应当设置内部页面 URL：`testkey://(后面你想要回调到 WebView 的数据)`。WebView 内部页面代码如下：

```html
<html>
<body>
    <dev>
        <input type="button" value="触发" onclick="onClick()"/>
    </dev>
</body>
<script>
    function onClick () {
        // 其中一个设置 URL 方案
        document.location = 'testkey://a=1&b=2';
    }
</script>
</html>
```

由于 Web 平台的限制，导致无法通过这种机制去实现，但是内部页面可以通过以下方式进行交互：

```html
<html>
<body>
    <dev>
        <input type="button" value="触发" onclick="onClick()"/>
    </dev>
</body>
<script>
    function onClick () {
        // 这里的 parent 其实就是外部的 window
        // 这样一来就可以访问到定义在 cc 的函数了
        parent.cc.TestCode();
        // 如果 TestCode 是定义在 window 上，则
        parent.TestCode();
    }
</script>
</html>
```