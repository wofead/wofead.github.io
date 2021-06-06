# JS入门

[toc]

## JavaScript 对象（Object）

我们像这样声明一个对象（object）：

```js
myProfile = {
    name: "Jare Guo",
    email: "blabla@gmail.com",
    'zip code': 12345,
    isInvited: true
}
```

在对象声明的语法（`myProfile = {...}`）之中，有一组用逗号相隔的键值对。每一对都包括一个 key（字符串类型，有时候会用双引号包裹）和一个 value（可以是任何类型：包括 string，number，boolean，变量名，数组，对象甚至是函数）。我们管这样的一对键值叫做对象的属性（property），key 是属性名，value 是属性值。

你可以在 value 中嵌套其他对象，或者由一组对象组成的数组：

```js
myProfile = {
    name: "Jare Guo",
    email: "blabla@gmail.com",
    city: "Xiamen",
    points: 1234,
    isInvited: true,
    friends: [
        {
            name: "Johnny",
            email: "blablabla@gmail.com"
        },
        {
            name: "Nantas",
            email: "piapiapia@gmail.com"
        }
    ]
}
```

访问对象的某个属性非常简单，我们只要使用 dot 语法就可以了，还可以和数组成员的访问结合起来：

```js
myProfile.name; // Jare Guo
myProfile.friends[1].name; // Nantas
```

JavaScript 中的对象无处不在，在函数的参数传递中也会大量使用，比如在 Cocos Creator 中，我们就可以像这样定义 FireClass 对象：

```js
var MyComponent = cc.Class({
    extends: cc.Component
});
```

`{extends: cc.Component}` 这就是一个用做函数参数的对象。在 JavaScript 中大多数情况我们使用对象时都不一定要为他命名，很可能会像这样直接使用。

## 匿名函数

我们之前试过了用变量声明的语法来定义函数：

```js
myFunction = function (myArgument) {
    // do something
}
```

再复习一下将函数作为参数传入其他函数调用中的用法：

```js
square = function (a) {
    return a * a;
}
applyOperation = function (f, a) {
    return f(a);
}
applyOperation(square, 10); // 100
```

我们还见识了 JavaScript 的语法是多么喜欢偷懒，所以我们就可以用这样的方式代替上面的多个函数声明：

```js
applyOperation = function (f, a) {
    return f(a);
}
applyOperation(
    function(a){
      return a*a;
    },
    10
) // 100
```

我们这次并没有声明 `square` 函数，并将 `square` 作为参数传递，而是在参数的位置直接写了一个新的函数体，这样的做法被称为匿名函数，在 JavaScript 中是最为广泛使用的模式。

## 链式语法

下面我们介绍一种在数组和字符串操作中常用的语法：

```js
var myArray = [123, 456];
myArray.push(789) // 123, 456, 789
var myString = "abcdef";
myString.replace("a", "z"); // "zbcdef"
```

上面代码中的点符号表示“调用 `myString` 字符串对象的 `replace` 函数，并且传递 `a` 和 `z` 作为参数，然后获得返回值。

使用点符号的表达式，最大的优点是你可以把多项任务链接在一个表达式里，当然前提是每个调用的函数必须有合适的返回值。我们不会过多介绍如何定义可链接的函数，但是使用它们是非常简单的，只要使用以下的模式：`something.function1().function2().function3()`

链条中的每个环节都会接到一个初始值，调用一个函数，然后把函数执行结果传递到下一环节：

```js
var n = 5;
n.double().square(); // 100
```

## This

`this` 可能是 JavaScript 中最难以理解和掌握的概念了。

简单地说，`this` 关键字能让你访问正在处理的对象：就像变色龙一样，`this` 也会随着执行环境的变化而变化。

解释 `this` 的原理是很复杂的，不妨让我们使用两种工具来帮助我们在实践中理解 `this` 的值：

首先是最普通又最常用的 `console.log()`，它能够将对象的信息输出到浏览器的控制台里。在每个函数体开始的地方加入一个 `console.log()`，确保我们了解当时运行环境下正在处理的对象是什么。

```js
myFunction = function (a, b) {
    console.log(this);
    // do something
}
```

另外一个方法是将 `this` 赋值给另外一个变量：

```js
myFunction = function (a, b) {
    var myObject = this;
    // do something
}
```

乍一看好像这样子并没有什么作用，实际上它允许你安全的使用 `myObject` 这个变量来指代最初执行函数的对象，而不用担心在后面的代码中 `this` 会变成其他东西。

