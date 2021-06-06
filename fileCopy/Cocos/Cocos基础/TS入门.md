# TS入

[toc]

TypeScript 是一种由微软开发的自由和开源的编程语言。它是 JavaScript 的一个严格超集，并添加了可选的静态类型和基于类的面向对象编程。TypeScript 的设计目标是开发大型应用，然后转译成 JavaScript 运行。由于 TypeScript 是 JavaScript 的超集，任何现有的 JavaScript 程序都是合法的 TypeScript 程序。

## TypeScript 和 Cocos Creator

Cocos Creator 的很多用户之前是使用其他强类型语言（如 C++/C#）来编写游戏的，因此在使用 Cocos Creator 的时候也希望能够使用强类型语言来增强项目在较大规模团队中的表现。

和其他 JavaScript 脚本一样，项目 `assets` 目录下的 TypeScript 脚本（.ts 文件) 在创建或修改后激活编辑器，就会被编译成兼容浏览器标准的 ES5 JavaScript 脚本。编译后的脚本存放在项目下的 `library`（还包括其他资源）目录。

### 在已有项目中添加 TypeScript 设置

如果希望在原有项目中添加 TypeScript 脚本，并获得 VS Code 等 IDE 的完整支持，需要执行主菜单的 **开发者 -> VS Code 工作流 -> 更新 VS Code 智能提示数据** 和 **开发者 -> VS Code 工作流 -> 添加 TypeScript 项目配置**，来添加 `creator.d.ts` 和 `tsconfig.json` 文件到你的项目根目录中。`creator.d.ts` 声明了引擎的所有 API，用于支持 VS Code 的智能提示。

`tsconfig.json` 用于设置 TypeScript 项目环境，你可以参考官方的 [tsconfig.json 说明](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html) 进行定制。

以下是常用的 `tsconfig.json` 配置方案：

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "lib": [ "es2015", "es2017", "dom" ],
    "target": "es5",
    "experimentalDecorators": true,
    "skipLibCheck": true,
    "outDir": "temp/vscode-dist"
  },
  "exclude": [
    "node_modules",
    "library",
    "local",
    "temp",
    "build",
    "settings"
  ]
}
```

### 在项目中创建 TypeScript 脚本

和创建 JavaScript 脚本一样，你可以直接在文本编辑器里新建 `.ts` 文件，或通过编辑器的 **资源管理器** 的创建菜单，右键点击一个文件夹，并选择 **新建 -> TypeScript**。

## 使用 TypeScript 声明 CCClass

在 [TypeScript 中 class 的声明方式](https://www.typescriptlang.org/docs/handbook/classes.html) 和 [ES6 Class](http://es6.ruanyifeng.com/#docs/class) 相似。但为了编辑器能够正确解析 **属性检查器** 里显示的各类属性，我们还需要使用引擎内置的一些装饰器，来将普通的 class 声明成 CCClass。这和目前将 JavaScript 中的 ES6 Class 声明为 CCClass 的方法类似。关于装饰器的更多信息请参考 [TypeScript decorator](http://www.typescriptlang.org/docs/handbook/decorators.html)。

下面是一个基本的 TypeScript 声明组件的实例：

```typescript
const {ccclass, property} = cc._decorator; // 从 cc._decorator 命名空间中引入 ccclass 和 property 两个装饰器

@ccclass // 使用装饰器声明 CCClass
export default class NewClass extends cc.Component { // ES6 Class 声明语法，继承 cc.Component

    @property(cc.Label)     // 使用 property 装饰器声明属性，括号里是属性类型，装饰器里的类型声明主要用于编辑器展示
    label: cc.Label = null; // 这里是 TypeScript 用来声明变量类型的写法，冒号后面是属性类型，等号后面是默认值

    // 也可以使用完整属性定义格式
    @property({
        visible: false
    })
    text: string = 'hello';

