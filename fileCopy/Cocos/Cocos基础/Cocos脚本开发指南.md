# Cocos脚本开发指南

[toc]

## 使用cc.class声明类型

`cc.Class` 是一个很常用的 API，用于声明 Cocos Creator 中的类，为了方便区分，我们把使用 cc.Class 声明的类叫做 **CCClass**。

## 定义 CCClass

调用 `cc.Class`，传入一个原型对象，在原型对象中以键值对的形式设定所需的类型参数，就能创建出所需要的类。

```javascript
var Sprite = cc.Class({
    name: "sprite"
});
```

以上代码用 cc.Class 创建了一个类型，并且赋给了 `Sprite` 变量。同时还将类名设为 "sprite"，类名用于序列化，一般可以省略。

## 实例化

`Sprite` 变量保存的是一个 JavaScript 构造函数，可以直接 new 出一个对象：

```javascript
var obj = new Sprite();
```

## 判断类型

需要做类型判断时，可以用 JavaScript 原生的 `instanceof`：

```javascript
cc.log(obj instanceof Sprite);       // true
```

## 构造函数

使用 `ctor` 声明构造函数：

```javascript
var Sprite = cc.Class({
    ctor: function () {
        cc.log(this instanceof Sprite);    // true
    }
});
```

## 实例方法

```javascript
var Sprite = cc.Class({
    // 声明一个名叫 "print" 的实例方法
    print: function () { }
});
```

## 继承

使用 `extends` 实现继承：

```javascript
// 父类
var Shape = cc.Class();

// 子类
var Rect = cc.Class({
    extends: Shape
});
```

### 父构造函数

继承后，CCClass 会统一自动调用父构造函数，你不需要显式调用。

```javascript
var Shape = cc.Class({
    ctor: function () {
        cc.log("Shape");    // 实例化时，父构造函数会自动调用，
    }
});

var Rect = cc.Class({
    extends: Shape
});

var Square = cc.Class({
    extends: Rect,
    ctor: function () {
        cc.log("Square");   // 再调用子构造函数
    }
});

var square = new Square();
```

以上代码将依次输出 "Shape" 和 "Square"。

## 声明属性

通过在组件脚本中声明属性，我们可以将脚本组件中的字段可视化地展示在 **属性检查器** 中，从而方便地在场景中调整属性值。

要声明属性，仅需要在 cc.Class 定义的 `properties` 字段中，填写属性名字和属性参数即可，如：

```javascript
cc.Class({
    extends: cc.Component,
    properties: {
        userID: 20,
        userName: "Foobar"
    }
});
```

这时候，你可以在 **属性检查器** 中看到你刚刚定义的两个属性。

### 完整声明

有些情况下，我们需要为属性声明添加参数，这些参数控制了属性在 **属性检查器** 中的显示方式，以及属性在场景序列化过程中的行为。例如：

```javascript
properties: {
    score: {
        default: 0,
        displayName: "Score (player)",
        tooltip: "The score of player",
    }
}
```

以上代码为 `score` 属性设置了三个参数 `default`、`displayName` 和 `tooltip`。这几个参数分别指定了 `score` 的默认值为 0，在 **属性检查器** 里，其属性名将显示为：“Score (player)”，并且当鼠标移到参数上时，显示对应的 Tooltip。

下面是常用参数：

