# 加载和切换场景

[toc]

在 Cocos Creator 中，我们使用场景文件名（不包含扩展名）来索引指代场景。并通过以下接口进行加载和切换操作：

```js
cc.director.loadScene("MyScene");
```

除此之外，从 v2.4 开始 Asset Bundle 还增加了一种新的加载方式：

```js
bundle.loadScene('MyScene', function (err, scene) {
    cc.director.runScene(scene);
});
```

Asset Bundle 提供的 `loadScene` 只会加载指定 bundle 中的场景，并不会自动运行场景，还需要使用 `cc.director.runScene` 来运行场景。
`loadScene` 还提供了更多参数来控制加载流程，开发者可以自行控制加载参数或者在加载完场景后做一些处理。

## 通过常驻节点进行场景资源管理和参数传递

引擎同时只会运行一个场景，当切换场景时，默认会将场景内所有节点和其他实例销毁。如果我们需要用一个组件控制所有场景的加载，或在场景之间传递参数数据，就需要将该组件所在节点标记为「常驻节点」，使它在场景切换时不被自动销毁，常驻内存。我们使用以下接口：

```js
cc.game.addPersistRootNode(myNode);
```

上面的接口会将 `myNode` 变为常驻节点，这样挂在上面的组件都可以在场景之间持续作用，我们可以用这样的方法来储存玩家信息，或下一个场景初始化时需要的各种数据。

如果要取消一个节点的常驻属性：

```js
cc.game.removePersistRootNode(myNode);
```

需要注意的是上面的 API 并不会立即销毁指定节点，只是将节点还原为可在场景切换时销毁的节点。

## 场景加载回调

加载场景时，可以附加一个参数用来指定场景加载后的回调函数：

```
cc.director.loadScene("MyScene", onSceneLaunched);
```

上一行里 `onSceneLaunched` 就是声明在本脚本中的一个回调函数，在场景加载后可以用来进一步的进行初始化或数据传递的操作。

由于回调函数只能写在本脚本中，所以场景加载回调通常用来配合常驻节点，在常驻节点上挂载的脚本中使用。

`cc.director.loadScene` 会在加载场景之后自动切换运行新场景，有些时候我们需要在后台静默加载新场景，并在加载完成后手动进行切换。那就可以预先使用 `cc.director.preloadScene` 接口对场景进行预加载：

```js
cc.director.preloadScene("table", function () {
    cc.log("Next scene preloaded");
});
```

之后在合适的时间调用 `loadScene`，就可以真正切换场景。

```js
cc.director.loadScene("table");
```

就算预加载没完成，依旧可以调用 `cc.director.loadScene`。

在 Creator 中，所有继承自 `cc.Asset` 的类型都统称资源，如 `cc.Texture2D`、`cc.SpriteFrame`、`cc.AnimationClip`、`cc.Prefab` 等。它们的加载是统一并且自动化的，相互依赖的资源能够被自动预加载。

> 例如，当引擎在加载场景时，会先自动加载场景关联到的资源，这些资源如果再关联其它资源，其它也会被先被加载，等加载全部完成后，场景加载才会结束。

脚本中可以这样定义一个 Asset 属性：

```javascript
// NewScript.js

cc.Class({
    extends: cc.Component,
    properties: {

        spriteFrame: {
            default: null,
            type: cc.SpriteFrame
        },

    }
});
```

只要在脚本中定义好类型，就能直接在 **属性检查器** 中很方便地设置资源。假设我们创建了这样一个脚本：

```javascript
// NewScript.js

cc.Class({
    extends: cc.Component,
    properties: {

        texture: {
            default: null,
            type: cc.Texture2D,
        },
        spriteFrame: {
            default: null,
            type: cc.SpriteFrame,
        },

    }
});
```

接下来我们从 **资源管理器** 里面分别将一张 Texture 和一个 SpriteFrame 拖到 **属性检查器** 的对应属性中

这样就能在脚本里直接拿到设置好的资源：

```javascript
onLoad: function () {
    var spriteFrame = this.spriteFrame;
    var texture = this.texture;

    spriteFrame.setTexture(texture);
}
```

在 **属性检查器** 中设置资源虽然很直观，但资源只能在编辑器里预先设好，没办法动态切换。

## 动态加载 resources

通常我们会把项目中需要动态加载的资源放在 `resources` 目录下，配合 `cc.resources.load` 等接口动态加载。你只要传入相对 resources 的路径即可，并且路径的结尾处 **不能** 包含文件扩展名。

```javascript
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

- 所有需要通过脚本动态加载的资源，都必须放置在 `resources` 文件夹或它的子文件夹下。`resources` 文件夹需要在 **assets 根目录** 下手动创建。

> **resources** 文件夹中的资源，可以引用文件夹外部的其它资源，同样也可以被外部场景或资源所引用。项目构建时，除了在 **构建发布** 面板中勾选的场景外，**resources** 文件夹中的所有资源，包括它们关联依赖的 **resources** 文件夹外部的资源，都会被导出。
>
> 如果一份资源仅仅是被 **resources** 中的其它资源所依赖，而不需要直接被 `cc.resources.load` 调用，那么 **请不要** 放在 resources 文件夹中。否则会增大 `config.json` 的大小，并且项目中无用的资源，将无法在构建的过程中自动剔除。同时在构建过程中，JSON 的自动合并策略也将受到影响，无法尽可能合并零碎的 JSON。

Creator 相比之前的 Cocos2d-JS，资源动态加载的时候都是 **异步** 的，需要在回调函数中获得载入的资源。这么做是因为 Creator 除了场景关联的资源，没有另外的资源预加载列表，动态加载的资源是真正的动态加载。