    // 成员方法
    onLoad() {
        // init logic
    }
}
```

装饰器使用 `@` 字符开头作为标记，装饰器主要用于编辑器对组件和属性的识别，而 TypeScript 语法中的类型声明 `myVar: Type` 则允许 VS Code 编码时自动识别变量类型并提示其成员。

### 更多属性类型声明方法

- 声明值类型

  ```typescript
    @property({
        type: cc.Integer
    })
    myInteger = 1;
  
    @property
    myNumber = 0;
  
    @property
    myText = "";
  
    @property(cc.Node)
    myNode: cc.Node = null;
  
    @property
    myOffset = new cc.Vec2(100, 100);
  ```

- 声明数组

  ```typescript
    @property([cc.Node])
    public myNodes: cc.Node[] = [];
  
    @property([cc.Color])
    public myColors: cc.Color[] = [];
  ```

- 声明 getset

  ```typescript
    @property
    _width = 100;
  
    @property
    get width () {
        return this._width;
    }
  
    set width (value) {
        cc.log('width changed');
        this._width = value;
    }
  ```

**注意**：TypeScript 的 public, private 修饰符不影响成员在 **属性检查器** 中的默认可见性，默认的可见性仍然取决于成员变量名是否以下划线开头。

### 提示其他组件属性和方法

首先我们声明一个组件：

```typescript
// MyModule.ts
const {ccclass, property} = cc._decorator;

@ccclass
export class MyModule extends cc.Component {
    @property(cc.String)
    myName: string = "";

    @property(cc.Node)
    myNode: cc.Node = null;
}
```

然后在其他组件中 import MyModule, 并且声明一个 `MyModule` 类型的成员变量：

```typescript
// MyUser.ts
const {ccclass, property} = cc._decorator;
import {MyModule} from './MyModule';

@ccclass
export class MyUser extends cc.Component {
    @property(MyModule)
    public myModule: MyModule = null;

    /*
     * // 声明自定义类型数组
     * @property(MyModule)
     * public myModule: MyModule[] = [];
     *
     * @property({
     *     type: MyModule
     * })
     * public myModule: MyModule[] = [];
     */

    public onLoad() {
        // init logic
        this.myModule.myName = 'John';
    }
}
```

输入 `this.myModule.` 时，就可以提示我们在 `MyModule.ts` 中声明的属性了。

![auto complete](https://docs.cocos.com/creator/manual/zh/scripting/assets/auto-complete.gif)门

**注意：如果将已声明属性修改为数组类型，但是在编辑器中却未生效。那么请通过组件菜单对组件进行重置。**

Creator 对资源类型进行了部分调整。`cc.Texture2D`、`cc.AudioClip`、`cc.ParticleAsset` 类型数据在 ts 中的声明一定要按照以下的格式进行声明：

```typescript
@property({
    type: cc.Texture2D,
})
texture: cc.Texture2D = null;

