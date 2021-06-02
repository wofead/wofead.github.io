# 监听和发射事件

[toc]

## 监听事件

事件处理是在节点（`cc.Node`）中完成的。对于组件，可以通过访问节点 `this.node` 来注册和监听事件。监听事件可以通过 `this.node.on()` 函数来注册，方法如下：

```javascript
cc.Class({
  extends: cc.Component,

  properties: {
  },

  onLoad: function () {
    this.node.on('mousedown', function ( event ) {
      console.log('Hello!');
    });
  },
});
```

值得一提的是，事件监听函数 `on` 可以传第三个参数 target，用于绑定响应函数的调用者。以下两种调用方式，效果上是相同的：

```javascript
// 使用函数绑定
this.node.on('mousedown', function ( event ) {
  this.enabled = false;
}.bind(this));

// 使用第三个参数
this.node.on('mousedown', function (event) {
  this.enabled = false;
}, this);
```

除了使用 `on` 监听，我们还可以使用 `once` 方法。`once` 监听在监听函数响应后就会关闭监听事件。

## 关闭监听

当我们不再关心某个事件时，我们可以使用 `off` 方法关闭对应的监听事件。需要注意的是，`off` 方法的参数必须和 `on` 方法的参数一一对应，才能完成关闭。

我们推荐的书写方法如下：

```javascript
cc.Class({
  extends: cc.Component,

  _sayHello: function () {
    console.log('Hello World');
  },

  onEnable: function () {
    this.node.on('foobar', this._sayHello, this);
  },

  onDisable: function () {
    this.node.off('foobar', this._sayHello, this);
  },
});
```

## 发射事件

发射事件有两种方式：`emit` 和 `dispatchEvent`。两者的区别在于，后者可以做事件传递。我们先通过一个简单的例子来了解 `emit` 事件：

```js
cc.Class({
  extends: cc.Component,

  onLoad () {
    // args are optional param.
    this.node.on('say-hello', function (msg) {
      console.log(msg);
    });
  },

  start () {
    // At most 5 args could be emit.
    this.node.emit('say-hello', 'Hello, this is Cocos Creator');
  },
});
```

## 事件参数说明

在 2.0 之后，我们优化了事件的参数传递机制。 在发射事件时，我们可以在 `emit` 函数的第二个参数开始传递我们的事件参数。同时，在 `on` 注册的回调里，可以获取到对应的事件参数。

```js
cc.Class({
  extends: cc.Component,

  onLoad () {
    this.node.on('foo', function (arg1, arg2, arg3) {
      console.log(arg1, arg2, arg3);  // print 1, 2, 3
    });
  },

  start () {
    let arg1 = 1, arg2 = 2, arg3 = 3;
    // At most 5 args could be emit.
    this.node.emit('foo', arg1, arg2, arg3);
  },
});
```

需要说明的是，出于底层事件派发的性能考虑，这里最多只支持传递 5 个事件参数。所以在传参时需要注意控制参数的传递个数。

## 派送事件

上文提到了 `dispatchEvent` 方法，通过该方法发射的事件，会进入事件派送阶段。在 Cocos Creator 的事件派送系统中，我们采用冒泡派送的方式。冒泡派送会将事件从事件发起节点，不断地向上传递给他的父级节点，直到到达根节点或者在某个节点的响应函数中做了中断处理 `event.stopPropagation()`。![bubble-event](../image/Cocos监听和发射事件/bubble-event.png)

如上图所示，当我们从节点 c 发送事件 “foobar”，倘若节点 a，b 均做了 “foobar” 事件的监听，则 事件会经由 c 依次传递给 b，a 节点。如：

```javascript
// 节点 c 的组件脚本中
this.node.dispatchEvent( new cc.Event.EventCustom('foobar', true) );
```

如果我们希望在 b 节点截获事件后就不再将事件传递，我们可以通过调用 `event.stopPropagation()` 函数来完成。具体方法如下：

```javascript
// 节点 b 的组件脚本中
this.node.on('foobar', function (event) {
  event.stopPropagation();
});
```

请注意，在发送用户自定义事件的时候，请不要直接创建 `cc.Event` 对象，因为它是一个抽象类，请创建 `cc.Event.EventCustom` 对象来进行派发。

在事件监听回调中，开发者会接收到一个 `cc.Event` 类型的事件对象 `event`，`stopPropagation` 就是 `cc.Event` 的标准 API，其它重要的 API 包含：

