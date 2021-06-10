# Cocos拓展编辑器

[toc]

Cocos Creator 提供了一系列方法来让用户定制和扩展编辑器的功能。这些扩展以包（package）的形式进行加载。用户通过将自己或第三方开发的扩展包安装到正确的路径进行扩展的加载，根据扩展功能的不同，有时可能会要求用户手动刷新窗口或者重新启动编辑器来完成扩展包的初始化。

Cocos Creator 的扩展包沿用了 Node.js 社区的包设计方式，通过 `package.json` 描述文件来定义扩展包的内容和注册信息。

# 你的第一个扩展包

本文会教你如何创建一个简单的 Cocos Creator 扩展包，并且向你介绍一些扩展包中的基本概念。通过学习，你将会创建一个扩展包，并在主菜单中建立一个菜单项，并通过该菜单项在主进程中执行一条扩展指令。

## 创建并安装扩展包

创建一个空文件夹命名为 “hello-world”，并在该文件夹中创建 `main.js` 和 `package.json` 两个文本文件。该扩展包的结构大致如下：

```
hello-world
  |--main.js
  |--package.json
```

将该文件夹放入到 `~/.CocosCreator/packages`（Windows 用户为 `C:\Users\${你的用户名}\.CocosCreator\packages`），或者放入到 `${你的项目路径}/packages` 文件夹下即可完成扩展包的安装。

## 定义你的包描述文件：package.json

每个包都需要一份 `package.json` 文件去描述他的用途，这样 Cocos Creator 编辑器才能知道这个包要扩展什么，从而正确加载。值得一提的是，虽然 `package.json` 在很多字段上的定义和 node.js 的 npm-package 相似，它们仍然是为不同的产品服务的，所以从 npm 社区中下载的包，并不能直接放入到 Cocos Creator 中变成插件，但是可以使用它。

我们在这里做一份简单的 `package.json`：

```json
{
  "name": "hello-world",
  "version": "0.0.1",
  "description": "一份简单的扩展包",
  "author": "Cocos Creator",
  "main": "main.js",
  "main-menu": {
    "Packages/Hello World": {
      "message": "hello-world:say-hello"
    }
  }
}
```

解释：