@property({
    type: cc.Texture2D,
})
textures: cc.Texture2D[] = [];
```

## 使用命名空间

在 TypeScript 里，命名空间是位于全局命名空间下的一个普通的带有名字的 JavaScript 对象。通常用于在使用全局变量时为变量加入命名空间限制，避免污染全局空间。命名空间和模块化是完全不同的概念，命名空间无法导出或引用，仅用来提供通过命名空间访问的全局变量和方法。

reator 中默认所有 assets 目录下的脚本都会进行编译，自动为每个脚本生成模块化封装，以便脚本之间可以通过 `import` 或 `require` 相互引用。当希望把一个脚本中的变量和方法放置在全局命名空间，而不是放在某个模块中时，我们需要选中这个脚本资源，并在 **属性检查器** 里设置该脚本 `导入为插件`。设为插件的脚本将不会进行模块化封装，也不会进行自动编译。

**注意**：在微信、百度、小米、支付宝小游戏环境中，全局变量需要显式地给 window 设置属性才能成功声明，如 `window.data = {};`。

所以对于包含命名空间的 TypeScript 脚本来说，我们既不能将这些脚本编译并进行模块化封装，也不能将其设为插件脚本（会导致 TS 文件不被编译成 JS）。如果需要使用命名空间，我们需要使用特定的工作流程。

### 命名空间工作流程

下面我们通过一个范例介绍命名空间的工作流程。

1. 如果是首次使用，需要安装 TypeScript 编译器。在命令行中执行以下命令：

   ```bash
    npm install -g typescript
   ```

2. 用 VS Code 打开项目根目录中的 **tsconfig.json** 文件，然后在 `compilerOptions` 字段中设置 `outDir`：

   ```json
    {
      "compilerOptions": {
   
        "outDir": "temp/vscode-dist"
   
        ......
      },
   
      ......
    }
   ```

3. 在项目的根目录下（assets 目录外），新建一个文件夹并命名为 **namespaces**，用于存放所有包含命名空间的 ts 脚本。然后在该文件夹下新建一个脚本 **foo.ts**，代码如下：

   ```ts
    namespace Foo {
        export let bar: number = 1;
    }
   ```

4. 在 VS Code 中按下 **Ctrl/Cmd + Shift + P**，在弹出的 Command Palette 中输入 `task`，并选择 `Tasks: Configure Task`。然后继续在弹出的选项中选择 `tsc: build - tsconfig.json`。

5. 按下 **Ctrl/Cmd + Shift + B**，在 Command Palette 中选择 `tsc: build - tsconfig.json` 启动 ts 编译任务。可以看到在 **temp** 目录下生成了 **vscode-dist** 文件夹。进入该文件夹，找到编译后的脚本 **foo.js**，此时脚本内的内容应该是：

   ```js
    var Foo;
    (function (Foo) {
        Foo.bar = 1;
    })(Foo || (Foo = {}));
   ```

6. 将 **foo.js** 脚本拷贝到 **assets** 目录下的任意有效位置。

7. 回到 Creator 编辑器，在 **资源管理器** 中选中 **foo.js**，然后在 **属性检查器** 中勾选 **导入为插件**，完成后点击右上角的 **应用**。此时 **foo.js** 中定义的命名空间就可以正常的工作了。

以上就是在 Creator 中使用 TypeScript 命名空间的完整工作流程。

## 更新引擎接口声明数据

Creator 每个新版本都会更新引擎接口声明，建议升级了 Creator 后，通过主菜单的 **开发者 -> VS Code 工作流 -> 更新 VS Code 智能提示数据** 来更新已有项目的 `creator.d.ts` 文件。

# CCClass 进阶参考

相比其它 JavaScript 的类型系统，CCClass 的特别之处在于功能强大，能够灵活的定义丰富的元数据。CCClass 的技术细节比较丰富，你可以在开发过程中慢慢熟悉。本文档将列举它的详细用法，阅读前需要先掌握 [使用 cc.Class 声明类型](https://docs.cocos.com/creator/manual/zh/scripting/class.html)。

## 术语

- CCClass：使用 `cc.Class` 声明的类。
- 原型对象：调用 `cc.Class` 时传入的字面量参数。
- 实例成员：包含 **成员变量** 和 **成员方法**。
- 静态成员：包含 **静态变量** 和 **类方法**。
- 运行时：项目脱离编辑器独立运行时，或者在模拟器和浏览器里预览的时候。
- 序列化：解析内存中的对象，将它的信息编码为一个特殊的字符串，以便保存到硬盘上或传输到其它地方。

## 原型对象参数说明

所有原型对象的参数都可以省略，用户只需要声明用得到的部分即可。

```javascript
cc.Class({

    // 类名，用于序列化
    // 值类型：String
    name: "Character",

    // 基类，可以是任意创建好的 cc.Class
    // 值类型：Function
    extends: cc.Component,

    // 构造函数
    // 值类型：Function
    ctor: function () {},

    // 属性定义（方式一，直接定义）
    properties: {
        text: ""
    },

    // 属性定义（方式二，使用 ES6 的箭头函数，详见下文）
    properties: () => ({
        text: ""
    }),

    // 实例方法
    print: function () {
        cc.log(this.text);
    },

    // 静态成员定义
    // 值类型：Object
    statics: {
        _count: 0,
        getCount: function () {}
    },

    // 提供给 Component 的子类专用的参数字段
    // 值类型：Object
    editor: {
        disallowMultiple: true
    }
});
```

### 类名

类名可以是任意字符串，但不允许重复。可以使用 `cc.js.getClassName` 来获得类名，使用 `cc.js.getClassByName` 来查找对应的类。对在项目脚本里定义的组件来说，序列化其实并不使用类名，因此那些组件不需要指定类名。对其他类来说，类名用于序列化，如果不需要序列化，类名可以省略。

### 构造函数

#### 通过 ctor 定义

CCClass 的构造函数使用 `ctor` 定义，为了保证反序列化能始终正确运行，`ctor` **不允许** 定义 **构造参数**。

> 开发者如果确实需要使用构造参数，可以通过 `arguments` 获取，但要记得如果这个类会被序列化，必须保证构造参数都缺省的情况下仍然能 new 出对象。

#### 通过 `__ctor__` 定义

`__ctor__` 和 `ctor` 一样，但是允许定义构造参数，并且不会自动调用父构造函数，因此用户可以自行调用父构造函数。`__ctor__` 不是标准的构造函数定义方式，如果没有特殊需要请一律使用 `ctor` 定义。

## 判断类型

### 判断实例

需要做类型判断时，可以用 JavaScript 原生的 `instanceof`：

```javascript
var Sub = cc.Class({
    extends: Base
});