| API 名                     |    类型    |                             意义                             |
| -------------------------- | :--------: | :----------------------------------------------------------: |
| `type`                     |  `String`  |                     事件的类型（事件名）                     |
| `target`                   | `cc.Node`  |                     接收到事件的原始对象                     |
| `currentTarget`            | `cc.Node`  | 接收到事件的当前对象，事件在冒泡阶段当前对象可能与原始对象不同 |
| `getType`                  | `Function` |                        获取事件的类型                        |
| `stopPropagation`          | `Function` | 停止冒泡阶段，事件将不会继续向父节点传递，当前节点的剩余监听器仍然会接收到事件 |
| `stopPropagationImmediate` | `Function` | 立即停止事件的传递，事件将不会传给父节点以及当前节点的剩余监听器 |
| `getCurrentTarget`         | `Function` |                 获取当前接收到事件的目标节点                 |
| `detail`                   | `Function` |       自定义事件的信息（属于 `cc.Event.EventCustom`）        |
| `setUserData`              | `Function` |     设置自定义事件的信息（属于 `cc.Event.EventCustom`）      |
| `getUserData`              | `Function` |     获取自定义事件的信息（属于 `cc.Event.EventCustom`）      |

完整的 API 列表可以参考 `cc.Event` 及其子类的 API 文档。

# 节点系统事件