- **default**：设置属性的默认值，这个默认值仅在组件 **第一次** 添加到节点上时才会用到
- **type**：限定属性的数据类型，详见 [CCClass 进阶参考：type 参数](http://docs.cocos.com/creator/manual/zh/scripting/reference/class.html#type)
- **visible**：设为 false 则不在 **属性检查器** 面板中显示该属性
- **serializable**：设为 false 则不序列化（保存）该属性
- **displayName**：在 **属性检查器** 面板中显示成指定名字
- **tooltip**：在 **属性检查器** 面板中添加属性的 Tooltip

#### 数组声明

数组的 default 必须设置为 `[]`，如果要在 **属性检查器** 中编辑，还需要设置 type 为构造函数，枚举，或者 `cc.Integer`，`cc.Float`，`cc.Boolean` 和 `cc.String`。

```javascript
properties: {
    names: {
        default: [],
        type: [cc.String]   // 用 type 指定数组的每个元素都是字符串类型
    },

    enemies: {
        default: [],
        type: [cc.Node]     // type 同样写成数组，提高代码可读性
    },
}
```

### get/set 声明

在属性中设置了 `get` 或 `set` 以后，访问属性的时候，就能触发预定义的 `get` 或 `set` 方法。定义方法如下：

```javascript
properties: {
    width: {
        get: function () {
            return this._width;
        },
        set: function (value) {
            this._width = value;
        }
    }
}
```

> 如果你只定义 get 方法，那相当于属性只读。

## 访问节点和组件

你可以在 **属性检查器** 里修改节点和组件，也能在脚本中动态修改。动态修改的好处是能够在一段时间内连续地修改属性、过渡属性，实现渐变效果。脚本还能够响应玩家输入，能够修改、创建和销毁节点或组件，实现各种各样的游戏逻辑。要实现这些效果，你需要先在脚本中获得你要修改的节点或组件。

- 获得组件所在的节点
- 获得其它组件
- 使用 **属性检查器** 设置节点和组件
- 查找子节点
- 全局节点查找
- 访问已有变量里的值

## 获得组件所在的节点

获得组件所在的节点很简单，只要在组件方法里访问 `this.node` 变量：

```js
start: function () {
    var node = this.node;
    node.x = 100;
}
```

## 获得其它组件

你会经常需要获得同一个节点上的其它组件，这就要用到 `getComponent` 这个 API，它会帮你查找你要的组件。

```js
start: function () {
    var label = this.getComponent(cc.Label);
    var text = this.name + 'started';

    // Change the text in Label Component
    label.string = text;
}
```

你也可以为 `getComponent` 传入一个类名。对用户定义的组件而言，类名就是脚本的文件名，并且 **区分大小写**。例如 `SinRotate.js` 里声明的组件，类名就是 `SinRotate`。

```js
var rotate = this.getComponent("SinRotate");
```

在节点上也有一个 `getComponent` 方法，它们的作用是一样的：

```js
start: function () {
    cc.log(this.node.getComponent(cc.Label) === this.getComponent(cc.Label));  // true
}
```

如果在节点上找不到你要的组件，`getComponent` 将返回 null，如果你尝试访问 null 的值，将会在运行时抛出 TypeError 这个错误。因此如果你不确定组件是否存在，请记得判断一下：

```js
start: function () {
    var label = this.getComponent(cc.Label);
    if (label) {
        label.string = "Hello";
    }
    else {
        cc.error("Something wrong?");
    }
}
```

## 获得其它节点及其组件

仅仅能访问节点自己的组件通常是不够的，脚本通常还需要进行多个节点之间的交互。例如，一门自动瞄准玩家的大炮，就需要不断获取玩家的最新位置。Cocos Creator 提供了一些不同的方法来获得其它节点或组件。

### 利用属性检查器设置节点

最直接的方式就是在 **属性检查器** 中设置你需要的对象。以节点为例，这只需要在脚本中声明一个 type 为 `cc.Node` 的属性：

```js
// Cannon.js

cc.Class({
    extends: cc.Component,
    properties: {
        // 声明 player 属性
        player: {
            default: null,
            type: cc.Node
        }
    }
});
```

这段代码在 `properties` 里面声明了一个 `player` 属性，默认值为 null，并且指定它的对象类型为 `cc.Node`。这就相当于在其它语言里声明了 `public cc.Node player = null;`。

接着你就可以将 **层级管理器** 中的任意一个节点拖到这个 Player 控件。

这样一来它的 player 属性就会被设置成功，你可以直接在脚本里访问 player：

```js
// Cannon.js

cc.Class({
    extends: cc.Component,
    properties: {
        // 声明 player 属性
        player: {
            default: null,
            type: cc.Node
        }
    },

    start: function () {
        cc.log("The player is " + this.player.name);
    },

    // ...
});
```

### 利用属性检查器设置组件

在上面的例子中，如果你先通过 [模块化](http://docs.cocos.com/creator/manual/zh/scripting/modular-script.html#require) 方式获取到脚本（例如 `Player.js`），然后再将 `Cannon.js` 中属性的 type 声明为 Player 脚本组件：

```js
// Cannon.js

// 通过模块化方式获取脚本 “Player”
var Player = require("Player");

cc.Class({
    extends: cc.Component,
    properties: {
        // 声明 player 属性，直接声明为组件类型
        player: {
            default: null,
            type: Player
        }
    },

    start: function () {
        cc.log("The player is " + this.player.name);
    },

    // ...
});
```


那么脚本编译之后，这个组件在 **属性检查器** 中看起来是这样的：

![img](../image/Cocos脚本开发指南/player-in-properties-null.png)

然后将挂载了 `Player.js` 的 **Player Node** 拖拽到这个组件的 `player` 属性框中。

还可以将属性的默认值由 `null` 改为数组 `[]`，这样你就能在 **属性检查器** 中同时设置多个对象。
不过如果需要在运行时动态获取其它对象，还需要用到下面介绍的查找方法。

### 查找子节点

有时候，游戏场景中会有很多个相同类型的对象，像是炮塔、敌人和特效，它们通常都有一个全局的脚本来统一管理。如果用 **属性检查器** 来一个一个将它们关联到这个脚本上，那工作就会很繁琐。为了更好地统一管理这些对象，我们可以把它们放到一个统一的父物体下，然后通过父物体来获得所有的子物体：

```js
// CannonManager.js

cc.Class({
    extends: cc.Component,

    start: function () {
        var cannons = this.node.children;
        // ...
    }
});
```

你还可以使用 `getChildByName`：

```js
this.node.getChildByName("Cannon 01");
```

如果子节点的层次较深，你还可以使用 `cc.find`，`cc.find` 将根据传入的路径进行逐级查找：

```js
cc.find("Cannon 01/Barrel/SFX", this.node);
```

### 全局名字查找

当 `cc.find` 只传入第一个参数时，将从场景根节点开始逐级查找：

```js
this.backNode = cc.find("Canvas/Menu/Back");
```

## 访问已有变量里的值

如果你已经在一个地方保存了节点或组件的引用，你也可以直接访问它们，一般有两种方式：

### 通过全局变量访问

> 你应当很谨慎地使用全局变量，当你要用全局变量时，应该很清楚自己在做什么，我们并不推荐滥用全局变量，即使要用也最好保证全局变量只读。

让我们试着定义一个全局对象 `window.Global`，这个对象里面包含了 `backNode` 和 `backLabel` 两个属性。

```js
// Globals.js, this file can have any name

window.Global = {
    backNode: null,
    backLabel: null,
};
```

由于所有脚本都强制声明为 "use strict"，因此定义全局变量时的 `window.` 不可省略。
接着你可以在合适的地方直接访问并初始化 `Global`:

```js
// Back.js

cc.Class({
    extends: cc.Component,

    onLoad: function () {
        Global.backNode = this.node;
        Global.backLabel = this.getComponent(cc.Label);
    }
});
```

初始化后，你就能在任何地方访问到 `Global` 里的值：

```js
// AnyScript.js

cc.Class({
    extends: cc.Component,

    // start 会在 onLoad 之后执行，所以这时 Global 已经初始化过了
    start: function () {
        var text = 'Back';
        Global.backLabel.string = text;
    }
});
```

> 访问全局变量时，如果变量未定义将会抛出异常。
> 添加全局变量时，请小心不要和系统已有的全局变量重名。
> 你需要小心确保全局变量使用之前都已初始化和赋值。

如果你不想用全局变量，你可以使用 `require` 来实现脚本的跨文件操作，让我们看个示例：

```js
// Global.js, now the filename matters

module.exports = {
    backNode: null,
    backLabel: null,
};
```

每个脚本都能用 `require` + 文件名(不含路径) 来获取到对方 exports 的对象。

```js
// Back.js

// this feels more safe since you know where the object comes from
var Global = require("Global");

cc.Class({
    extends: cc.Component,

    onLoad: function () {
        Global.backNode = this.node;
        Global.backLabel = this.getComponent(cc.Label);
    }
});
// AnyScript.js

// this feels more safe since you know where the object comes from
var Global = require("Global");

cc.Class({
    extends: cc.Component,

    // start 会在 onLoad 之后执行，所以这时 Global 已经初始化过了
    start: function () {
        var text = "Back";
        Global.backLabel.string = text;
    }
});
```

# 常用节点和组件接口

## 节点状态和层级操作

假设我们在一个组件脚本中，通过 `this.node` 访问当前脚本所在节点。

### 激活/关闭节点

节点默认是激活的，我们可以在代码中设置它的激活状态，方法是设置节点的 `active` 属性：

```js
this.node.active = false;
```

设置 `active` 属性和在编辑器中切换节点的激活、关闭状态，效果是一样的。当一个节点是关闭状态时，它的所有组件都将被禁用。同时，它所有子节点，以及子节点上的组件也会跟着被禁用。要注意的是，子节点被禁用时，并不会改变它们的 `active` 属性，因此当父节点重新激活的时候它们就会回到原来的状态。

也就是说，`active` 表示的其实是该节点 **自身的** 激活状态，而这个节点 **当前** 是否可被激活则取决于它的父节点。并且如果它不在当前场景中，它也无法被激活。我们可以通过节点上的只读属性 `activeInHierarchy` 来判断它当前是否已经激活。

```js
this.node.active = true;
```

若节点原先就处于 **可被激活** 状态，修改 `active` 为 true 就会立即触发激活操作：

- 在场景中重新激活该节点和节点下所有 active 为 true 的子节点
- 该节点和所有子节点上的所有组件都会被启用，它们中的 `update` 方法之后每帧都会执行
- 这些组件上如果有 `onEnable` 方法，这些方法将被执行

```js
this.node.active = false;
```

若节点原先就处于 **可被激活** 状态，修改 `active` 为 true 就会立即触发激活操作：

- 在场景中重新激活该节点和节点下所有 active 为 true 的子节点
- 该节点和所有子节点上的所有组件都会被启用，它们中的 `update` 方法之后每帧都会执行
- 这些组件上如果有 `onEnable` 方法，这些方法将被执行

```js
this.node.active = false;
```

若该节点原先就已经被激活，修改 `active` 为 false 就会立即触发关闭操作：

- 在场景中隐藏该节点和节点下的所有子节点
- 该节点和所有子节点上的所有组件都将被禁用，也就是不会再执行这些组件中的 `update` 中的代码
- 这些组件上如果有 `onDisable` 方法，这些方法将被执行

### 更改节点的父节点

假设父节点为 `parentNode`，子节点为 `this.node`，您可以：

```js
this.node.parent = parentNode;
```

或

```js
this.node.removeFromParent(false);
parentNode.addChild(this.node);
```

这两种方法是等价的。

**注意**：

- `removeFromParent` 通常需要传入一个 `false`，否则默认会清空节点上绑定的事件和 action 等。
- 通过 [创建和销毁节点](http://docs.cocos.com/creator/manual/zh/scripting/create-destroy.html) 介绍的方法创建出新节点后，要为节点设置一个父节点才能正确完成节点的初始化。

### 索引节点的子节点

- `this.node.children` 将返回节点的所有子节点数组。
- `this.node.childrenCount` 将返回节点的子节点数量。

**注意**：以上两个 API 都只会返回节点的直接子节点，不会返回子节点的子节点。

## 更改节点的变换（位置、旋转、缩放、尺寸）

### 更改节点位置

分别对 x 轴和 y 轴坐标赋值：

```js
this.node.x = 100;
this.node.y = 50;
```

还可以使用 `setPosition` 方法进行赋值：

```js
this.node.setPosition(100, 50);
this.node.setPosition(cc.v2(100, 50));
```

或者通过设置 `position` 变量进行赋值：

```js
this.node.position = cc.v2(100, 50);
```

以上几种用法等价。

### 更改节点旋转

```js
this.node.rotation = 90;
```

或

```js
this.node.setRotation(90);
```

### 更改节点缩放

```js
this.node.scaleX = 2;
this.node.scaleY = 2;
```

或

```js
this.node.setScale(2);
this.node.setScale(2, 2);
```

以上两种方法等价。`setScale` 传入单个参数时，会同时修改 `scaleX` 和 `scaleY`。

### 更改节点尺寸

```js
this.node.setContentSize(100, 100);
this.node.setContentSize(cc.size(100, 100));
```

或

```js
this.node.width = 100;
this.node.height = 100;
```

以上两种方式等价。

### 更改节点锚点位置

```js
this.node.anchorX = 1;
this.node.anchorY = 0;
```

或

```js
this.node.setAnchorPoint(1, 0);
```

注意以上这些修改变换的方法会影响到节点上挂载的渲染组件，比如 Sprite 图片的尺寸、旋转等等。

## 颜色和不透明度

在使用 Sprite、Label 这些基本的渲染组件时，要注意修改颜色和不透明度的操作只能在节点的实例上进行，因为这些渲染组件本身并没有设置颜色和不透明度的接口。

假如我们有一个 Sprite 的实例为 `mySprite`，如果需要设置它的颜色：

```js
mySprite.node.color = cc.Color.RED;
```

设置不透明度：

```js
mySprite.node.opacity = 128;
```

## 常用组件接口

`cc.Component` 是所有组件的基类，任何组件都包括如下的常见接口（假设我们在该组件的脚本中，以 this 指代本组件）：

- `this.node`：该组件所属的节点实例
- `this.enabled`：是否每帧执行该组件的 `update` 方法，同时也用来控制渲染组件是否显示
- `update(dt)`：作为组件的成员方法，在组件的 `enabled` 属性为 `true` 时，其中的代码会每帧执行
- `onLoad()`：组件所在节点进行初始化时（节点添加到节点树时）执行
- `start()`：会在该组件第一次 `update` 之前执行，通常用于需要在所有组件的 `onLoad` 初始化完毕后执行的逻辑

## 生命周期

Cocos Creator 为组件脚本提供了生命周期的回调函数。用户只要定义特定的回调函数，Creator 就会在特定的时期自动执行相关脚本，用户不需要手工调用它们。

目前提供给用户的生命周期回调函数主要有：

- onLoad
- start
- update
- lateUpdate
- onDestroy
- onEnable
- onDisable

## onLoad

组件脚本的初始化阶段，我们提供了 `onLoad` 回调函数。`onLoad` 回调会在节点首次激活时触发，比如所在的场景被载入，或者所在节点被激活的情况下。在 `onLoad` 阶段，保证了你可以获取到场景中的其他节点，以及节点关联的资源数据。`onLoad` 总是会在任何 `start` 方法调用前执行，这能用于安排脚本的初始化顺序。通常我们会在 `onLoad` 阶段去做一些初始化相关的操作。例如：

```js
cc.Class({
  extends: cc.Component,

  properties: {
    bulletSprite: cc.SpriteFrame,
    gun: cc.Node,
  },

  onLoad: function () {
    this._bulletRect = this.bulletSprite.getRect();
    this.gun = cc.find('hand/weapon', this.node);
  },
});
```

## start

`start` 回调函数会在组件第一次激活前，也就是第一次执行 `update` 之前触发。`start` 通常用于初始化一些需要经常修改的数据，这些数据可能在 update 时会发生改变。

```js
cc.Class({
  extends: cc.Component,

  start: function () {
    this._timer = 0.0;
  },

  update: function (dt) {
    this._timer += dt;
    if ( this._timer >= 10.0 ) {
      console.log('I am done!');
      this.enabled = false;
    }
  },
});
```

## update

游戏开发的一个关键点是在每一帧渲染前更新物体的行为，状态和方位。这些更新操作通常都放在 `update` 回调中。

```js
cc.Class({
  extends: cc.Component,

  update: function (dt) {
    this.node.setPosition( 0.0, 40.0 * dt );
  }
});
```

## lateUpdate

`update` 会在所有动画更新前执行，但如果我们要在动效（如动画、粒子、物理等）更新之后才进行一些额外操作，或者希望在所有组件的 `update` 都执行完之后才进行其它操作，那就需要用到 `lateUpdate` 回调。

```js
cc.Class({
  extends: cc.Component,

  lateUpdate: function (dt) {
    this.node.rotation = 20;
  }
});
```

## onEnable

当组件的 `enabled` 属性从 `false` 变为 `true` 时，或者所在节点的 `active` 属性从 `false` 变为 `true` 时，会激活 `onEnable` 回调。倘若节点第一次被创建且 `enabled` 为 `true`，则会在 `onLoad` 之后，`start` 之前被调用。

## onDisable

当组件的 `enabled` 属性从 `true` 变为 `false` 时，或者所在节点的 `active` 属性从 `true` 变为 `false` 时，会激活 `onDisable` 回调。

## onDestroy

当组件或者所在节点调用了 `destroy()`，则会调用 `onDestroy` 回调，并在当帧结束时统一回收组件。当同时声明了 `onLoad` 和 `onDestroy` 时，它们将总是被成对调用。也就是说从组件初始化到销毁的过程中，它们要么就都会被调用，要么就都不会被调用。

## Tips

一个组件从初始化到激活，再到最终销毁的完整生命周期函数调用顺序为：`onLoad` -> `onEnable` -> `start` -> `update` -> `lateUpdate` -> `onDisable` -> `onDestroy`。

其中，`onLoad` 和 `start` 常常用于组件的初始化，只有在节点 `activeInHierarchy` 的情况下才能调用，并且最多只会被调用一次。除了上文提到的内容以及调用顺序的不同，它们还有以下区别：

|        | 节点激活时 | 组件 enabled 时才会调用？ |
| :----: | :--------: | :-----------------------: |
| onLoad |  立即调用  |            否             |
| start  |  延迟调用  |            是             |