var sub = new Sub();
cc.log(sub instanceof Sub);       // true
cc.log(sub instanceof Base);      // true

var base = new Base();
cc.log(base instanceof Sub);      // false
```

### 判断类

使用 `cc.isChildClassOf` 来判断两个类的继承关系：

```javascript
var Texture = cc.Class();
var Texture2D = cc.Class({
    extends: Texture
});

cc.log(cc.isChildClassOf(Texture2D, Texture));   // true
```

两个传入参数都必须是类的构造函数，而不是类的对象实例。如果传入的两个类相等，`isChildClassOf` 同样会返回 true。

## 成员

### 实例变量

在构造函数中定义的实例变量不能被序列化，也不能在 **属性检查器** 中查看。

```javascript
var Sprite = cc.Class({
    ctor: function () {
        // 声明实例变量并赋默认值
        this.url = "";
        this.id = 0;
    }
});
```

> 如果是私有的变量，建议在变量名前面添加下划线 `_` 以示区分。

### 实例方法

实例方法请在原型对象中声明：

```javascript
var Sprite = cc.Class({
    ctor: function () {
        this.text = "this is sprite";
    },

    // 声明一个名叫 "print" 的实例方法
    print: function () {
        cc.log(this.text);
    }
});

var obj = new Sprite();
// 调用实例方法
obj.print();
```

### 静态变量和静态方法

静态变量或静态方法可以在原型对象的 `statics` 中声明：

```javascript
var Sprite = cc.Class({
    statics: {
        // 声明静态变量
        count: 0,
        // 声明静态方法
        getBounds: function (spriteList) {
            // ...
        }
    }
});
```

上面的代码等价于：

```javascript
var Sprite = cc.Class({ ... });

// 声明静态变量
Sprite.count = 0;
// 声明静态方法
Sprite.getBounds = function (spriteList) {
    // ...
};
```

静态成员会被子类继承，继承时会将父类的静态变量 **浅拷贝** 给子类，因此：

```javascript
var Object = cc.Class({
    statics: {
        count: 11,
        range: { w: 100, h: 100 }
    }
});

var Sprite = cc.Class({
    extends: Object
});

cc.log(Sprite.count);    // 结果是 11，因为 count 继承自 Object 类

Sprite.range.w = 200;
cc.log(Object.range.w);  // 结果是 200，因为 Sprite.range 和 Object.range 指向同一个对象
```

如果你不需要考虑继承，私有的静态成员也可以直接定义在类的外面：

```javascript
// 局部方法
function doLoad (sprite) {
    // ...
};
// 局部变量
var url = "foo.png";

var Sprite = cc.Class({
    load: function () {
        this.url = url;
        doLoad(this);
    };
});
```

## 继承

### 父构造函数

请注意，不论子类是否有定义构造函数，子类实例化前父类的构造函数都会被自动调用。

```javascript
var Node = cc.Class({
    ctor: function () {
        this.name = "node";
    }
});

var Sprite = cc.Class({
    extends: Node,
    ctor: function () {
        // 子构造函数被调用前，父构造函数已经被调用过，所以 this.name 已经被初始化过了
        cc.log(this.name);    // "node"
        // 重新设置 this.name
        this.name = "sprite";
    }
});

var obj = new Sprite();
cc.log(obj.name);    // "sprite"
```

因此你不需要尝试调用父类的构造函数，否则父构造函数就会重复调用。

```javascript
var Node = cc.Class({
    ctor: function () {
        this.name = "node";
    }
});