`cc.Node` 有一套完整的 [事件监听和分发机制](http://docs.cocos.com/creator/manual/zh/scripting/events.html)。在这套机制之上，我们提供了一些基础的节点相关的系统事件，这篇文档将介绍这些事件的使用方式。

Cocos Creator 支持的系统事件包含鼠标、触摸、键盘、重力传感四种，其中本章节重点介绍与节点树相关联的鼠标和触摸事件，这些事件是被直接触发在相关节点上的，所以被称为节点系统事件。与之对应的，键盘和重力传感事件被称为全局系统事件.

系统事件遵守通用的注册方式，开发者既可以使用枚举类型也可以直接使用事件名来注册事件的监听器，事件名的定义遵循 DOM 事件标准。

```javascript
// 使用枚举类型来注册
node.on(cc.Node.EventType.MOUSE_DOWN, function (event) {
  console.log('Mouse down');
}, this);

// 使用事件名来注册
node.on('mousedown', function (event) {
  console.log('Mouse down');
}, this);
```

## 鼠标事件类型和事件对象

鼠标事件在桌面平台才会触发，系统提供的事件类型如下：

| 枚举对象定义                    | 对应的事件名 | 事件触发的时机                             |
| :------------------------------ | :----------- | :----------------------------------------- |
| `cc.Node.EventType.MOUSE_DOWN`  | `mousedown`  | 当鼠标在目标节点区域按下时触发一次         |
| `cc.Node.EventType.MOUSE_ENTER` | `mouseenter` | 当鼠标移入目标节点区域时，不论是否按下     |
| `cc.Node.EventType.MOUSE_MOVE`  | `mousemove`  | 当鼠标在目标节点区域中移动时，不论是否按下 |
| `cc.Node.EventType.MOUSE_LEAVE` | `mouseleave` | 当鼠标移出目标节点区域时，不论是否按下     |
| `cc.Node.EventType.MOUSE_UP`    | `mouseup`    | 当鼠标从按下状态松开时触发一次             |
| `cc.Node.EventType.MOUSE_WHEEL` | `mousewheel` | 当鼠标滚轮滚动时                           |

鼠标事件（`cc.Event.EventMouse`）的重要 API 如下（`cc.Event` 标准事件 API 除外）：

| 函数名                | 返回值类型 | 意义                                                         |
| :-------------------- | :--------- | :----------------------------------------------------------- |
| `getScrollY`          | `Number`   | 获取滚轮滚动的 Y 轴距离，只有滚动时才有效                    |
| `getLocation`         | `Object`   | 获取鼠标位置对象，对象包含 x 和 y 属性                       |
| `getLocationX`        | `Number`   | 获取鼠标的 X 轴位置                                          |
| `getLocationY`        | `Number`   | 获取鼠标的 Y 轴位置                                          |
| `getPreviousLocation` | `Object`   | 获取鼠标事件上次触发时的位置对象，对象包含 x 和 y 属性       |
| `getDelta`            | `Object`   | 获取鼠标距离上一次事件移动的距离对象，对象包含 x 和 y 属性   |
| `getButton`           | `Number`   | `cc.Event.EventMouse.BUTTON_LEFT` 或 `cc.Event.EventMouse.BUTTON_RIGHT` 或 `cc.Event.EventMouse.BUTTON_MIDDLE` |

触摸事件在移动平台和桌面平台都会触发，这样做的目的是为了更好得服务开发者在桌面平台调试，只需要监听触摸事件即可同时响应移动平台的触摸事件和桌面端的鼠标事件。系统提供的触摸事件类型如下：

| 枚举对象定义                     | 对应的事件名  | 事件触发的时机                   |
| :------------------------------- | :------------ | :------------------------------- |
| `cc.Node.EventType.TOUCH_START`  | `touchstart`  | 当手指触点落在目标节点区域内时   |
| `cc.Node.EventType.TOUCH_MOVE`   | `touchmove`   | 当手指在屏幕上移动时             |
| `cc.Node.EventType.TOUCH_END`    | `touchend`    | 当手指在目标节点区域内离开屏幕时 |
| `cc.Node.EventType.TOUCH_CANCEL` | `touchcancel` | 当手指在目标节点区域外离开屏幕时 |

触摸事件（`cc.Event.EventTouch`）的重要 API 如下（`cc.Event` 标准事件 API 除外）：

| API 名                | 类型       | 意义                                                       |
| :-------------------- | :--------- | :--------------------------------------------------------- |
| `touch`               | `cc.Touch` | 与当前事件关联的触点对象                                   |
| `getID`               | `Number`   | 获取触点的 ID，用于多点触摸的逻辑判断                      |
| `getLocation`         | `Object`   | 获取触点位置对象，对象包含 x 和 y 属性                     |
| `getLocationX`        | `Number`   | 获取触点的 X 轴位置                                        |
| `getLocationY`        | `Number`   | 获取触点的 Y 轴位置                                        |
| `getPreviousLocation` | `Object`   | 获取触点上一次触发事件时的位置对象，对象包含 x 和 y 属性   |
| `getStartLocation`    | `Object`   | 获取触点初始时的位置对象，对象包含 x 和 y 属性             |
| `getDelta`            | `Object`   | 获取触点距离上一次事件移动的距离对象，对象包含 x 和 y 属性 |

需要注意的是，触摸事件支持多点触摸，每个触点都会发送一次事件给事件监听器。

## 触摸事件的传递

### 触摸事件冒泡

触摸事件支持节点树的事件冒泡，以下图为例：![propagation](../image/Cocos监听和发射事件/propagation.png)

在图中的场景里，假设 A 节点拥有一个子节点 B，B 拥有一个子节点 C。开发者对 A、B、C 都监听了触摸事件（以下的举例都默认节点监听了触摸事件）。

当鼠标或手指在 C 节点区域内按下时，事件将首先在 C 节点触发，C 节点监听器接收到事件。接着 C 节点会将事件向其父节点传递这个事件，B 节点的监听器将会接收到事件。同理 B 节点会将事件传递给 A 父节点。这就是最基本的事件冒泡过程。需要强调的是，在触摸事件冒泡的过程中不会有触摸检测，这意味着即使触点不在 A B 节点区域内，A B 节点也会通过触摸事件冒泡的机制接收到这个事件。

触摸事件的冒泡过程与普通事件的冒泡过程并没有区别。所以，调用 `event.stopPropagation()` 可以主动停止冒泡过程。

### 同级节点间的触点归属问题

假设上图中 B、C 为同级节点，C 节点部分覆盖在 B 节点之上。这时候如果 C 节点接收到触摸事件后，就宣布了触点归属于 C 节点，这意味着同级节点的 B 就不会再接收到触摸事件了，即使触点同时也在 B 节点内。同级节点间，触点归属于处于顶层的节点。

此时如果 C 节点还存在父节点，则还可以通过事件冒泡的机制传递触摸事件给父节点。

### 将触摸或鼠标事件注册在捕获阶段

有时候我们需要父节点的触摸或鼠标事件先于他的任何子节点派发，比如 CCScrollView 组件就是这样设计的。这时候事件冒泡已经不能满足我们的需求了，需要将父节点的事件注册在捕获阶段。

要实现这个需求，可以在给 node 注册触摸或鼠标事件时，传入第四个参数 `true`，表示 `useCapture`。例如：

```js
this.node.on(cc.Node.EventType.TOUCH_START, this.onTouchStartCallback, this, true);
```

当节点触发 `touchstart` 事件时，会先将 `touchstart` 事件派发给所有注册在捕获阶段的父节点监听器，然后派发给节点自身的监听器，最后才到了事件冒泡阶段。

只有触摸或鼠标事件可以注册在捕获阶段，其他事件不能注册在捕获阶段。

### 触摸事件举例

以下图举例，总结下触摸事件的传递机制：

![example](../image/Cocos监听和发射事件/example.png)

图中有 A、B、C、D 四个节点，其中 A、B 为同级节点。具体层级关系如下：

![hierarchy](../image/Cocos监听和发射事件/hierarchy.png)

1. 若触点在 A、B 的重叠区域内，此时 B 接收不到触摸事件，事件的传递顺序是 **A -> C -> D**
2. 若触点在 B 节点内（可见的蓝色区域），则事件的传递顺序是 **B -> C -> D**
3. 若触点在 C 节点内，则事件的传递顺序是 **C -> D**
4. 若以第 2 种情况为前提，同时 C D 节点的触摸事件注册在捕获阶段，则事件的传递顺序是 **D -> C -> B**

## `cc.Node` 的其它事件

| 枚举对象定义 | 对应的事件名       | 事件触发的时机   |
| :----------- | :----------------- | :--------------- |
| 无           | `position-changed` | 当位置属性修改时 |
| 无           | `rotation-changed` | 当旋转属性修改时 |
| 无           | `scale-changed`    | 当缩放属性修改时 |
| 无           | `size-changed`     | 当宽高属性修改时 |
| 无           | `anchor-changed`   | 当锚点属性修改时 |

## 多点触摸事件

引擎在 v2.3 版本中新增了多点触摸事件的屏蔽开关，多点触摸事件默认为开启状态。对于有些类型的项目为了防止多点误触，需要屏蔽多点触摸事件，可以通过以下代码进行关闭：

```javascript
cc.macro.ENABLE_MULTI_TOUCH = false;
```

## 暂停或恢复节点系统事件

暂停节点系统事件

```js
// 暂停当前节点上注册的所有节点系统事件，节点系统事件包含触摸和鼠标事件。
// 如果传递参数 true，那么这个 API 将暂停本节点和它的所有子节点上的节点系统事件。
// example
this.node.pauseSystemEvents();
```

恢复节点系统事件

```js
// 恢复当前节点上注册的所有节点系统事件，节点系统事件包含触摸和鼠标事件。
// 如果传递参数 true，那么这个 API 将恢复本节点和它的所有子节点上的节点系统事件。
// example
this.node.resumeSystemEvents();
```

# 全局系统事件

全局系统事件是指与节点树不相关的各种全局事件，由 `cc.systemEvent` 来统一派发，目前支持以下几种事件：

- 键盘事件
- 设备重力传感事件

## 如何定义输入事件

键盘、设备重力传感器此类全局事件是通过函数 `cc.systemEvent.on(type, callback, target)` 注册的。

可选的 `type` 类型有:

1. cc.SystemEvent.EventType.KEY_DOWN (键盘按下)
2. cc.SystemEvent.EventType.KEY_UP (键盘释放)
3. cc.SystemEvent.EventType.DEVICEMOTION (设备重力传感)

### 键盘事件

- 事件监听器类型：`cc.SystemEvent.EventType.KEY_DOWN` 和 `cc.SystemEvent.EventType.KEY_UP`
- 事件触发后的回调函数：
  - 自定义回调函数：`callback(event);`
- 回调参数：
  - KeyCode: [API 传送门](http://docs.cocos.com/creator/api/zh/classes/Event.EventKeyboard.html)
  - Event：[API 传送门](http://docs.cocos.com/creator/api/zh/classes/Event.html)

```js
cc.Class({
    extends: cc.Component,
    onLoad: function () {
        // add key down and key up event
        cc.systemEvent.on(cc.SystemEvent.EventType.KEY_DOWN, this.onKeyDown, this);
        cc.systemEvent.on(cc.SystemEvent.EventType.KEY_UP, this.onKeyUp, this);
    },

    onDestroy () {
        cc.systemEvent.off(cc.SystemEvent.EventType.KEY_DOWN, this.onKeyDown, this);
        cc.systemEvent.off(cc.SystemEvent.EventType.KEY_UP, this.onKeyUp, this);
    },

    onKeyDown: function (event) {
        switch(event.keyCode) {
            case cc.macro.KEY.a:
                console.log('Press a key');
                break;
        }
    },

    onKeyUp: function (event) {
        switch(event.keyCode) {
            case cc.macro.KEY.a:
                console.log('release a key');
                break;
        }
    }
});
```

### 设备重力传感事件

- 事件监听器类型：`cc.SystemEvent.EventType.DEVICEMOTION`
- 事件触发后的回调函数：
  - 自定义回调函数：`callback(event);`
- 回调参数：
  - Event：[API 传送门](http://docs.cocos.com/creator/api/zh/classes/Event.html)

```js
cc.Class({
    extends: cc.Component,
    onLoad () {
        // open Accelerometer
        cc.systemEvent.setAccelerometerEnabled(true);
        cc.systemEvent.on(cc.SystemEvent.EventType.DEVICEMOTION, this.onDeviceMotionEvent, this);
    },

    onDestroy () {
        cc.systemEvent.off(cc.SystemEvent.EventType.DEVICEMOTION, this.onDeviceMotionEvent, this);
    },

    onDeviceMotionEvent (event) {
        cc.log(event.acc.x + "   " + event.acc.y);
    },
});
```

详情可参考 **官方范例**（[GitHub](https://github.com/cocos-creator/example-cases) | [Gitee](https://gitee.com/mirrors_cocos-creator/example-cases)）`assets/cases/03_gameplay/01_player_control` 目录下的完整范例（包含了键盘、重力感应、单点触摸、多点触摸等范例）。