- `name` String - 定义了包的名字，包的名字是全局唯一的，关系到今后在官网服务器上登录时的名字。插件若要上传到 Cocos Store，对包名有一定的限制，只允许使用 **小写字母**、**数字**，**连字符（`-`）**、**下划线（`_`）** 和 **点（`.`）**，并以 **小写字母** 或 **数字** 开头。
- `version` String - 版本号，我们推荐使用 [semver](http://semver.org/) 格式管理你的包版本。
- `description` String（可选）- 一句话描述你的包是做什么的。
- `author` String（可选）- 扩展包的作者
- `main` String (可选) - 入口程序
- `main-menu` Object (可选) - 主菜单定义

## 入口程序

当你定义好你的描述文件以后，接下来就要书写你的入口程序 `main.js` 了。定义如下：

```javascript
'use strict';

module.exports = {
  load () {
    // 当 package 被正确加载的时候执行
  },

  unload () {
    // 当 package 被正确卸载的时候执行
  },

  messages: {
    'say-hello' () {
      Editor.log('Hello World!');
    }
  },
};
```

这份入口程序会在 Cocos Creator 的主进程中被加载，在加载成功后，他会调用入口程序中的 `load` 函数。并且会将定义在 `messages` 字段中的函数注册成 IPC 消息。更多关于入口函数中的消息注册，以及 IPC 消息的内容，我们会在 [IPC简介](https://docs.cocos.com/creator/manual/zh/extension/introduction-to-ipc.html) 中讲解。

这里我们只要明白，入口函数中的 `messages` 字段中的函数，将会在主进程的 IPC 监听模块中，注册一份消息，其格式为 `${扩展包名}:${函数名}`，并将对应的函数作为 IPC 响应的函数。

## 运行扩展包程序

现在你可以打开你的 Cocos Creator，你将会发现你的主菜单中多出了一份 `Packages` 的菜单，点击 `Packages` 菜单中的 `Hello World` 将会发送一个消息 “hello-world:say-hello” 给我们的扩展包的 `main.js`，它会在 Creator 的控制台中打印出 “Hello World” 的日志信息。

恭喜你完成了第一个简单的编辑器扩展工具。

# 安装与分享

当你启动 Cocos Creator 并且打开了一个指定项目后，Cocos Creator 会开始搜索并加载扩展包。Cocos Creator 有两个扩展包搜索路径，“全局扩展包路径”和“项目扩展包路径”。

## 从扩展商店获取扩展插件

点击主菜单的 `扩展/扩展商店`，即可打开扩展商店。

在扩展商店里可以搜索或浏览不同类别的插件，Cocos Creator 的编辑器扩展插件会归类到 `Creator 扩展` 里。

## 全局扩展包路径

当你打算将你的扩展应用到所有的 Cocos Creator 项目中，你可以选择将扩展包安装到全局扩展包路径中。根据你的目标平台，全局扩展包路径将会放在：

- **Windows** `%USERPROFILE%\.CocosCreator\packages`
- **Mac** `$HOME/.CocosCreator/packages`

## 项目扩展包路径

有时候我们只希望某些扩展功能被应用于指定项目中，这个时候我们可以将扩展包安装到项目的扩展包存放路径上。项目的扩展包路径为 `$你的项目地址/packages`。

## 卸载扩展包

直接从上述文件夹中删除扩展包即可。

## 扩展包包名和目录名

扩展包的包名 ( `package.json` 声明中的 `name` 字段的内容) 最好和扩展包所在路径的路径名一致，比如前面第一个扩展包 `hello-world`，如果放在项目扩展包路径中，其路径结构应该是：

```
MyProject
 |--assets
 |--packages
     |--hello-world
        |--package.json
        |--main.js
```

这样我们在编写扩展包逻辑时可以更方便的获取到扩展包所在路径下的文件 url，减少出错的可能。

## 开发扩展包时的实时改动监控

在开发扩展包的过程中，编辑器进程会对扩展包里的脚本内容进行监控，当有脚本内容发生变化时，会自动对扩展包进行重新载入。文件监控的规则可以在 `package.json` 中定制，详情请阅读 [package.json 字段：reload](https://docs.cocos.com/creator/manual/zh/extension/reference/package-json-reference.html#reload-object-)。

# IPC 简介

在学习 Cocos Creator 的插件编写之前，我们先要理解 Cocos Creator 插件开发中的重要一环，进程间通信（IPC）。Cocos Creator 的编辑器是基于 GitHub 开发的 [Electron](https://github.com/atom/electron) 内核。Electron 是一个集成了 Node.js 和 Chromimu 的跨平台开发框架。

在 Electron 的架构中，一份应用程序由主进程和渲染进程组成，其主进程负责管理平台相关的调度，如窗口的开启关闭，菜单选项，基础对话框等等。而每一个新开启的窗口就是一个独立的渲染进程。在 Electron 中，每个进程独立享有自己的 JavaScript 内容，彼此之间无法直接访问。当我们需要在进程之间传递数据时，就需要使用进程间通信（IPC）。你可以通过阅读 [Electron's introduction document](https://github.com/atom/electron/blob/master/docs/tutorial/quick-start.md) 更深入的理解 Electron 中的主进程和渲染进程的关系。简单点说，Electron 的主进程相当于一个 Node.js 服务端程序，而每一个窗口（渲染进程）则相当于一份客户端网页程序。

Cocos Creator 沿用了 Electron 的主进程和渲染进程的设计结构，在 Creator 启动前，我们将许多服务如：资源数据库，脚本编译器，预览服务器，打包工具等在主进程中开启，之后我们在主窗口也就是渲染进程中启动编辑器的界面操作部分。当界面操作需要使用主进程的模块时，我们就会进行进程间的通信（IPC）来完成请求。

## 进程间通信（IPC）

前面我们已经说到了两个进程之间的 JavaScript 内容是相互独立的，必须靠进程间通信的方式来交换数据。进程间通信实际上就是在一个进程中发消息，然后在另外一个进程中监听消息的过程。Electron 为我们提供了进程间通信对应的模块 [ipcMain](https://github.com/atom/electron/blob/master/docs/api/ipc-main.md) 和 [ipcRenderer](https://github.com/atom/electron/blob/master/docs/api/ipc-renderer.md) 来帮助我们完成这个任务。由于这两个模块仅完成了非常基本的通信功能，并不能满足编辑器，插件面板与主进程之间的通信需求，所以 Cocos Creator 在这之上又进行了封装，扩展了进程间消息收发的方法，方便插件开发者和编辑器开发者制作更多复杂情景。

## IPC 消息命名规范

IPC 的消息名就是一个字符串，对于消息的发送者，可以自由的进行消息的命名。但是我们希望开发人员在 Cocos Creator 中定制消息时遵守一定的规范，防止混乱。消息的命名规范为：

```javascript
'module-name:action-name'
// or
'package-name:action-name'
```

你可以在扩展程序中定义自己的的 IPC 消息，也可以监听编辑器内置的各个 IPC 消息，请参考 [编辑器内置的 IPC 消息列表](https://docs.cocos.com/creator/manual/zh/extension/reference/ipc-reference.html)。

## 扩展包中的进程关系

Cocos Creator 的扩展包也分为主进程和渲染进程两个部分。我们在上一篇文章中已经介绍了扩展包的入口程序，它是一份跑在主进程的脚本程序（就是前面声明过的入口程序 `main.js`，下一节会对 [入口程序](https://docs.cocos.com/creator/manual/zh/extension/entry-point.html) 进行详细介绍），如果你的扩展包需要用户界面，那么还需要一个或多个渲染进程（会在后面的 [扩展编辑器面板](https://docs.cocos.com/creator/manual/zh/extension/extends-panel.html) 介绍）。

# 入口程序

每一个插件都可以指定一份入口程序，入口程序是在 **主进程** 中被执行的。通常我们会在入口程序中：

- 初始化扩展包
- 执行后台操作程序（文件I/O，服务器逻辑）
- 调用 Cocos Creator 主进程中的方法
- 管理扩展面板的开启和关闭，以及响应主菜单和其他面板发送来的 IPC 消息

这里有一份入口程序的最简单样例：

```javascript
'use strict';

module.exports = {
  load () {
    Editor.log('package loaded');
  },

  unload () {
    Editor.log('package unloaded');
  },
};
```

## 生命周期回调

### load

当扩展包正确载入后，将会执行用户入口程序中的 load 函数。我们可以在这里做一些关于扩展包本身的初始化操作。

### unload

当扩展包卸载进行到最后阶段，将会执行用户入口程序中的 unload 函数。我们可以在这里做一些扩展包卸载前的清理操作。

### 加载和卸载注意事项

Cocos Creator 支持在编辑器运行时动态的添加和删除扩展包，所以要注意如果扩展包依赖编辑器其他模块的特定工作状态时，必须在 `load` 和 `unload` 里进行妥善处理。如果插件的动态加载和卸载导致其他模块工作异常时，扩展包的用户总是可以选择关闭编辑器后重新启动。

## IPC 消息注册

在入口程序中添加 `messages` 字段，可以让扩展包在加载的时候进行主进程的 IPC 消息注册。样例如下：

```javascript
'use strict';

module.exports = {
  messages {
    'foo-bar' ( event ) { Editor.log('hello foobar'); },
    'scene:saved' ( event ) { Editor.log('scene saved!'); },
  },
};
```

通过上面的例子，我们可以看到注册的 IPC 消息接受两种格式：

### 短命名消息

短命名消息是指消息名不带 `:` 的消息。这些消息将被视为该扩展包内的消息，在注册阶段，实际注册是，会被写成 `${你的扩展包名}:${消息名}`。以上面的代码为例，假设我们的扩展包名字为 `simple-demo`，那么 `foo-bar` 这个消息实际注册时，将会扩展成 `simple-demo:foo-bar`。

实际应用中，我们就可以通过 `Editor.Ipc.sendToPackage` 函数发送 IPC 消息到主进程的指定扩展包的注册函数中。

```javascript
Editor.Ipc.sendToPackage('simple-demo', 'foo-bar');
```

当然，我们还有更多的消息发送策略，我们会在后续的章节中进行详细介绍。

### 全名消息

全名消息就是指消息名中带有 `:` 的消息命名方式。通常我们书写全名消息是为了监听其他扩展包，或者其他模块的 IPC 消息。通过全名消息很清晰地知道这份消息是从哪个扩展包中发出的，也更好的避免了消息冲突的问题。

如上面的例子，我们可以清楚的了解到，“scene:saved” 这个消息是从 scene 这个内置扩展中发送的。

# 扩展包工作流程模式

在设计和开发扩展包时，我们总是希望扩展包在我们给予一定的输入时，完成特定的工作并返回结果。这个过程可以由以下几种工作模式来完成：

## 入口程序完成全部工作

如果我们的插件不需要任何用户输入，而且只要一次性的执行一些主进程逻辑，我们可以将所有工作放在 `main.js` 的 `load` 生命周期回调里：

```js
// main.js
module.exports = {
  load () {
    let fs = require('fs');
    let path = require('path');
    // 插件加载后在项目根目录自动创建指定文件夹
    fs.mkdirSync(path.join(Editor.Project.path, 'myNewFolder'));
    Editor.success('New folder created!');
  }
}
```

如果你的插件会自动完成工作，别忘记通过 `Editor.log`, `Editor.success` 接口（上述接口可以在 [Console API](https://docs.cocos.com/creator/manual/zh/extension/api/editor-framework/main/console.html#) 查看详情），来告诉用户刚刚完成了哪些工作。

示例中使用到的 `Editor.Project.path` 接口会返回当前打开项目的绝对路径，详情可以在 [Editor API](https://docs.cocos.com/creator/manual/zh/extension/api/editor-framework/main/editor.html) 中找到。

这种工作模式的更推荐的变体是将执行工作的逻辑放在菜单命令后触发，如 [第一个扩展包](https://docs.cocos.com/creator/manual/zh/extension/your-first-extension.html) 文档所示，我们在 `package.json` 里定义了 `main-menu` 字段和选择菜单项后触发的 IPC 消息，之后就可以在入口程序里监听这个消息并开始实际的业务逻辑：

```js
  messages: {
    'start' () {
      //开始工作！
    }
  }
```

关于菜单命令的声明，请参考 [扩展主菜单](https://docs.cocos.com/creator/manual/zh/extension/extends-main-menu.html)。

## 入口程序和编辑器面板配合实现复杂交互功能

入口程序除了可以在主进程执行 [Node.js](http://nodejs.org/) 所有标准接口以外，还可以打开编辑器面板、窗口，并通过 IPC 消息在主进程的入口程序和渲染进程的编辑器面板间进行通讯，通过编辑器面板和用户进行复杂的交互，并在相关的进程中完成业务逻辑的处理。

要通过入口程序打开一个编辑器面板：

```js
  messages: {
    'open' () {
      // open entry panel registered in package.json
      Editor.Panel.open('myPackage');
    }
  }
```

其中 `myPackage` 是面板的 ID，在单面板的扩展程序中，这个面板 ID 和扩展包名是一致的。用户可以通过 `package.json` 里的 `panel` 字段声明自定义的编辑器面板。我们会在下一节的 [扩展编辑器面板](https://docs.cocos.com/creator/manual/zh/extension/extends-panel.html) 文档中进行详细介绍。

启动面板后，在主进程和面板渲染进程间就可以通过 `Editor.Ipc.sendToPanel`，`Editor.Ipc.sendToMain` 等方法来进行进程间通讯，我们会在后面的文章中进行详细介绍。

## 插件只提供组件和资源

由于 Cocos Creator 本身采用的组件系统就有很高的扩展性和复用性，所以一些运行时相关的功能可以通过单纯开发和扩展组件的形式完成，而扩展包可以作为这些组件和相关资源（如 Prefab、贴图、动画等）的载体。通过扩展包声明字段 `runtime-resource` 可以将扩展包目录下的某个文件夹映射到项目路径下，并正确参与构建和编译等流程：

```json
//package.json
  "runtime-resource": {
    "path": "path/to/runtime-resource",
    "name": "shared-resource"
  }
```

上述声明会将 `projectPath/packages/myPackage/path/to/runtime-resource` 路径下的全部资源都映射到资源管理器中，显示为 `[myPackage]-[shared-resource]`。

这个路径下的内容（包括组件和其他资源）可以由项目中的其他场景、组件引用。使用这个工作流程，开发者可以将常用的控件、游戏架构以插件形式封装在一起，并在多个项目之间共享。

更多信息请阅读 [runtime-resource 字段参考](https://docs.cocos.com/creator/manual/zh/extension/reference/package-json-reference.html#runtime-resource-object-)。

# 扩展主菜单

Cocos Creator 的主菜单是可以自由扩展的。扩展方法是在 `package.json` 文件中的 `main-menu` 字段里，加入自己的菜单路径和菜单设置选项。下面是一份主菜单的配置样例：

```json
{
  "main-menu": {
    "Examples/FooBar/Foo": {
      "message": "my-package:foo"
    },
    "Examples/FooBar/Bar": {
      "message": "my-package:bar"
    }
  }
}
```

在样例中，我们通过配置菜单路径，在主菜单 "Example" > "Foobar" 里加入了 "Foo" 和 "Bar" 两个菜单选项。当我们点击这个菜单项，他会发送定义在其 `message` 字段中的 IPC 消息到主进程中。比如我们点击 "Foo" 将会发送 `my-package:foo` 这个消息。

## 菜单路径

在主菜单中注册中，其键值（Key）需要是一份菜单路径。菜单路径是 posix 路径格式，使用 `/` 作为分割符。在主菜单注册过程中，Cocos Creator 会根据路径一级一级往下寻找子菜单项，如果在寻找路径中没有找到对应的子菜单，就会在对应的路径中进行创建，直到最后一级菜单，将被当做菜单项加入。

在注册过程中也会遇到一些出错的情况：

### 注册的菜单项已经存在

这种情况多发生在多个扩展包之间，当你的扩展包和其他用户的扩展包的菜单注册路径相同时，就会发生该冲突。

### 注册菜单项的父级菜单已经被其他菜单项注册，其类型不是一个子菜单（submenu）

这个情况和上一种情况类似，我们可以用以下这个例子来说明：

```json
{
  "main-menu": {
    "Examples/FooBar": {
      "message": "my-package:foo"
    },
    "Examples/FooBar/Bar": {
      "message": "my-package:bar"
    }
  }
}
```

在这个例子中，我们先在主菜单中注册了一份菜单路径 `Examples/Foobar`，这之后我们又注册了 `Examples/Foobar/Bar`，而第二个菜单路径的注册要求 Foobar 的类型为一个分级子菜单（submenu），然而由于上一次的注册已经将 Foobar 的类型定义为菜单选项（menu-item），从而导致了注册失败。

## 菜单选项

在上面的例子中，我们已经使用了 `message` 菜单选项。菜单注册过程中还有许多其他的可选项，例如：icon、accelerator、type 等。更多选项，请阅读 [主菜单字段参考](https://docs.cocos.com/creator/manual/zh/extension/reference/main-menu-reference.html)。

## 插件专用菜单分类

为了避免用户安装多个插件时，每个插件随意注册菜单项，降低可用性，我们推荐所有编辑器扩展插件的菜单都放在统一的菜单分类里，并以插件包名对不同插件的菜单项进行划分。

插件专用的菜单分类路径是 `i18n:MAIN_MENU.package.title`，在中文语言环境会显示为一级菜单 `插件`，接下来的二级菜单路径名应该是插件的包名，最后的三级路径是具体的菜单项功能，如：

```
i18n:MAIN_MENU.package.title/FooBar/Bar
```

在中文环境的编辑器下就会显示如 `插件/FooBar/Bar` 这样的菜单。

`i18n:MAIN_MENU.package.title` 是多语言专用的路径表示方法，详情请见 [扩展包多语言化](https://docs.cocos.com/creator/manual/zh/extension/i18n.html) 文档。

# 扩展编辑器面板

Cocos Creator 允许用户定义一份面板窗口做编辑器的 UI 交互。

## 定义方法

在插件的 package.json 文件中定义 `panel` 字段如下:

```json
{
  "name": "simple-package",
  "panel": {
    "main": "panel/index.js",
    "type": "dockable",
    "title": "Simple Panel",
    "width": 400,
    "height": 300
  }
}
```

目前编辑器扩展系统支持每个插件注册一个面板，面板的信息通过 `panel` 字段对应的对象来申明。其中 `main` 字段用来标记面板的入口程序，和整个扩展包的入口程序概念类似，`panel.main` 字段指定的文件路径相当于扩展包在渲染进程的入口。

另外值得注意的是 `type` 字段规定了面板的基本类型：

- `dockable`：可停靠面板，打开该面板后，可以通过拖拽面板标签到编辑器里，实现扩展面板嵌入到编辑器中。下面我们介绍的面板入口程序都是按照可停靠面板的要求声明的。
- `simple`：简单 Web 面板，不可停靠到编辑器主窗口，相当于一份通用的 HTML 前端页面。详细情况请见 [定义简单面板](https://docs.cocos.com/creator/manual/zh/extension/define-simple-panel.html)。

其他面板定义的说明请参考 [面板字段参考](https://docs.cocos.com/creator/manual/zh/extension/reference/panel-json-reference.html)。

## 定义入口程序

要定义一份面板的入口程序，我们需要通过 `Editor.Panel.extend()` 函数来注册面板。如以下代码：

```javascript
// panel/index.js
Editor.Panel.extend({
  style: `
    :host { margin: 5px; }
    h2 { color: #f90; }
  `,

  template: `
    <h2>标准面板</h2>
    <ui-button id="btn">点击</ui-button>
    <hr />
    <div>状态: <span id="label">--</span></div>
  `,

  $: {
    btn: '#btn',
    label: '#label',
  },

  ready () {
    this.$btn.addEventListener('confirm', () => {
      this.$label.innerText = '你好';
      setTimeout(() => {
        this.$label.innerText = '--';
      }, 500);
    });
  },
});
```

`Editor.Panel.extend()` 接口传入的参数是一个包括特定字段的对象，用来描述整个面板的外观和功能。

在这份对象代码中，我们定义了面板的样式（style）和模板（template），并通过定义选择器 `$` 获得面板元素，最后在 ready 初始化回调函数中对面板元素的事件进行注册和处理。

在完成了上述操作后，我们就可以通过在主进程（入口程序）调用 `Editor.Panel.open('simple-package')` 激活我们的面板窗口。关于 `Editor.Panel` 接口的用法请参考 [Panel API](https://docs.cocos.com/creator/manual/zh/extension/api/editor-framework/main/panel.html)。

更多关于面板定义对象字段的说明，请阅读 [面板定义参考](https://docs.cocos.com/creator/manual/zh/extension/reference/panel-reference.html)。

## 在主菜单中添加打开面板选项

为了方便打开窗口，通常我们会将打开窗口的方法注册到主菜单中，并通过发消息给插件主进程代码来完成。要做到这些事情，我们需要在 `package.json` 中注册主进程入口函数和主菜单选项：

```json
{
  "name": "simple-package",
  "main": "main.js",
  "main-menu": {
    "Panel/Simple Panel": {
      "message": "simple-package:open"
    }
  },
  "panel": {
    "main": "panel/index.js",
    "type": "dockable",
    "title": "Simple Panel",
    "width": 400,
    "height": 300
  }
}
```

## 在主菜单中添加打开面板选项

为了方便打开窗口，通常我们会将打开窗口的方法注册到主菜单中，并通过发消息给插件主进程代码来完成。要做到这些事情，我们需要在 `package.json` 中注册主进程入口函数和主菜单选项：

```json
{
  "name": "simple-package",
  "main": "main.js",
  "main-menu": {
    "Panel/Simple Panel": {
      "message": "simple-package:open"
    }
  },
  "panel": {
    "main": "panel/index.js",
    "type": "dockable",
    "title": "Simple Panel",
    "width": 400,
    "height": 300
  }
}
```

更多关于在 `package.json` 文件中注册面板时的字段描述，请阅读 [面板字段参考](https://docs.cocos.com/creator/manual/zh/extension/reference/panel-json-reference.html)。

## 窗口面板与主进程交互

通常我们需要在窗口面板中设置一些 UI，然后通过发送 IPC 消息将任务交给主进程处理。这里我们可以使用 `Editor.Ipc` 模块来完成，在上面定义的 `index.js` 的 `ready()` 函数中处理按钮消息来达成。

```javascript
this.$btn.addEventListener('confirm', () => {
    Editor.Ipc.sendToMain('simple-package:say-hello', 'Hello, this is simple panel');
});
```

当点击按钮时，它将会给插件主进程发送 'say-hello' 消息，并附带对应的参数。你可以用任何你能想得到的前端技术编辑你的窗口界面，还可以结合 Electron 的 内置 node 在窗口内 require 你希望的 node 模块，完成任何你希望做的操作。

更全面和详细的主进程和面板之间的 IPC 通讯交互方法，请继续阅读 [进程间通讯工作流程](https://docs.cocos.com/creator/manual/zh/extension/ipc-workflow.html)。

# 进程间通讯（IPC）工作流程

关于扩展包进程间通讯（以下简称 IPC）的基本概念，请先阅读 [IPC 简介](https://docs.cocos.com/creator/manual/zh/extension/introduction-to-ipc.html)。

我们前面介绍了主进程中的 [入口程序](https://docs.cocos.com/creator/manual/zh/extension/entry-point.html) 和渲染进程中的 [面板程序](https://docs.cocos.com/creator/manual/zh/extension/extends-panel.html) 的基本声明方法和交互方式，接下来我们将结合实际需求介绍两种进程间通讯的详细工作流程。

本节提及的所有相关 API 均可查询 [Editor.Ipc 主进程 API](https://docs.cocos.com/creator/manual/zh/extension/api/editor-framework/main/ipc.html) 和 [Editor.Ipc 渲染进程 API](https://docs.cocos.com/creator/manual/zh/extension/api/editor-framework/renderer/ipc.html)。

## 发送消息

### 主进程向面板发送消息

在主进程中，主要使用

```
Editor.Ipc.sendToPanel('panelID', 'message' [, ...args, callback, timeout])
```

接口向特定面板发送消息。对于目前支持的单面板插件来说：

- `panelID` 面板 ID，对于单面板扩展包来说，面板 ID 就是插件的包名，如 `foobar`
- `message` 是 IPC 消息的全名，如 `do-some-work`，我们推荐在定义 IPC 消息名时使用 `-` 来连接单词，而不是使用驼峰或下划线。
- **可选** `args`，从第三个参数开始，可以定义数量不定的多个传参，用于传递更具体的信息到面板进程。
- **可选** `callback`，在传参后面可以添加回调方法，在面板进程中接受到 IPC 消息后可以选择向主进程发送回调，并通过 callback 回调方法进行处理。回调方法的参数第一个是 error（如果没有错误则传入 `null`），之后才是传参。
- **可选** `timeout`，回调超时，只能配合回调方法一起使用，如果规定了超时，在消息发送后的一定时间内没有接到回调方法，就会触发超时错误。如果不指定超时，则默认的超时设置是 5000 毫秒。

### 面板向主进程发送消息

```
Editor.Ipc.sendToMain('message', [, ...args, callback, timeout])
```

和 `sendToPanel` 相比，除了缺少第一个面板 ID 参数之外，其他参数的意义和用法完全相同。

### 其他消息发送方法

主进程对面板和面板对主进程是最常见的两种消息发送方式，但实际上 IPC 不局限于不同的两类进程之间，我们可以把消息发送方式做以下归类：

- 任意进程对主进程 `Editor.Ipc.sendToMain`
- 任意进程对面板 `Editor.Ipc.sendToPanel`
- 任意进程对编辑器主窗口（也就是对主窗口里的所有渲染进程广播）`Editor.Ipc.sendToMainWin`
- 任意进程对所有窗口（对包括弹出窗口在内的所有窗口渲染进程广播）`Editor.Ipc.sendToWins`
- 任意进程对所有进程广播 `Editor.Ipc.sendToAll`

上述方法在两种进程里写法都是一致的，只要注意消息接收的对象是在渲染进程还是主进程，并选择对应的方法即可。详细的接口用法请参考上文的描述和本文最上面的 IPC 接口文档链接。

**注意: 由于通讯基于 Electron 的底层 IPC 实现，所以切记传输的数据不可以包含原生对象，否则可能导致进程崩溃或者内存暴涨。推荐只传输纯 JSON 对象。**

## 接收消息

要在主进程或渲染进程中接受 IPC 消息，最简单的办法是在声明对象的 `messages` 字段中注册以 IPC 消息为名的消息处理方法。

### 面板渲染进程消息监听

```js
//packages/foobar/panel/index.js
Editor.Panel.extends({
    //...
    messages: {
        'my-message': function (event, ...args) {
            //do some work
        }
    }
});
```

### 主进程消息监听

```js
//packages/foobar/main.js
module.exports = {
    //...
    messages: {
        'my-message': function (event, ...args) {
            //do some work
        }
    }
}
```

注册监听消息时，我们使用的消息名是省略了扩展包名的短命名，上述消息短名 `my-message` 在发送时应该是 `Editor.Ipc.sendToPanel('foobar:my-message')` 和 `Editor.Ipc.sendToMain('foobar:my-messages')`。

可以看到主进程和渲染进程中监听 IPC 消息的函数声明方式是一致的，传入的第一个参数是一个 `event` 对象，我们可以通过这个对象发送回调。

### 其他消息监听方式

除了在 `messages` 字段内注册之外还可以使用 Electron 的 Ipc 消息接口来监听，形式上更灵活：

渲染进程中：

```js
require('electron').ipcRenderer.on('foobar:message', function(event, args) {});
```

主进程中：

```js
require('electron').ipcMain.on('foobar:message', function(event, args) {});
```

关于 Electron 的 IPC 接口可以参考 [Electron API: ipcMain](http://electron.atom.io/docs/api/ipc-main/) [Electron API: ipcRenderer](http://electron.atom.io/docs/api/ipc-renderer/)。

## 向消息来源发送回调

假如我们从主进程发送了一个消息：

```js
//packages/foobar/main.js
Editor.Ipc.sendToPanel('foobar', 'greeting', 'How are you?', function (error, answer) {
    Editor.log(answer);
});
```

在面板监听消息的方法中，我们可以使用 `event.reply` 来发送回调：

```js
//packages/foobar/panel/index.js
Editor.Panel.extends({
    //...
    messages: {
        'greeting': function (event, question) {
            console.log(question); //How are you?
            if (event.reply) {
                //if no error, the first argument should be null
                event.reply(null, 'Fine, thank you!');
            }
        }
    }
});
```

注意 `event.reply` 第一个参数是报错，没有错误时应该传入 `null`，此外建议总是检查 `event.reply` 是否存在，如果发送消息时参数中不包含回调方法，则 `event.reply` 的检查将返回 `undefined`，这种情况下调用 `event.reply` 会产生错误。

### 处理超时

发送消息时的最后一个参数是超时时限，单位是毫秒，如果未指定超时时限，则使用默认的 5000 ms 超时限制。

如果要取消超时限制，最后一次参数应该传入 `-1`，这种情况下应该靠其他逻辑保证回调必将触发。

从消息发送开始，在超过规定的时限后仍然没有接到消息监听方法中返回的回调的话，就会收到系统发送的超时错误回调：

```js
Editor.Ipc.sendToMain('foobar:greeting', function (error, answer) {
  if ( error && error.code === 'ETIMEOUT' ) { //check the error code to confirm a timeout
    Editor.error('Timeout for ipc message foobar:greeting');
    return;
  }
  Editor.log(answer);
});
```

# 定义简单面板

有时候我们希望扩展的面板并不用停靠在主窗口中，而是希望他是一个独立窗口，利用标准的 HTML 页面载入 方式展现。这个时候我们可以考虑使用 `simple` 类型的面板窗口。

## 定义方法

在插件的 package.json 文件中定义 `panel` 字段如下:

```json
{
  "name": "simple-package",
  "panel": {
    "main": "panel/index.html",
    "type": "simple",
    "title": "Simple Panel",
    "width": 400,
    "height": 300
  }
}
```

通过定义 `panel` 字段，并申明面板 `type` 为 `simple` 我们即可获得该份面板窗口。通过定义 `main` 字段我们可以为我们的面板窗口置顶一份入口 html。

和 [扩展编辑器面板](https://docs.cocos.com/creator/manual/zh/extension/extends-panel.html) 中介绍的面板定义的区别在于，`type` 使用 `simple`，而 `main` 索引的是 html 文件而不是 javascript 文件。

接下来我们可以定义一份简单的 html 入口，就和我们写任何网页一样:

```html
<html>
  <head>
    <title>Simple Panel Window</title>
    <meta charset="utf-8">
    <style>
      body {
        margin: 10px;
      }

      h1 {
        color: #f90
      }
    </style>
  </head>

  <body>
    <h1>A simple panel window</h1>
    <button id="btn">Send Message</button>

    <script>
      let btn = document.getElementById('btn');
      btn.addEventListener('click', () => {
        Editor.log('on button clicked!');
      });
    </script>
  </body>
</html>
```

在完成了上述操作后，我们就可以通过 `Editor.Panel.open('simple-package')` 激活我们的窗口。

使用简单窗口更接近于纯粹的网页编程，也更适合那些将已有 Web APP 移植或整合到 Cocos Creator 中的情况。

# 多语言化 (i18n)

编辑器扩展系统中内置的多语言方案允许你配置多份语言的键值映射，并根据编辑器当前的语言设置在插件界面显示不同语言的文字。

要启用多语言功能（以下简称 i18n），请在扩展包的目录下新建一个名叫 `i18n` 的文件夹，并为每种语言添加一个相应的 JavaScript 文件，作为键值映射数据。数据文件名应该和语言的代号一致，如 `en.js` 对应英语映射数据。

下面是键值映射数据源的例子：

```javascript
// en.js
module.exports = {
  'search': 'Search',
  'edit': 'Edit',
};

// zh.js
module.exports = {
  'search': '搜索',
  'edit': '编辑',
};
```

该文件将会将包含的映射注册到 `i18n` 的全局表里以扩展包名命名的部分里，假设包名为 `foobar`，则对应搜索文字的完整 key 为 `foobar.search`。

## 显示对应语言的文本

### 在脚本中使用

在 JavaScript 脚本中，你可以通过 `Editor.T` 接口获取当前语言对应的翻译后的文本：

```javascript
// NOTE: my package name is "foobar"
Editor.T('foobar.search');
```

在编辑器面板的模板定义文件里，也可以直接使用这个接口：

```javascript
// NOTE: my package name is "foobar"
Editor.Panel.extend({
  template: `
    <div class="btn">${Editor.T('foobar.edit')}</div>
  `
});
```

### 在菜单项中使用

在扩展包的 `package.json` 中注册菜单路径时支持 i18n 格式的路径，该类路径以 `i18n:${key}` 的形式表示。我们可以写 `i18n:MAIN_MENU.package.title/foobar/i18n:foobar.edit`，Cocos Creator 会帮助我们查找对应的 i18n 字符串并进行替换。

其中 `MAIN_MENU.package.title` 是 Cocos Creator 内置主菜单里的一级菜单名，为了方便大家使用插件，请尽量将扩展包的菜单注册到这个一级菜单下面，上述菜单路径变成用户实际看到的菜单路径就是 `扩展/foobar/编辑`。

# 工作路径和常用 URL

## 访问项目路径

- `Editor.Project.path`（主进程）当前在编辑器打开项目的根目录绝对路径。

## 自定义协议 URL

由于主进程和渲染进程在路径查询上有复杂的差异，我们引入了一些自定义协议的 URL 来方便的访问各个不同模块和路径的文件

- `db://` 在 [管理项目资源](https://docs.cocos.com/creator/manual/zh/extension/asset-management.html) 中介绍过，这个协议会映射到项目根目录，可以直接写 `db://assets/script/MyScript.js` 来获取特定的项目文件。注意在运行编辑器时的插件加载阶段，不能使用 `Editor.url('db://')`，这个阶段还没有初始化项目路径，要在编辑器窗口初始化完毕，项目文件全部加载后才能使用。

- `packages://` 映射到项目本地的插件目录 `packages` 和全局的插件目录 `$HOME/.CocosCreator/packages`，也就是说在这两个目录下的任何扩展包和其中的文件都可以通过这个协议索引，如 `packages://foobar/package.json` 就表示 `foobar` 这个扩展包中的配置文件。

- ```
  unpack://
  ```

   

  访问 Cocos Creator 安装目录下的开源内容，包括

  - `unpack://engine` JavaScript 引擎路径
  - `unpack://cocos2d-x` C++ 引擎路径
  - `unpack://simulator` 模拟器路径

要将这些自定义协议 URL 转换为绝对路径，使用 `Editor.url()` 接口。

### 声明面板时使用独立的 HTML 和 CSS 文件

使用 `Editor.url` 配合插件路径，我们就可以在声明面板的时候读取其他文件里包含的 HTML 和 CSS 定义，如：

```js
var Fs = require('fs');
Editor.Panel.extend({
  // css style for panel
  style: Fs.readFileSync(Editor.url('packages://foobar/panel/index.css', 'utf8')),

  // html template for panel
  template: Fs.readFileSync(Editor.url('packages://foobar/panel/index.html', 'utf8')),
  //...
});
```

# 提交资源到商店

Cocos Creator 内置了 [扩展商店](https://docs.cocos.com/creator/manual/zh/extension/install-and-share.html)，可供用户浏览、下载和自动安装官方或者第三方插件、资源。同时用户也可以将自己开发的扩展插件、美术素材、音乐音效等资源提交到扩展商店以便分享或者售卖。接下来我们以提交扩展插件为例来看一下具体的提交流程。

## 打包插件

假设开发者完成的插件扩展包目录结构如下：

```
foobar
    |--panel
        |--index.js
    |--package.json
    |--main.js
```

开发者需要将 `foobar` 文件夹打包成 `foobar.zip` 文件，然后提交到 Cocos 开发者中心。

更多关于插件扩展包的内容，可参考文档 [创建扩展包](https://docs.cocos.com/creator/manual/zh/extension/your-first-extension.html)。

### 第三方库

目前扩展包安装系统中没有安装 NPM 等包括管理系统的工作流程，因此使用了第三方库的扩展包应该将 `node_modules` 等文件夹也一起打包到 zip 包中。

## 提交插件

访问 [Cocos 开发者中心](https://auth.cocos.com/#/) 并登录，然后进入 [商店](https://store-my.cocos.com/#/seller/resources/) 栏目，点击右上方的 **发布新资源**。

![img](../image/Cocos拓展编辑器/create.png)

- 然后进入 **资源类别** 页面，填写 **名称** 和 **类别**，勾选已阅读协议。

  ![img](../image/Cocos拓展编辑器/category.png)

  - **名称**：插件显示在扩展商店中的名称。需要注意的是，名称一经确认后不可更改，请谨慎填写。
  - **类别**：要提交的资源类别，我们这里选择 **Creator 扩展 -> 插件**。

  设置完成后点击 **下一步**，进入 **资源介绍** 页面。

- 在 **资源介绍** 页面填写相关信息。

- 
  ![img](https://docs.cocos.com/creator/manual/zh/extension/submit-to-store/introduction.png)

  - **关键字**：方便用户更快搜索到您的插件，支持多个关键字
  - **支持平台**：包括 Android、iOS、HTML5
  - **图标**：图标尺寸为 **256 \* 256**，大小不超过 **500KB**，**png** 格式。
  - **截图**：最多上传 **5** 张截图，**jpg** / **png** 格式。每张截图的尺寸限制范围为最小 **640px**，最大 **2048px**，且大小不超过 **1000KB**。
  - **资源描述**：填写插件的基本功能和使用方法。包括 **中文** 和 **英文** 两种语言，填写后才会显示在对应语言版本的扩展商店中。例如：只有填写了英文的名称和描述后，插件才会显示在英文版的扩展商店中。

  填写完成后点击 **下一步** 进入 **定价** 页面。

- 在 **定价** 页面设置插件的售价，包括 **CNY** 和 **USD** 两种，如果免费请填写 **0**。

  ![img](https://docs.cocos.com/creator/manual/zh/extension/submit-to-store/pricing.png)

  填写完成后点击 **下一步** 进入 **上传资源** 页面

- 在 **上传资源** 页面上传插件扩展包资源并填写相关信息。

  ![img](https://docs.cocos.com/creator/manual/zh/extension/submit-to-store/upload.png)

  - **资源包**：zip 格式，最大 100MB。
  - **插件扩展包名**：插件扩展包的名称，定义在扩展包的 `package.json` 文件中。
  - **版本号**：插件版本号，定义在扩展包的 `package.json` 文件中。书写规范请遵守 [semver 规范](http://semver.org/lang/zh-CN/)。
  - **Creator 版本要求**：插件对 Creator 版本的要求。

  填写完成后点击 **下一步** 进入 **提交审核** 页面。

- 在 **提交审核** 页面点击 **提交审核** 按钮即可，也可以点击 **查看** 按钮重新编辑该插件资源。

  ![img](https://docs.cocos.com/creator/manual/zh/extension/submit-to-store/submit-for-review.png)

- 提交审核后，扩展商店的管理人员会在 **3** 个工作日内审核插件内容和信息：

  - 如果没有问题，则审核通过并将插件上架到扩展商店。
  - 如果有问题需要修改，则审核不通过并备注理由。

  以上审核结果会发送到 Cocos 开发者账号的注册邮箱，请及时关注邮件。

# 调用引擎 API 和项目脚本

在插件中可以声明一个特殊的脚本文件（场景脚本），该脚本和项目中的脚本（`assets` 目录下的脚本）具有相同的环境，也就是说在这个脚本里可以调用引擎 API 和其他项目脚本，实现：

- 遍历场景中的节点，获取或改动数据
- 调用项目中的其他脚本完成工作

## 注册场景脚本

首先在 `package.json` 里添加 `scene-script` 字段，该字段的值是一个脚本文件的路径，相对于扩展包目录：

```Json
{
    "name": "my-plugin-name",
    "scene-script": "scene-walker.js"
}
```

该路径将指向 `packages/foobar/scene-walker.js`，接下来我们看看如何编写场景脚本。

## 编写场景脚本

`scene-walker.js` 需要用这样的形式定义：

```js
module.exports = {
    'get-canvas-children': function (event) {
        var canvas = cc.find('Canvas');
        Editor.log('children length : ' + canvas.children.length);

        if (event.reply) {
            event.reply(null, canvas.children.length);
        }
    }
};
```

可以看到场景脚本由一个或多个 IPC 消息监听方法组成，收到相应的 IPC 消息后，我们在函数体内可以使用包括全部引擎 API 和用户组件脚本里声明的方法和属性。

## 从扩展包中向场景脚本发送消息

接下来在扩展包程序的主进程和渲染进程中，都可以使用下面的接口来向 `scene-walker.js` 发送消息（假设扩展包名是 `foobar`）：

```js
Editor.Scene.callSceneScript('foobar', 'get-canvas-children', function (err, length) {
    console.log(`get-canvas-children callback : length - ${length}`);
});
```

这样就可以在扩展包中获取到场景里的 `Canvas` 根节点有多少子节点，当然还可以用来对场景节点进行更多的查询和操作。

在发送消息时 `callSceneScript` 接受的参数输入和其他 IPC 消息发送接口一致，也可以指定更多传参和 timeout 超时时限。详情请看 [IPC 工作流程](https://docs.cocos.com/creator/manual/zh/extension/ipc-workflow.html)。

**注意: 由于通讯基于 Electron 的底层 IPC 实现，所以切记传输的数据不可以包含原生对象，否则可能导致进程崩溃或者内存暴涨。推荐只传输纯 JSON 对象。**

## 在场景脚本中引用模块和插件脚本

除了通过 `cc.find` 在场景脚本中获取特定节点，并操作该节点和挂载的组件以外，我们还可以引用项目中的非组件模块，或者通过全局变量访问插件脚本。

### 引用模块

```js
//MyModule.js
module.exports = {
    init: function () {
        //do initialization work
    }
}

//scene-walker.js
module.exports = {
    'init-module': function (event) {
        var myModule = cc.require('MyModule');
        myModule.init();
    }
};
```

注意，要使用和项目脚本相同的模块引用机制，在场景脚本里必须使用 `cc.require` 的写法。

### 引用插件脚本

直接使用 `window.globalVar` 来访问插件脚本里声明的全局变量和方法即可。

# 管理项目资源

## 管理场景

### 新建场景

通过 `Editor.Ipc` 模块新建场景:

```
Editor.Ipc.sendToPanel('scene', 'scene:new-scene');
```

### 保存当前场景

对场景数据修改完成后可以通过 `Editor.Ipc` 模块来保存当前场景:

```
Editor.Ipc.sendToPanel('scene', 'scene:stash-and-save');
```

### 加载其他场景

我们的扩展包可能需要遍历多个场景并依次操作和保存，在上一节 [调用引擎 API 和项目脚本](https://docs.cocos.com/creator/manual/zh/extension/scene-script.html) 中我们介绍了通过场景脚本访问引擎 API 和用户项目脚本的方法，要加载新场景，请使用：

```js
_Scene.loadSceneByUuid(uuid, function(error) {
    //do more work
});
```

其中 _Scene 是一个特殊的单例，用来控制场景编辑器里加载的场景实例。
传入的参数是场景资源的 uuid，可以通过下面介绍的资源管理器接口来获取。

## 资源 URL 和 UUID 的映射

在 Cocos Creator 编辑器和扩展中，资源的 url 由形如

```
db://assets/path/to/scene.fire
```

这样的形式表示。其中 `db` 是 AssetDB 的简称。项目中 `assets` 路径下的全部资源都会被 AssetDB 导入到资源库（library）中，并可以通过 uuid 来引用。

在扩展包的主进程中 url 和 uuid 之间可以互相转化：

- `Editor.assetdb.urlToUuid(url)`
- `Editor.assetdb.uuidToUrl(uuid)`

此外如果希望直接使用资源在本地文件系统中的绝对路径，也可以使用 `fspathToUuid` 和 `uuidToFspath` 接口，其中 `fspath` 就表示绝对路径。

## 管理资源

### 导入资源

要将新资源导入到项目中，可以使用以下接口

```js
// main process
Editor.assetdb.import(['/User/user/foo.js', '/User/user/bar.js'], 'db://assets/foobar', function ( err, results ) {
    results.forEach(function ( result ) {
    // result.uuid
    // result.parentUuid
    // result.url
    // result.path
    // result.type
    });
});


// renderer process
Editor.assetdb.import( [
    '/file/to/import/01.png',
    '/file/to/import/02.png',
    '/file/to/import/03.png',
], 'db://assets/foobar', callback);
```

### 创建资源

使用扩展包管理资源的一个常见误区，就是当扩展包需要创建新资源时直接使用了 Node.js 的 [fs 模块](https://nodejs.org/dist/latest-v6.x/docs/api/fs.html)，这样即使创建文件到了 `assets` 目录，也无法自动被资源管理器导入。正确的工作流程应该是使用 `create` 接口来创建资源。

```js
// main process or renderer process
Editor.assetdb.create( 'db://assets/foo/bar.js', data, function ( err, results ) {
    results.forEach(function ( result ) {
    // result.uuid
    // result.parentUuid
    // result.url
    // result.path
    // result.type
    });
});
```

传入的 `data` 就是该资源文件内容的字符串。在创建完成后会自动进行该资源的导入操作，回调成功后就可以在资源管理器中看到该资源了。

### 保存已有资源

要使用新的数据替换原有资源内容，可以使用以下接口

```js
// main process or renderer process
Editor.assetdb.saveExists( 'db://assets/foo/bar.js', data, function ( err, meta ) {
    // do something
});
```

如果要在保存前检查资源是否存在，可以使用

```js
// main process
Editor.assetdb.exists(url); //return true or false
```

在渲染进程，如果给定了一个目标 url，如果该 url 指向的资源不存在则创建，资源存在则保存新数据的话，可以使用

```js
// renderer process
Editor.assetdb.createOrSave( 'db://assets/foo/bar/foobar.js', data, callback);
```

### 刷新资源

当资源文件在 `assets` 中已经修改，而由于某种原因没有进行重新导入的情况下，会出现 `assets` 里的资源数据和数据库里展示的资源数据不一致的情况（如果使用 `fs` 模块直接操作文件内容就会出现），可以通过手动调用资源刷新接口来重新导入资源

```js
// main process or renderer process
Editor.assetdb.refresh('db://assets/foo/bar/', function (err, results) {});
```

### 移动和删除资源

由于资源导入后会生成对应的 `meta` 文件，所以单独删除和移动资源文件本身都会造成数据库中数据一致性受损，推荐使用专门的 AssetDB 接口来完成这些工作

```js
Editor.assetdb.move(srcUrl, destUrl);
Editor.assetdb.delete([url1, url2]);
```

关于这些接口的详情，请查阅 [AssetDB API Main](https://docs.cocos.com/creator/manual/zh/extension/api/asset-db/asset-db-main.html) 和 [AssetDB API Renderer](https://docs.cocos.com/creator/manual/zh/extension/api/asset-db/asset-db-renderer.html)。

# 编写面板界面

Cocos Creator 的面板界面使用 HTML5 标准编写。你可以为界面指定 HTML 模板和 CSS 样式，然后对界面元素绑定消息编写逻辑和交互代码。如果你之前有过前端页面编程经验，那么这些内容对你来说再熟悉不过。而没有前端编程经验的开发者也不必太担心，通过本节的学习，将可以让你在短时间内掌握 Creator 面板界面的编写技巧。

## 定制你的模板

通常在开始编写界面之前，我们总是希望能够在界面中看见点什么。我们可以通过面板定义函数的 `template` 和 `style` 选项来稍微在面板界面上绘制点东西。

一般我们会选择绘制一些区块用于规划界面布局，我们可以写以下代码：

```javascript
Editor.Panel.extend({
  style: `
    .wrapper {
      box-sizing: border-box;
      border: 2px solid white;
      font-size: 20px;
      font-weight: bold;
    }

    .top {
      height: 20%;
      border-color: red;
    }

    .middle {
      height: 60%;
      border-color: green;
    }

    .bottom {
      height: 20%;
      border-color: blue;
    }
  `,

  template: `
    <div class="wrapper top">
      Top
    </div>

    <div class="wrapper middle">
      Middle
    </div>

    <div class="wrapper bottom">
      Bottom
    </div>
  `,
});
```

通过以上代码，我们获得了一个如下的界面效果：

![panel-01](../image/Cocos拓展编辑器/panel-01.png)

## 界面排版

界面排版是通过在 `style` 中书写 CSS 来完成的。在上面的例子中，我们已经对界面做了简单的排版。如果对 CSS 不熟悉，推荐大家可以阅读 [W3 School 的 CSS 教程](http://www.w3school.com.cn/css/) 来加强。

在界面排版过程中，有时候我们希望更好的表达元素之间的布局关系，比如我们喜欢 Top 和 Bottom 元素的高度固定为 30px，而 Middle 元素的高度则撑满剩余空间。这个时候我们就可以使用 [CSS Flex](https://css-tricks.com/snippets/css/a-guide-to-flexbox/) 布局来制作。

我们可以这么修改 `style` 部分：

```javascript
Editor.Panel.extend({
  style: `
    :host {
      display: flex;
      flex-direction: column;
    }

    .wrapper {
      box-sizing: border-box;
      border: 2px solid white;
      font-size: 20px;
      font-weight: bold;
    }

    .top {
      height: 30px;
      border-color: red;
    }

    .middle {
      flex: 1;
      border-color: green;
    }

    .bottom {
      height: 30px;
      border-color: blue;
    }
  `
});
```

由于 CSS Flex 布局语法有些复杂，为了方便大家使用，Cocos Creator 对这部分进行了重新包装，关于这部分的详细介绍，请阅读 [界面排版](https://docs.cocos.com/creator/manual/zh/extension/layout-ui-element.html)。

## 添加 UI 元素

规划好布局后，我们就可以考虑加入界面元素来完成界面功能。通常，熟悉前端编程的开发人员会想到一些常用的界面元素，如 `<button>`，`<input>` 等等。这些元素当然是可以直接被使用，但是我们强烈推荐大家使用 Cocos Creator 的内置 UI Kit 元素。这些内置元素都是以 `ui-` 开头，例如 `<ui-button>`，`<ui-input>`。

Cocos Creator 提供了非常丰富的内置元素，开发人员可以通过 [掌握 UI Kit](https://docs.cocos.com/creator/manual/zh/extension/using-ui-kit.html) 章节获得更详细 的了解。内置元素不但在样式上经过细致的调整，同时也统一了消息发送规则并且能够更好的处理 focus 等系统事件。

让我们稍微丰富一下我们上面的面板：

```javascript
Editor.Panel.extend({
  style: `
    :host {
      display: flex;
      flex-direction: column;
      margin: 5px;
    }

    .top {
      height: 30px;
    }

    .middle {
      flex: 1;
      overflow: auto;
    }

    .bottom {
      height: 30px;
    }
  `,

  template: `
    <div class="top">
      Mark Down 预览工具
    </div>

    <div class="middle layout vertical">
      <ui-text-area resize-v value="请编写你的 Markdown 文档"></ui-text-area>
      <ui-markdown class="flex-1"></ui-markdown>
    </div>

    <div class="bottom layout horizontal end-justified">
      <ui-button class="green">预览</ui-button>
    </div>
  `,
});
```

如果一切正常，你将会看到如下界面：

![panel-02](../image/Cocos拓展编辑器/panel-02.png)

## 为 UI 元素添加逻辑交互

最后让我们通过标准的事件处理代码来完成面板的逻辑部分。假设我们需要在每次点击预览按钮后，都会将 text-area 中输入的 Markdown 文档，渲染并显示在下方。我们可以做如下代码操作：

```javascript
Editor.Panel.extend({
  // ...

  $: {
    txt: 'ui-text-area',
    mkd: 'ui-markdown',
    btn: 'ui-button',
  },

  ready () {
    this.$btn.addEventListener('confirm', () => {
      this.$mkd.value = this.$txt.value;
    });

    // init
    this.$mkd.value = this.$txt.value;
  },
});
```

这里我们通过 `$` 选择器，预先索引了我们需要的 ui 元素。再利用 HTML 标准 API `addEventListener` 为元素添加事件。对于内置 UI Kit 元素，每个 UI 元素都拥有一组标准事件，它们分别是：`cancel`、`change` 和 `confirm`。同时，多数 UI 元素都会携带 `value` 属性，记录元素内相关的值信息。

希望通过本节的代码示例，能够启发你进行面板界面开发的工作。当然，要灵活运用面板界面，还是需要深入学习和掌握 HTML5 标准。

# 掌握 UI Kit

Cocos Creator 为开发者提供了非常丰富的界面元素，帮助开发者快速地开发面板界面。与此同时，我们还为开发者提供了控件预览面板，方便开发者在使用控件时，查看控件的各种属性和这些属性对应的效果。

要打开控件预览窗口，仅需要在主菜单中选择 **开发者 -> UI Kit Preview**。

目前界面元素包括：

## 控件

- [ui-button](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-button.html)
- [ui-checkbox](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-checkbox.html)
- [ui-color](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-color.html)
- [ui-input](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-input.html)
- [ui-num-input](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-num-input.html)
- [ui-select](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-select.html)
- [ui-slider](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-slider.html)
- [ui-text-area](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-text-area.html)

## 特殊控件

- [ui-asset](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-asset.html)
- [ui-node](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-node.html)

## 控件容器

- [ui-box-container](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-box-container.html)
- [ui-drop-area](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-drop-area.html)
- [ui-prop](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-prop.html)
- [ui-prop-table](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-prop-table.html)
- [ui-section](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-section.html)
- [ui-webview](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-webview.html)

## 视图元素

- [ui-hint](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-hint.html)
- [ui-loader](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-loader.html)
- [ui-markdown](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-markdown.html)
- [ui-progress](https://docs.cocos.com/creator/manual/zh/extension/reference/ui-progress.html)

# 扩展 UI Kit

如果发现 Cocos Creator 内置的界面元素仍然满足不了你的需求，也不必太担心，你可以通过自定义元素 来对 UI Kit 进行扩展。

UI Kit 的扩展是基于 HTML5 的 [Custom Elements](http://www.html5rocks.com/zh/tutorials/webcomponents/customelements/) 标准。

通过 `Editor.UI.registerElement(tagName, prototype)` 我们可以很轻松的注册自定义元素。这里 有一个简单的范例。

```javascript
Editor.UI.registerElement('foobar-label', {
  template: `
    <div class="text">Foobar</div>
  `,

  style: `
    .text {
      color: black;
      padding: 2px 5px;
      border-radius: 3px;
      background: #09f;
    }
  `
});
```

当你完成上面的代码，并在 Panel 中正确 require 后，你就可以在编辑器中使用 `<foobar-label>` 这个元素。

更多关于自定义元素定义的选项，请阅读[自定义界面元素定义参考](https://docs.cocos.com/creator/manual/zh/extension/reference/custom-element-reference.html)。

## 内容分布

有时候我们会在自定义元素内加入内容，为了能够让自定义元素正确处理内容，需要在模板中通过 `<content>` 标签加以说明。 这个过程我们称作“内容分布”。

拿上面的例子来说，假设我们希望 `<foobar-label>` 不只是显示 Foobar，而是根据我们加入的内容进行显示，例如：

```javascript
<foobar-label>Hello World</foobar-label>
```

这个时候就需要使用内容分发功能。我们可以对之前的范例做一个小小的更改：

```javascript
template: `
  <div class="text">
    <content></content>
  </div>
`
```

通过使用 `<content>` 标签告诉样板，我们希望将用户内容放置在这个地方。

## 内容分布选择器

有时候自定义元素的内容不止是文字，而是一些复合元素，我们在做内容分发的时候，希望有些元素在某些标签下，有些元素位于另外的标签中。这个时候就可以考虑使用内容分发选择器。考虑如下样例：

```javascript
  <foobar-label>
    <div class="title">Hello World</div>
    <div class="body">This is Foobar</div>
  </foobar-label>
```

如果我们希望将 `.title` 和 `.body` 元素内容区分对待，我们可以做如下代码：

```javascript
  template: `
    <div class="text title">
      <content select=".title"></content>
    </div>
    <div class="text body">
      <content select=".body"></content>
    </div>
  `
```

通过 `<content>` 标签中加入 `select` 属性，我们可以利用选择器来分布内容。

# 界面排版

Cocos Creator 的界面排版是通过在 `style` 中书写 CSS 来完成的。如果对 CSS 不熟悉，推荐大家可以阅读 [W3 School 的 CSS 教程](http://www.w3school.com.cn/css/) 来加强。

然而普通的 CSS 排版并不适合界面元素，为此，CSS 最新标准中加入了 [CSS Flex](https://css-tricks.com/snippets/css/a-guide-to-flexbox/) 布局。通过 Flex 布局，我们可以很轻易的对界面元素进行横排和纵排。为了方便开发者使用 CSS Flex，Cocos Creator 也对其进行了一些封装。本章节主要就是介绍 Cocos Creator 中的界面排版方法。

## 横排和纵排

**layout horizontal**

```html
<div class="layout horizontal">
  <div>1</div>
  <div class="flex-1">2 (flex-1)</div>
  <div>3</div>
</div>
```

![layout-horizontal](../image/Cocos拓展编辑器/layout-horizontal.png)

**layout vertical**

```html
<div class="layout vertical">
  <div>1</div>
  <div class="flex-1">2 (flex-1)</div>
  <div>3</div>
</div>
```

![layout-vertical](../image/Cocos拓展编辑器/layout-vertical.png)

## 对齐元素

我们在横排纵排时，会希望对所有子元素进行对齐操作。可以通过 `start`、`center` 和 `end` 来进行子元素的对齐操作。

对于横排元素，它们分别代表上，中，下对齐。
对于纵排元素，它们分别代表左，中，右对齐。

以横排为例，我们来看一组例子：

```html
<div class="layout horizontal start">
  <div>1</div>
  <div>2</div>
  <div>3</div>
</div>
<div class="layout horizontal center">
  <div>1</div>
  <div>2</div>
  <div>3</div>
</div>
<div class="layout horizontal end">
  <div>1</div>
  <div>2</div>
  <div>3</div>
</div>
```

![layout-align-items](../image/Cocos拓展编辑器/layout-align-items.png)

有时候，我们需要对排版容器中的某个元素进行对齐调整，这个时候就可以通过 `self-` 关键字来操作。在 Cocos Creator 中，我们提供了：`self-start`、`self-center`、`self-end` 和 `self-stretch`。

让我们以横排为例，来看看这么做的效果：

```html
<div class="layout horizontal">
  <div class="self-start">self-start</div>
  <div class="self-center">self-center</div>
  <div class="self-end">self-end</div>
  <div class="self-stretch">self-stretch</div>
</div>
```

![layout-self-align](../image/Cocos拓展编辑器/layout-self-align.png)

## 元素分布

元素分布主要描述元素在排版方向上如何分布。比如所有元素都从排版容器的左边开始排，或者从右边，或者根据元素大小散布在排版容器中。

我们提供了：`justified`、`around-justified`、`start-justified`、`center-justified` 和 `end-justified`。

还是以横排为例，来看一组例子：

```html
<div class="layout horizontal justified">
  <div>1</div>
  <div>2</div>
  <div>3</div>
</div>
<div class="layout horizontal around-justified">
  <div>1</div>
  <div>2</div>
  <div>3</div>
</div>
  ...
  ...
```

![layout-justified](../image/Cocos拓展编辑器/layout-justified.png)

## 元素自适应

有些时候我们希望元素撑满布局的剩余控件。我们可以通过在布局容器的子元素中使用 `flex-1`，`flex-2`，…… `flex-12` 来操作。

来看一组例子：

```html
<div class="layout horizontal">
  <div class="flex-1">flex-1</div>
  <div class="flex-2">flex-2</div>
  <div class="flex-3">flex-3</div>
</div>
<div class="layout horizontal">
  <div class="flex-none">flex-none</div>
  <div class="flex-1">flex-1</div>
  <div class="flex-none">flex-none</div>
</div>
  ...
  ...
```

![layout-flex](../image/Cocos拓展编辑器/layout-flex.png)

还有些时候我们希望元素本身就撑满容器的整个空间。这个时候就可以考虑使用 `fit` 这个 class。方法和效果如下：

```html
<div class="wrapper">
  <div class="fit">fit</div>
</div>
```

![layout-fit](../image/Cocos拓展编辑器/layout-fit.png)

# 在面板中使用 Vue

如果你已经掌握了 [编写面板界面](https://docs.cocos.com/creator/manual/zh/extension/writing-your-panel.html) 这章中的界面编写方法，你或许会觉得这样编写界面有些繁琐。是否能够使用一些前端界面框架来提升界面编写效率呢？答案是肯定的。Cocos Creator 支持任何界面框架如 [Vue](http://vuejs.org/)，[React](https://facebook.github.io/react/)、[Polymer](http://polymer-project.org/) 等等。

在测试过程中，我们发现 [Vue](http://vuejs.org/) 非常符合 Cocos Creator 的整体设计思路，所以我们重点介绍一下如何在 Cocos Creator 中使用 [Vue](http://vuejs.org/) 编写面板界面。

## 部署 Vue

事实上你不用做任何准备工作，Cocos Creator 的面板窗口在打开时就会默认加载 Vue。

## 初始化 Vue 面板

我们可以在 `ready()` 函数中初始化 Vue 面板。初始化方式如下：

```javascript
ready () {
  new window.Vue({
    el: this.shadowRoot,
  });
}
```

通过传入 `panel-frame` 的 shadow root 元素，我们可以让 Vue 在该元素节点下生成一份 vm。让我们来看一个更详细的使用例子：

```javascript
Editor.Panel.extend({
  style: `
    :host {
      margin: 10px;
    }
  `,

  template: `
    <h2>A Simple Vue Panel</h2>

    <input v-model="message">
    <p>Input Value = <span>{{message}}</span></p>
  `,

  ready () {
    new window.Vue({
      el: this.shadowRoot,
      data: {
        message: 'Hello World',
      },
    });
  },
});
```

## 数据绑定

我们可以在面板的 `template` 关键字中，定义 Vue 的数据绑定规则。然后通过在 Vue 定义的 `data` 关键字中写入绑定数据来完成整个操作。

具体例子如下：

```javascript
Editor.Panel.extend({
  template: `
    <ui-button>{{txtOK}}</ui-button>
    <ui-button v-if="showCancel">{{txtCancel}}</ui-button>
    <ui-input v-for="item in items" value="{{item.message}}"></ui-input>
  `,

  ready () {
    new window.Vue({
      el: this.shadowRoot,
      data: {
        txtOK: 'OK',
        txtCancel: 'Cancel',
        showCancel: false,
        items: [
          { message: 'Foo' },
          { message: 'Bar' },
        ]
      },
    });
  },
});
```

## 事件绑定

除了使用数据绑定，我们还可以通过 Vue 的 `@` 方式来将事件和方法绑定在一起。值得注意的是，绑定的 方法必须定义在 Vue 定义中的 `methods` 关键字中。

具体例子如下：

```javascript
Editor.Panel.extend({
  template: `
    <ui-button @confirm="onConfirm">Click Me</ui-button>
  `,

  ready () {
    new window.Vue({
      el: this.shadowRoot,
      methods: {
        onConfirm ( event ) {
          event.stopPropagation();
          console.log('On Confirm!');
        },
      },
    });
  },
});
```

# 扩展 Inspector

Inspector 是在 **属性检查器** 里展示的组件控件面板，有时候我们需要对自己书写的 Component 定义一份 Inspector，用自定义的方式对组件属性显示样式的修改。如最经典的 Widget 组件的样式就是通过扩展 inspector 获得的。

![extend inspector](../image/Cocos拓展编辑器/extend-inspector.png)

下面让我们进行一次简单的扩展，步骤如下：

1. 在 Component 中注明自定义 Inspector 的扩展文件
2. 创建自定义 Inspector 的扩展包
3. 在扩展包中编写自定义 Inspector 的扩展文件

## 在 Component 中注明自定义 Inspector 的扩展文件

首先我们需要定义一份 Component 脚本，并且为这个脚本注明使用自定义 Inspector，方法如下：

```javascript
cc.Class({
  name: 'Foobar',
  extends: cc.Component,
  editor: {
    inspector: 'packages://foobar/inspector.js',
  },
  properties: {
    foo: 'Foo',
    bar: 'Bar'
  },
});
```

**注意1:** 这里我们定义了一个 `editor` 字段，并在该字段中定义了 `inspector` 文件。编辑器会自动根据 `inspector.js` 文件内定义的 Vue 模板在 `inspector` 面板中生成对应框架。

**注意2:** 在 `inspector` 中我们采用 `packages://` 协议定义文件路径。Cocos Creator 会将 `packages://` 协议后面的分路径名当做扩展包名字进行搜索，并根据搜索结果将整个协议替换成扩展包的路径并做后续搜索。

## 创建自定义 Inspector 的扩展包

和我们创建一份扩展包没有任何区别，你可以按照 [你的第一个扩展包](https://docs.cocos.com/creator/manual/zh/extension/your-first-extension.html) 中的方式创建一份 `main.js` 和 `package.json` 文件。这里我们假设我们的扩展包名为 foobar。

注意，在创建完扩展包后，你需要重启一下 Cocos Creator 以便让他正确读入该扩展包。

## 在扩展包中编写自定义 Inspector 的扩展文件

接下来我们就可以在 `foobar` 包中定义 `inspector.js`：

```javascript
// target 默认指向 Componet 自定义组件
Vue.component('foobar-inspector', {
  // 修改组件在 inspector 的显示样式
  template: `
    <ui-prop v-prop="target.foo"></ui-prop>
    <ui-prop v-prop="target.bar"></ui-prop>
  `,

  props: {
    target: {
      twoWay: true,
      type: Object,
    },
  },
});
```

Cocos Creator 的 Inspector 扩展使用了 [Vue](http://vuejs.org/)。这里我们通过定义一份 Vue 的组件，并在组件中定义 `props`，使得其包含 `target` 数据来完成整个 Inspector 的数据定义。

该 `target` 就是我们的 `Foobar` Class 在 Inspector 中对应的实例。

## 关于 target

上一小节中提到的 `target` 实例是经过 Inspector 处理过的 target。其内部包含了对属性的加工处理。在使用的时候，我们不能简单的认为 `target.foo` 就代表 foo 的值。如果你去查看 `target.foo` 你会发现他是一个 Object 而不是我们在最开始定义的那样一个 'Foo' 的字符串。该份 Object 中包含了 `attrs`、`type`、`value` 等信息。其中的 `value` 才是我们真正的值。

这么做的目的是为了让 Inspector 可以更好的获得数据可视化的各方面信息。例如当你定义了 cc.Class 的属性为：

```javascript
properties: {
  foo: {
    default: 'Foo',
    readonly: true
  }
}
```

这个时候在 Inspector 中的 target 里反应出的信息为 `target.foo.value` 为 'Foo'，`target.foo.attrs.readonly` 为 `true`。这些信息有助于帮助你创建多变的界面组合。

## 关于属性绑定

由于这些信息非常繁琐，Cocos Creator 也对 Vue 的 directive 做了一定的扩展。目前我们扩展了 `v-prop`，`v-value`，`v-readonly` 和 `v-disable`。

当你还是想利用 Cocos Creator 的默认方案显示数据段时，你可以使用 `v-prop` 配合 `<ui-prop>` 控件做绑定，如：

```html
<ui-prop v-prop="target.foo"></ui-prop>
```

当你想使用 `<ui-xxx>` 的原生控件时，你可以使用 `v-value` 来做数据绑定，如：

```html
<ui-input v-value="target.foo.value"></ui-input>
```

当你想对控件应用 readonly 或者 disable 行为时，请使用 `v-readonly` 和 `v-disable`。如：

```html
<ui-button v-readonly="target.foo.attrs.readonly">Foo</ui-button>
<ui-button v-disable="target.bar.value">Bar</ui-button>
```