var Sprite = cc.Class({
    extends: Node,
    ctor: function () {
        Node.call(this);        // 别这么干！
        this._super();          // 也别这么干！

        this.name = "sprite";
    }
});
```

> 在一些很特殊的情况下，父构造函数接受的参数可能和子构造函数无法兼容。这时开发者就只能自己手动调用父构造函数并且传入需要的参数，这时应该将构造函数定义在 [`__ctor__`](https://docs.cocos.com/creator/manual/zh/scripting/reference/class.html#__ctor__) 中。

### 重写

所有成员方法都是虚方法，子类方法可以直接重写父类方法：

```javascript
var Shape = cc.Class({
    getName: function () {
        return "shape";
    }
});

var Rect = cc.Class({
    extends: Shape,
    getName: function () {
        return "rect";
    }
});

var obj = new Rect();
cc.log(obj.getName());    // "rect"
```

和构造函数不同的是，父类被重写的方法并不会被 CCClass 自动调用，如果你要调用的话：

- 方法一：使用 CCClass 封装的 `this._super`

  ```javascript
    var Shape = cc.Class({
        getName: function () {
            return "shape";
        }
    });
  
    var Rect = cc.Class({
        extends: Shape,
        getName: function () {
            var baseName = this._super();
            return baseName + " (rect)";
        }
    });
  
    var obj = new Rect();
    cc.log(obj.getName());    // "shape (rect)"
  ```

- 方法二：使用 JavaScript 原生写法

  ```javascript
    var Shape = cc.Class({
        getName: function () {
            return "shape";
        }
    });
  
    var Rect = cc.Class({
        extends: Shape,
        getName: function () {
            var baseName = Shape.prototype.getName.call(this);
            return baseName + " (rect)";
        }
    });
  
    var obj = new Rect();
    cc.log(obj.getName());    // "shape (rect)"
  ```

如果你想实现继承的父类和子类都不是 CCClass，只是原生的 JavaScript 构造函数，你可以用更底层的 API `cc.js.extend` 来实现继承。

## 属性

属性是特殊的实例变量，能够显示在 **属性检查器** 中，也能被序列化。

### 属性和构造函数

属性 **不用** 在构造函数里定义，在构造函数被调用前，属性已经被赋为默认值了，可以在构造函数内访问到。如果属性的默认值无法在定义 CCClass 时提供，需要在运行时才能获得，你也可以在构造函数中重新给属性赋 **默认** 值。

```javascript
var Sprite = cc.Class({
    ctor: function () {
        this.img = LoadImage();
    },

    properties: {
        img: {
            default: null,
            type: Image
        }
    }
});
```

不过要注意的是，属性被反序列化的过程紧接着发生在构造函数执行 **之后**，因此构造函数中只能获得和修改属性的默认值，还无法获得和修改之前保存（序列化）的值。

### 属性参数

所有属性参数都是可选的，但至少必须声明 `default`、`get`、`set` 参数中的其中一个。

#### default 参数

`default` 用于声明属性的默认值，声明了默认值的属性会被 CCClass 实现为成员变量。默认值只有在 **第一次创建** 对象的时候才会用到，也就是说修改默认值时，并不会改变已添加到场景里的组件的当前值。

> 当你在编辑器中添加了一个组件以后，再回到脚本中修改一个默认值的话，**属性检查器** 里面是看不到变化的。因为属性的当前值已经序列化到了场景中，不再是第一次创建时用到的默认值了。如果要强制把所有属性设回默认值，可以在 **属性检查器** 的组件菜单中选择 Reset。

`default` 允许设置为以下几种值类型：

- 任意 number, string 或 boolean 类型的值

- `null` 或 `undefined`

- 继承自 `cc.ValueType` 的子类，如 `cc.Vec2`, `cc.Color` 或 `cc.Rect` 的实例化对象：

  ```javascript
    properties: {
        pos: {
            default: new cc.Vec2(),
        }
    }
  ```

- 空数组 `[]` 或空对象 `{}`

- 一个允许返回任意类型值的 function，这个 function 会在每次实例化该类时重新调用，并且以返回值作为新的默认值：

  ```javascript
    properties: {
        pos: {
            default: function () {
                return [1, 2, 3];
            },
        }
    }
  ```

#### visible 参数

默认情况下，是否显示在 **属性检查器** 取决于属性名是否以下划线 `_` 开头。如果以下划线开头，则默认不显示在 **属性检查器**，否则默认显示。

如果要强制显示在 **属性检查器**，可以设置 `visible` 参数为 true:

```javascript
properties: {
    _id: {      // 下划线开头原本会隐藏
        default: 0,
        visible: true
    }
}
```

如果要强制隐藏，可以设置 `visible` 参数为 false:

```javascript
properties: {
    id: {       // 非下划线开头原本会显示
        default: 0,
        visible: false
    }
}
```

#### serializable 参数

指定了 `default` 默认值的属性默认情况下都会被序列化，序列化后就会将编辑器中设置好的值保存到场景等资源文件中，并且在加载场景时自动还原之前设置好的值。如果不想序列化，可以设置 `serializable: false`。

```javascript
temp_url: {
    default: "",
    serializable: false
}
```

#### type 参数

当 `default` 不能提供足够详细的类型信息时，为了能在 **属性检查器** 显示正确的输入控件，就要用 `type` 显式声明具体的类型：

- 当默认值为 null 时，将 type 设置为指定类型的构造函数，这样 **属性检查器** 才知道应该显示一个 Node 控件。

  ```javascript
    enemy: {
        default: null,
        type: cc.Node
    }
  ```

- 当默认值为数值（number）类型时，将 type 设置为 `cc.Integer`，用来表示这是一个整数，这样属性在 **属性检查器** 里就不能输入小数点。

  ```javascript
    score: {
        default: 0,
        type: cc.Integer
    }
  ```

- 当默认值是一个枚举（`cc.Enum`）时，由于枚举值本身其实也是一个数字（number），所以要将 type 设置为枚举类型，才能在 **属性检查器** 中显示为枚举下拉框。

  ```javascript
    wrap: {
        default: Texture.WrapMode.Clamp,
        type: Texture.WrapMode
    }
  ```

#### override 参数

所有属性都将被子类继承，如果子类要覆盖父类同名属性，需要显式设置 `override` 参数，否则会有重名警告：

```javascript
_id: {
    default: "",
    tooltip: "my id",
    override: true
},