## this的值到底是什么？

```js
var obj = {
  foo: function(){
    console.log(this)
  }
}

var bar = obj.foo
obj.foo() // 打印出的 this 是 obj
bar() // 打印出的 this 是 window
```

请解释最后两行函数的值为什么不一样。

## 函数调用

首先需要从函数的调用开始讲起。

JS（ES5）里面有三种函数调用形式：

```js
func(p1, p2) 
obj.child.method(p1, p2)
func.call(context, p1, p2) // 先不讲 apply
```

一般，初学者都知道前两种形式，而且认为前两种形式「优于」第三种形式。

从看到这篇文章起，你一定要记住，第三种调用形式，才是正常调用形式：

```text
func.call(context, p1, p2)
```

其他两种都是语法糖，可以等价地变为 call 形式：

```js
func(p1, p2) 等价于
func.call(undefined, p1, p2)

obj.child.method(p1, p2) 等价于
obj.child.method.call(obj.child, p1, p2)
```

请记下来。（我们称此代码为「转换代码」，方便下文引用）

至此我们的函数调用只有一种形式：

```text
func.call(context, p1, p2)
```

## 这样，this 就好解释了

this，就是上面代码中的 context。就这么简单。

this 是你 call 一个函数时传的 context，由于你从来不用 call 形式的函数调用，所以你一直不知道。

**先看 func(p1, p2) 中的 this 如何确定：**

当你写下面代码时

```text
function func(){
  console.log(this)
}

func()
```

用「转换代码」把它转化一下，得到

```text
function func(){
  console.log(this)
}

func.call(undefined) // 可以简写为 func.call()
```

按理说打印出来的 this 应该就是 undefined 了吧，但是浏览器里有一条规则：

> 如果你传的 context 是 null 或 undefined，那么 window 对象就是默认的 context（严格模式下默认 context 是 undefined）

因此上面的打印结果是 window。

如果你希望这里的 this 不是 window，很简单：

```text
func.call(obj) // 那么里面的 this 就是 obj 对象了
```

**再看 obj.child.method(p1, p2) 的 this 如何确定**

```text
var obj = {
  foo: function(){
    console.log(this)
  }
}

obj.foo() 
```

按照「转换代码」，我们将 obj.foo() 转换为

```text
obj.foo.call(obj)
```

好了，this 就是 obj。搞定。

回到题目：

```text
var obj = {
  foo: function(){
    console.log(this)
  }
}

var bar = obj.foo
obj.foo() // 转换为 obj.foo.call(obj)，this 就是 obj
bar() 
// 转换为 bar.call()
// 由于没有传 context
// 所以 this 就是 undefined
// 最后浏览器给你一个默认的 this —— window 对象
```

## [ ] 语法

```js
function fn (){ console.log(this) }
var arr = [fn, fn2]
arr[0]() // 这里面的 this 又是什么呢？
```

我们可以把 arr[0]( ) 想象为arr.0( )，虽然后者的语法错了，但是形式与转换代码里的 obj.child.method(p1, p2) 对应上了，于是就可以愉快的转换了：

```js
        arr[0]() 
假想为    arr.0()
然后转换为 arr.0.call(arr)
那么里面的 this 就是 arr 了 :)
```

## **箭头函数**

我不明白为什么需要讨论箭头函数，实际上箭头函数里并没有 this，如果你在箭头函数里看到 this，你直接把它当作箭头函数外面的 this 即可。外面的 this 是什么，箭头函数里面的 this 就还是什么，因为箭头函数本身不支持 this。

有人说「箭头函数里面的 this 指向箭头函数外面的 this」，**这很傻**，因为箭头函数内外 this 就是同一个东西，并不存在什么指向不指向。

## 总结

1. this 就是你 call 一个函数时，传入的第一个参数。（请务必背下来「this 就是 call 的第一个参数」）
2. 如果你的函数调用形式不是 call 形式，请按照「转换代码」将其转换为 call 形式。
3. 因为 bind 本质就是自动在你调用一个函数的时候，把 context 换掉而已。

## JavaScript Resources

以下是 JavaScript 的一些入门教程:

- [JavaScript 标准参考教程](http://javascript.ruanyifeng.com/)
- [JavaScript 秘密花园](http://bonsaiden.github.io/JavaScript-Garden/zh/)
- [JavaScript 内存详解 & 分析指南](https://mp.weixin.qq.com/s/EuJzQajlU8rpZprWkXbJVg)