name: {
    get: function () {
        return this._name;
    },
    displayName: "Name",
    override: true
}
```

更多参数内容请查阅 [属性参数](https://docs.cocos.com/creator/manual/zh/scripting/reference/attributes.html)。

### 属性延迟定义

如果两个类相互引用，脚本加载阶段就会出现循环引用，循环引用将导致脚本加载出错：

- Game.js

  ```javascript
    var Item = require("Item");
  
    var Game = cc.Class({
        properties: {
            item: {
                default: null,
                type: Item
            }
        }
    });
  
    module.exports = Game;
  ```

- Item.js

  ```javascript
    var Game = require("Game");
  
    var Item = cc.Class({
        properties: {
            game: {
                default: null,
                type: Game
            }
        }
    });
  
    module.exports = Item;
  ```

上面两个脚本加载时，由于它们在 require 的过程中形成了闭环，因此加载会出现循环引用的错误，循环引用时 type 就会变为 undefined。

因此我们提倡使用以下的属性定义方式：

- Game.js

  ```javascript
    var Game = cc.Class({
        properties: () => ({
            item: {
                default: null,
                type: require("Item")
            }
        })
    });
  
    module.exports = Game;
  ```

- Item.js

  ```javascript
    var Item = cc.Class({
        properties: () => ({
            game: {
                default: null,
                type: require("Game")
            }
        })
    });
  
    module.exports = Item;
  ```

这种方式就是将 properties 指定为一个 ES6 的箭头函数（lambda 表达式），箭头函数的内容在脚本加载过程中并不会同步执行，而是会被 CCClass 以异步的形式在所有脚本加载成功后才调用。因此加载过程中并不会出现循环引用，属性都可以正常初始化。

> 箭头函数的用法符合 JavaScript 的 ES6 标准，并且 Creator 会自动将 ES6 转义为 ES5，用户不用担心浏览器的兼容问题。

你可以这样来理解箭头函数：

```js
// 箭头函数支持省略掉 return 语句，我们推荐的是这种省略后的写法：

properties: () => ({    // <- 箭头右边的括号 "(" 不可省略
    game: {
        default: null,
        type: require("Game")
    }
})

// 如果要完整写出 return，那么上面的写法等价于：

properties: () => {
    return {
        game: {
            default: null,
            type: require("Game")
        }
    };      // <- 这里 return 的内容，就是原先箭头右边括号里的部分
}

// 我们也可以不用箭头函数，而是用普通的匿名函数：

properties: function () {
    return {
        game: {
            default: null,
            type: require("Game")
        }
    };
}
```

## GetSet 方法

在属性中设置了 get 或 set 以后，访问属性的时候，就能触发预定义的 get 或 set 方法。

### get

在属性中设置 get 方法：

```javascript
properties: {
    width: {
        get: function () {
            return this.__width;
        }
    }
}
```

get 方法可以返回任意类型的值。
这个属性同样能显示在 **属性检查器** 中，并且可以在包括构造函数内的所有代码里直接访问。

```javascript
var Sprite = cc.Class({
    ctor: function () {
        this.__width = 128;
        cc.log(this.width);    // 128
    },

    properties: {
        width: {
            get: function () {
                return this.__width;
            }
        }
    }
});
```

**注意**：

1. 设定了 get 以后，这个属性就不能被序列化，也不能指定默认值，但仍然可附带除了 `default`、`serializable` 外的大部分参数。

   ```javascript
    width: {
        get: function () {
            return this.__width;
        },
   
        type: cc.Integer,
        tooltip: "The width of sprite"
    }
   ```

2. get 属性本身是只读的，但返回的对象并不是只读的。用户使用代码依然可以修改对象内部的属性，例如：

   ```javascript
    var Sprite = cc.Class({
        ...
        position: {
            get: function () {
                return this._position;
            },
        }
        ...
    });
   
    var obj = new Sprite();
    obj.position = new cc.Vec2(10, 20);   // 失败！position 是只读的！
    obj.position.x = 100;                 // 允许！position 返回的 _position 对象本身可以修改！
   ```

### set

在属性中设置 set 方法：

```javascript
width: {
    set: function (value) {
        this._width = value;
    }
}
```

set 方法接收一个传入参数，这个参数可以是任意类型。

set 一般和 get 一起使用：

```javascript
width: {
    get: function () {
        return this._width;
    },

    set: function (value) {
        this._width = value;
    },

    type: cc.Integer,
    tooltip: "The width of sprite"
}
```

> 如果没有和 get 一起定义，则 set 自身不能附带任何参数。
> 和 get 一样，设定了 set 以后，这个属性就不能被序列化，也不能指定默认值。

使用 get/set 可以更好地封装接口，更多使用案例可参考社区教程 [CocosCreator 开发中为什么 get/set 如此重要？](https://mp.weixin.qq.com/s/gS6BTdBLTLzAtIUlHbBjtA)。

## editor 参数

`editor` 只能定义在 `cc.Component` 的子类。

```javascript
cc.Class({
  extends: cc.Component,

  editor: {

    // 允许当前组件在编辑器模式下运行。
    // 默认情况下，所有组件都只会在运行时执行，也就是说它们的生命周期回调在编辑器模式下并不会触发。
    //
    // 值类型：Boolean
    // 默认值：false
    executeInEditMode: false,

    // requireComponent 参数用来指定当前组件的依赖组件。
    // 当组件添加到节点上时，如果依赖的组件不存在，引擎将会自动将依赖组件添加到同一个节点，防止脚本出错。
    // 该选项在运行时同样有效。
    // 
    // 值类型：Function（必须是继承自 cc.Component 的构造函数，如 cc.Sprite）
    // 默认值：null
    requireComponent: null,

    // 脚本生命周期回调的执行优先级。
    // 小于 0 的脚本将优先执行，大于 0 的脚本将最后执行。
    // 该优先级只对 onLoad, onEnable, start, update 和 lateUpdate 有效，对 onDisable 和 onDestroy 无效。
    //
    // 值类型：Number
    // 默认值：0
    executionOrder: 0,

    // 当本组件添加到节点上后，禁止同类型（含子类）的组件再添加到同一个节点，
    // 防止逻辑发生冲突。
    // 
    // 值类型：Boolean
    // 默认值：false
    disallowMultiple: false,

    // menu 用来将当前组件添加到组件菜单中，方便用户查找。
    // 
    // 值类型：String（如 "Rendering/Camera"）
    // 默认值：""
    menu: "",

    // 当设置了 "executeInEditMode" 以后，playOnFocus 可以用来设定选中当前组件所在的节点时，
    // 编辑器的场景刷新频率。
    // playOnFocus 如果设置为 true，场景渲染将保持 60 FPS，如果为 false，场景就只会在必要的时候进行重绘。
    // 
    // 值类型：Boolean
    // 默认值：false
    playOnFocus: false,

    // 自定义当前组件在 **属性检查器** 中渲染时所用的网页 url。
    // 
    // 值类型：String
    // 默认值：""
    inspector: "",

    // 自定义当前组件在编辑器中显示的图标 url。
    // 
    // 值类型：String
    // 默认值：""
    icon: "",

    // 指定当前组件的帮助文档的 url，设置过后，在 **属性检查器** 中就会出现一个帮助图标，
    // 用户点击将打开指定的网页。
    // 
    // 值类型：String
    // 默认值：""
    help: "",
  }
});
```

# 属性参数

> 属性参数用来给已定义的属性附加元数据，类似于脚本语言的 Decorator 或者 C# 的 Attribute。

## 属性检查器相关参数

| 参数名      | 说明                                   | 类型             | 默认值    | 备注                                                         |
| :---------- | :------------------------------------- | :--------------- | :-------- | :----------------------------------------------------------- |
| type        | 限定属性的数据类型                     | (Any)            | undefined | 详见 [type 参数](https://docs.cocos.com/creator/manual/zh/scripting/reference/class.html#type-参数) |
| visible     | 在 **属性检查器** 中显示或隐藏         | boolean          | (注1)     | 详见 [visible 参数](https://docs.cocos.com/creator/manual/zh/scripting/reference/class.html#visible-参数) |
| displayName | 在 **属性检查器** 中显示为另一个名字   | string           | undefined | -                                                            |
| tooltip     | 在 **属性检查器** 中添加属性的 Tooltip | string           | undefined | -                                                            |
| multiline   | 在 **属性检查器** 中使用多行文本框     | boolean          | false     | -                                                            |
| readonly    | 在 **属性检查器** 中只读               | boolean          | false     | -                                                            |
| min         | 限定数值在编辑器中输入的最小值         | number           | undefined | -                                                            |
| max         | 限定数值在编辑器中输入的最大值         | number           | undefined | -                                                            |
| step        | 指定数值在编辑器中调节的步长           | number           | undefined | -                                                            |
| range       | 一次性设置 min, max, step              | [min, max, step] | undefined | step 值可选                                                  |
| slide       | 在 **属性检查器** 中显示为滑动条       | boolean          | false     | -                                                            |

## 序列化相关参数

这些参数不能用于 get 方法：

| 参数名               | 说明                       | 类型    | 默认值    | 备注                                                         |
| :------------------- | :------------------------- | :------ | :-------- | :----------------------------------------------------------- |
| serializable         | 序列化该属性               | boolean | true      | 详见 [serializable 参数](https://docs.cocos.com/creator/manual/zh/scripting/reference/class.html#serializable-参数) |
| formerlySerializedAs | 指定之前序列化所用的字段名 | string  | undefined | 重命名属性时，声明这个参数来兼容之前序列化的数据             |
| editorOnly           | 在导出项目前剔除该属性     | boolean | false     | -                                                            |

## 其它参数

| 参数名     | 说明                                  | 类型                     | 默认值    | 备注                                                         |
| :--------- | :------------------------------------ | :----------------------- | :-------- | :----------------------------------------------------------- |
| default    | 定义属性的默认值                      | (Any)                    | undefined | 详见 [default 参数](https://docs.cocos.com/creator/manual/zh/scripting/reference/class.html#default-参数) |
| notify     | 当属性被赋值时触发指定方法            | `function (oldValue) {}` | undefined | 需要定义 default 属性并且不能用于数组。 不支持 ES6 定义方式  |
| override   | 当重写父类属性时需要定义该参数为 true | boolean                  | false     | 详见 [override 参数](https://docs.cocos.com/creator/manual/zh/scripting/reference/class.html#override-参数) |
| animatable | 该属性是否能被 **动画编辑器** 修改    | boolean                  | undefined | -                                                            |

**注 1**：visible 的默认值取决于属性名。当属性名以下划线 `_` 开头时，默认隐藏，否则默认显示。

