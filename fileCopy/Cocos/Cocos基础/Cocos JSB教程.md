# JSB教程

[toc]

## 抽象层

### 架构

![img](../image/Cocos JSB教程/JSB2.0-Architecture.png)

### 宏（Macro）

抽象层必然会比直接使用 JS 引擎 API 的方式多占用一些 CPU 执行时间，如何把抽象层本身的开销降到最低成为设计的第一目标。

JS 绑定的大部分工作其实就是设定 JS 相关操作的 CPP 回调，在回调函数中关联 CPP 对象。其实主要包含如下两种类型：

- 注册 JS 函数（包含全局函数，类构造函数、类析构函数、类成员函数，类静态成员函数），绑定一个 CPP 回调
- 注册 JS 对象的属性读写访问器，分别绑定读与写的 CPP 回调

如何做到抽象层开销最小而且暴露统一的 API 供上层使用？

以注册 JS 函数的回调定义为例，JavaScriptCore、SpiderMonkey、V8、ChakraCore 的定义各不相同，具体如下：

- JavaScriptCore

  ```c++
    JSValueRef JSB_foo_func(
            JSContextRef _cx,
            JSObjectRef _function,
            JSObjectRef _thisObject,
            size_t argc,
            const JSValueRef _argv[],
            JSValueRef* _exception
        );
  ```

- SpiderMonkey

  ```c++
    bool JSB_foo_func(
            JSContext* _cx,
            unsigned argc,
            JS::Value* _vp
        );
  ```

- V8

  ```c++
    void JSB_foo_func(
            const v8::FunctionCallbackInfo<v8::Value>& v8args
        );
  ```

- ChakraCore

  ```c++
    JsValueRef JSB_foo_func(
            JsValueRef _callee,
            bool _isConstructCall,
            JsValueRef* _argv,
            unsigned short argc,
            void* _callbackState
        );
  ```

我们评估了几种方案，最终确定使用 `宏` 来抹平不同 JS 引擎回调函数定义与参数类型的不同，不管底层是使用什么引擎，开发者统一使用一种回调函数的定义。我们借鉴了 lua 的回调函数定义方式，抽象层所有的 JS 到 CPP 的回调函数的定义为：

```c++
bool foo(se::State& s)
{
    ...
    ...
}
SE_BIND_FUNC(foo) // 此处以回调函数的定义为例
```

开发者编写完回调函数后，记住使用 `SE_BIND_XXX` 系列的宏对回调函数进行包装。目前提供了如下几个宏：

- **SE_BIND_PROP_GET**：包装一个 JS 对象属性读取的回调函数
- **SE_BIND_PROP_SET**：包装一个 JS 对象属性写入的回调函数
- **SE_BIND_FUNC**：包装一个 JS 函数，可用于全局函数、类成员函数、类静态函数
- **SE_DECLARE_FUNC**：声明一个 JS 函数，一般在 `.h` 头文件中使用
- **SE_BIND_CTOR**：包装一个 JS 构造函数
- **SE_BIND_SUB_CLS_CTOR**：包装一个 JS 子类的构造函数，此子类使用 `cc.Class.extend` 继承 Native 绑定类
- **SE_FINALIZE_FUNC**：包装一个 JS 对象被 GC 回收后的回调函数
- **SE_DECLARE_FINALIZE_FUNC**：声明一个 JS 对象被 GC 回收后的回调函数
- **_SE**：包装回调函数的名称，转义为每个 JS 引擎能够识别的回调函数的定义，注意，第一个字符为下划线，类似 Windows 下用的 `_T("xxx")` 来包装 Unicode 或者 MultiBytes 字符串

## API

### CPP 命名空间（namespace）

CPP 抽象层所有的类型都在 `se` 命名空间下，其为 ScriptEngine 的缩写。

### 类型

#### se::ScriptEngine

`se::ScriptEngine` 为 JS 引擎的管理员，掌管 JS 引擎初始化、销毁、重启、Native 模块注册、加载脚本、强制垃圾回收、JS 异常清理、是否启用调试器。 它是一个单例，可通过 `se::ScriptEngine::getInstance()` 得到对应的实例。

#### se::Value

`se::Value` 可以被理解为 JS 变量在 CPP 层的引用。JS 变量有 `object`、`number`、`string`、`boolean`、`null`、`undefined` 六种类型。因此 `se::Value` 使用 `union` 包含 `object`、`number`、`string`、`boolean` 4 种 **有值类型**。**无值类型** 包含 `null` 和 `undefined`，可由 `_type` 直接表示。

```c++
namespace se {
    class Value {
        enum class Type : char
        {
            Undefined = 0,
            Null,
            Number,
            Boolean,
            String,
            Object
        };
        ...
        ...
    private:
        union {
            bool _boolean;
            double _number;
            std::string* _string;
            Object* _object;
        } _u;

        Type _type;
        ...
        ...
    };
}
```

如果 `se::Value` 中保存基础数据类型，比如 `number`、`string` 和 `boolean`，其内部是直接存储一份值副本。
`object` 的存储比较特殊，是通过 `se::Object*` 对 JS 对象的弱引用 (weak reference)。

#### se::Object

`se::Object` 继承于 `se::RefCounter` 引用计数管理类。目前抽象层中只有 `se::Object` 继承于 `se::RefCounter`。

上一小节我们说到，`se::Object` 是保存了对 JS 对象的弱引用，这里笔者有必要解释一下为什么是弱引用。

- 原因一：JS 对象控制 CPP 对象的生命周期的需要

  当在脚本层中通过 `var sp = new cc.Sprite("a.png");` 创建了一个 Sprite 后，在构造回调函数绑定中我们会创建一个 se::Object 并保留在一个全局的 map (NativePtrToObjectMap) 中，此 map 用于查询 `cocos2d::Sprite*` 指针获取对应的 JS 对象 `se::Object*`。

  ```c++
    static bool js_cocos2d_Sprite_finalize(se::State& s)
    {
        CCLOG("jsbindings: finalizing JS object %p (cocos2d::Sprite)", s.nativeThisObject());
        cocos2d::Sprite* cobj = (cocos2d::Sprite*)s.nativeThisObject();
        if (cobj->getReferenceCount() == 1)
            cobj->autorelease();
        else
            cobj->release();
        return true;
    }
    SE_BIND_FINALIZE_FUNC(js_cocos2d_Sprite_finalize)
  
    static bool js_cocos2dx_Sprite_constructor(se::State& s)
    {
        cocos2d::Sprite* cobj = new (std::nothrow) cocos2d::Sprite(); // cobj 将在 finalize 函数中被释放
        s.thisObject()->setPrivateData(cobj); // setPrivateData 内部会去保存 cobj 到 NativePtrToObjectMap 中
        return true;
    }
    SE_BIND_CTOR(js_cocos2dx_Sprite_constructor, __jsb_cocos2d_Sprite_class, js_cocos2d_Sprite_finalize)
  ```

- 设想如果强制要求 se::Object 为 JS 对象的强引用(strong reference)，即让 JS 对象不受 GC 控制，由于 se::Object 一直存在于 map 中，finalize 回调将永远无法被触发，从而导致内存泄露。

  正是由于 se::Object 保存的是 JS 对象的弱引用，JS 对象控制 CPP 对象的生命周期才能够实现。以上代码中，当 JS 对象被释放后，会触发 finalize 回调，开发者只需要在 `js_cocos2d_Sprite_finalize` 中释放对应的 c++ 对象即可，se::Object 的释放已经被包含在 `SE_BIND_FINALIZE_FUNC` 宏中自动处理，开发者无需管理在`JS 对象控制 CPP 对象`模式中 se::Object 的释放，但是在 `CPP 对象控制 JS 对象` 模式中，开发者需要管理对 se::Object 的释放，具体下一节中会举例说明。

- 原因二：更加灵活，手动调用 root 方法以支持强引用

  se::Object 中提供了 root/unroot 方法供开发者调用，root 会把 JS 对象放入到不受 GC 扫描到的区域，调用 root 后，se::Object 就强引用了 JS 对象，只有当 unroot 被调用，或者 se::Object 被释放后，JS 对象才会放回到受 GC 扫描到的区域。

  一般情况下，如果对象是非 `cocos2d::Ref` 的子类，会采用 CPP 对象控制 JS 对象的生命周期的方式去绑定。引擎内 spine、dragonbones、box2d、anysdk 等第三方库的绑定就是采用此方式。当 CPP 对象被释放的时候，需要在 NativePtrToObjectMap 中查找对应的 se::Object，然后手动 unroot 和 decRef。以 spine 中 spTrackEntry 的绑定为例：

  ```c++
    spTrackEntry_setDisposeCallback([](spTrackEntry* entry){
        // spTrackEntry 的销毁回调
        se::Object* seObj = nullptr;
  
        auto iter = se::NativePtrToObjectMap::find(entry);
        if (iter != se::NativePtrToObjectMap::end())
        {
            // 保存 se::Object 指针，用于在下面的 cleanup 函数中释放其内存
            seObj = iter->second;
            // Native 对象 entry 的内存已经被释放，因此需要立马解除 Native 对象与 JS 对象的关联。
            // 如果解除引用关系放在下面的 cleanup 函数中处理，有可能触发 se::Object::setPrivateData 中
            // 的断言，因为新生成的 Native 对象的地址可能与当前对象相同，而 cleanup 可能被延迟到帧结束前执行。
            se::NativePtrToObjectMap::erase(iter);
        }
        else
        {
            return;
        }
  
        auto cleanup = [seObj](){
            auto se = se::ScriptEngine::getInstance();
            if (!se->isValid() || se->isInCleanup())
                return;
  
            se::AutoHandleScope hs;
            se->clearException();
  
            // 由于上面逻辑已经把映射关系解除了，这里传入 false 表示不用再次解除映射关系,
            // 因为当前 seObj 的 private data 可能已经是另外一个不同的对象
            seObj->clearPrivateData(false);
            seObj->unroot(); // unroot，使 JS 对象受 GC 管理
            seObj->decRef(); // 释放 se::Object
        };
  
        // 确保不在垃圾回收中去操作 JS 引擎的 API
        if (!se::ScriptEngine::getInstance()->isGarbageCollecting())
        {
            cleanup();
        }
        else
        { // 如果在垃圾回收，把清理任务放在帧结束中进行
            CleanupTask::pushTaskToAutoReleasePool(cleanup);
        }
    });
  ```

**对象类型**

绑定对象的创建已经被隐藏在对应的 `SE_BIND_CTOR` 和 `SE_BIND_SUB_CLS_CTOR` 函数中，开发者在绑定回调中如果需要用到当前对象对应的 se::Object，只需要通过 s.thisObject() 即可获取。其中 s 为 se::State 类型，具体会在后续章节中说明。

此外，se::Object 目前支持以下几种对象的手动创建：

- Plain Object：通过 se::Object::createPlainObject 创建，类似 JS 中的 `var a = {};`
- Array Object：通过 se::Object::createArrayObject 创建，类似 JS 中的 `var a = [];`
- Uint8 Typed Array Object：通过 se::Object::createTypedArray 创建，类似 JS 中的 `var a = new Uint8Array(buffer);`
- Array Buffer Object：通过 se::Object::createArrayBufferObject，类似 JS 中的 `var a = new ArrayBuffer(len);`

**手动创建对象的释放**

se::Object::createXXX 方法与 cocos2d-x 中的 create 方法不同，抽象层是完全独立的一个模块，并不依赖与 cocos2d-x 的 autorelease 机制。虽然 se::Object 也是继承引用计数类，但开发者需要处理 **手动创建出来的对象** 的释放。

```c++
se::Object* obj = se::Object::createPlainObject();
...
...
obj->decRef(); // 释放引用，避免内存泄露
```

#### se::HandleObject（推荐的管理手动创建对象的辅助类）

- 在比较复杂的逻辑中使用手动创建对象，开发者往往会忘记在不同的逻辑中处理 decRef

  ```c++
    bool foo()
    {
        se::Object* obj = se::Object::createPlainObject();
        if (var1)
            return false; // 这里直接返回了，忘记做 decRef 释放操作
  
        if (var2)
            return false; // 这里直接返回了，忘记做 decRef 释放操作
        ...
        ...
        obj->decRef();
        return true;
    }
  ```

  就算在不同的返回条件分支中加上了 decRef 也会导致逻辑复杂，难以维护，如果后期加入另外一个返回分支，很容易忘记 decRef。

- JS 引擎在 se::Object::createXXX 后，如果由于某种原因 JS 引擎做了 GC 操作，导致后续使用的 se::Object 内部引用了一个非法指针，引发程序崩溃

为了解决上述两个问题，抽象层定义了一个辅助管理 **手动创建对象** 的类型，即 `se::HandleObject`。

`se::HandleObject` 是一个辅助类，用于更加简单地管理手动创建的 se::Object 对象的释放、root 和 unroot 操作。 以下两种代码写法是等价的，使用 se::HandleObject 的代码量明显少很多，而且更加安全。

```c++
{
    se::HandleObject obj(se::Object::createPlainObject());
    obj->setProperty(...);
    otherObject->setProperty("foo", se::Value(obj));
}
```

等价于：

```c++
{
    se::Object* obj = se::Object::createPlainObject();
    obj->root(); // 在手动创建完对象后立马 root，防止对象被 GC

    obj->setProperty(...);
    otherObject->setProperty("foo", se::Value(obj));

    obj->unroot(); // 当对象被使用完后，调用 unroot
    obj->decRef(); // 引用计数减一，避免内存泄露
}
```

> **注意**：
>
> 1. 不要尝试使用 `se::HandleObject` 创建一个 native 与 JS 的绑定对象，在 JS 控制 CPP 的模式中，绑定对象的释放会被抽象层自动处理，在 CPP 控制 JS 的模式中，前一章节中已经有描述了。
> 2. `se::HandleObject` 对象只能够在栈上被分配，而且栈上构造的时候必须传入一个 `se::Object` 指针。

#### se::Class

`se::Class` 用于暴露 CPP 类到 JS 中，它会在 JS 中创建一个对应名称的 `constructor function`。

它有如下方法：

- `static se::Class* create(className, obj, parentProto, ctor)`：创建一个 Class，注册成功后，在 JS 层中可以通过`var xxx = new SomeClass();`的方式创建一个对象
- `bool defineFunction(name, func)`：定义 Class 中的成员函数
- `bool defineProperty(name, getter, setter)`：定义 Class 属性读写器
- `bool defineStaticFunction(name, func)`：定义 Class 的静态成员函数，可通过 `SomeClass.foo()` 这种非 new 的方式访问，与类实例对象无关
- `bool defineStaticProperty(name, getter, setter)`：定义 Class 的静态属性读写器，可通过 SomeClass.propertyA 直接读写，与类实例对象无关
- `bool defineFinalizeFunction(func)`：定义 JS 对象被 GC 后的 CPP 回调
- `bool install()`：注册此类到 JS 虚拟机中
- `Object* getProto()`：获取注册到 JS 中的类（其实是 JS 的 constructor）的 prototype 对象，类似 `function Foo(){}` 的 `Foo.prototype`
- `const char* getName() const`：获取当前 Class 的名称

> **注意**：Class 类型创建后，不需要手动释放内存，它会被封装层自动处理。

更具体的 API 说明可以翻看 API 文档或者代码注释

#### se::AutoHandleScope

se::AutoHandleScope 对象类型完全是为了解决 V8 的兼容问题而引入的概念。

V8 中，当有 CPP 函数中需要触发 JS 相关操作，比如调用 JS 函数，访问 JS 属性等任何调用 v8::Local<> 的操作，V8 强制要求在调用这些操作前必须存在一个 v8::HandleScope 作用域，否则会引发程序崩溃。

因此抽象层中引入了 se::AutoHandleScope 的概念，其只在 V8 上有实现，其他 JS 引擎目前都只是空实现。

开发者需要记住，在任何代码执行中，需要调用 JS 的逻辑前，声明一个 se::AutoHandleScope 即可，比如：

```c++
class SomeClass {
    void update(float dt) {
        se::ScriptEngine::getInstance()->clearException();
        se::AutoHandleScope hs;

        se::Object* obj = ...;
        obj->setProperty(...);
        ...
        ...
        obj->call(...);
    }
};
```

#### se::State

之前章节我们有提及 State 类型，它是绑定回调中的一个环境，我们通过 se::State 可以取得当前的 CPP 指针、se::Object 对象指针、参数列表、返回值引用。

```c++
bool foo(se::State& s)
{
    // 获取 native 对象指针
    SomeClass* cobj = (SomeClass*)s.nativeThisObject();
    // 获取 se::Object 对象指针
    se::Object* thisObject = s.thisObject();
    // 获取参数列表
    const se::ValueArray& args = s.args();
    // 设置返回值
    s.rval().setInt32(100);
    return true;
}
SE_BIND_FUNC(foo)
```

## 抽象层依赖 Cocos 引擎么？

不依赖。

ScriptEngine 这层设计之初就将其定义为一个独立模块，完全不依赖 Cocos 引擎。开发者完整可以通过 copy、paste 把 `cocos/scripting/js-bindings/jswrapper` 下的所有抽象层源码拷贝到其他项目中直接使用。

## 手动绑定

### 回调函数声明

```c++
static bool Foo_balabala(se::State& s)
{
    const auto& args = s.args();
    int argc = (int)args.size();

    if (argc >= 2) // 这里约定参数个数必须大于等于 2，否则抛出错误到 JS 层且返回 false
    {
        ...
        ...
        return true;
    }

    SE_REPORT_ERROR("wrong number of arguments: %d, was expecting %d", argc, 2);
    return false;
}

// 如果是绑定函数，则用 SE_BIND_FUNC，构造函数、析构函数、子类构造函数等类似
SE_BIND_FUNC(Foo_balabala)
```

### 为 JS 对象设置一个属性值

```c++
se::Object* globalObj = se::ScriptEngine::getInstance()->getGlobalObject(); // 这里为了演示方便，获取全局对象
globalObj->setProperty("foo", se::Value(100)); // 给全局对象设置一个 foo 属性，值为 100
```

在 JS 中就可以直接使用 foo 这个全局变量了

```js
cc.log("foo value: " + foo); // 打印出 foo value: 100
```

### 为 JS 对象定义一个属性读写回调

```c++
// 全局对象的 foo 属性的读回调
static bool Global_get_foo(se::State& s)
{
    NativeObj* cobj = (NativeObj*)s.nativeThisObject();
    int32_t ret = cobj->getValue();
    s.rval().setInt32(ret);
    return true;
}
SE_BIND_PROP_GET(Global_get_foo)

// 全局对象的 foo 属性的写回调
static bool Global_set_foo(se::State& s)
{
    const auto& args = s.args();
    int argc = (int)args.size();
    if (argc >= 1)
    {
        NativeObj* cobj = (NativeObj*)s.nativeThisObject();
        int32_t arg1 = args[0].toInt32();
        cobj->setValue(arg1);
        // void 类型的函数，无需设置 s.rval，未设置默认返回 undefined 给 JS 层
        return true;
    }

    SE_REPORT_ERROR("wrong number of arguments: %d, was expecting %d", argc, 1);
    return false;
}
SE_BIND_PROP_SET(Global_set_foo)

void some_func()
{
    se::Object* globalObj = se::ScriptEngine::getInstance()->getGlobalObject(); // 这里为了演示方便，获取全局对象
    globalObj->defineProperty("foo", _SE(Global_get_foo), _SE(Global_set_foo)); // 使用_SE 宏包装一下具体的函数名称
}
```

### 为 JS 对象设置一个函数

```c++
static bool Foo_function(se::State& s)
{
    ...
    ...
}
SE_BIND_FUNC(Foo_function)

void some_func()
{
    se::Object* globalObj = se::ScriptEngine::getInstance()->getGlobalObject(); // 这里为了演示方便，获取全局对象
    globalObj->defineFunction("foo", _SE(Foo_function)); // 使用_SE 宏包装一下具体的函数名称
}
```

### 注册一个 CPP 类到 JS 虚拟机中

```c++
static se::Object* __jsb_ns_SomeClass_proto = nullptr;
static se::Class* __jsb_ns_SomeClass_class = nullptr;

namespace ns {
    class SomeClass
    {
    public:
        SomeClass()
        : xxx(0)
        {}

        void foo() {
            printf("SomeClass::foo\n");

            Director::getInstance()->getScheduler()->schedule([this](float dt){
                static int counter = 0;
                ++counter;
                if (_cb != nullptr)
                    _cb(counter);
            }, this, 1.0f, CC_REPEAT_FOREVER, 0.0f, false, "iamkey");
        }

        static void static_func() {
            printf("SomeClass::static_func\n");
        }

        void setCallback(const std::function<void(int)>& cb) {
            _cb = cb;
            if (_cb != nullptr)
            {
                printf("setCallback(cb)\n");
            }
            else
            {
                printf("setCallback(nullptr)\n");
            }
        }

        int xxx;
    private:
        std::function<void(int)> _cb;
    };
} // namespace ns {

static bool js_SomeClass_finalize(se::State& s)
{
    ns::SomeClass* cobj = (ns::SomeClass*)s.nativeThisObject();
    delete cobj;
    return true;
}
SE_BIND_FINALIZE_FUNC(js_SomeClass_finalize)

static bool js_SomeClass_constructor(se::State& s)
{
    ns::SomeClass* cobj = new ns::SomeClass();
    s.thisObject()->setPrivateData(cobj);
    return true;
}
SE_BIND_CTOR(js_SomeClass_constructor, __jsb_ns_SomeClass_class, js_SomeClass_finalize)

static bool js_SomeClass_foo(se::State& s)
{
    ns::SomeClass* cobj = (ns::SomeClass*)s.nativeThisObject();
    cobj->foo();
    return true;
}
SE_BIND_FUNC(js_SomeClass_foo)

static bool js_SomeClass_get_xxx(se::State& s)
{
    ns::SomeClass* cobj = (ns::SomeClass*)s.nativeThisObject();
    s.rval().setInt32(cobj->xxx);
    return true;
}
SE_BIND_PROP_GET(js_SomeClass_get_xxx)

static bool js_SomeClass_set_xxx(se::State& s)
{
    const auto& args = s.args();
    int argc = (int)args.size();
    if (argc > 0)
    {
        ns::SomeClass* cobj = (ns::SomeClass*)s.nativeThisObject();
        cobj->xxx = args[0].toInt32();
        return true;
    }

    SE_REPORT_ERROR("wrong number of arguments: %d, was expecting %d", argc, 1);
    return false;
}
SE_BIND_PROP_SET(js_SomeClass_set_xxx)

static bool js_SomeClass_static_func(se::State& s)
{
    ns::SomeClass::static_func();
    return true;
}
SE_BIND_FUNC(js_SomeClass_static_func)

bool js_register_ns_SomeClass(se::Object* global)
{
    // 保证 namespace 对象存在
    se::Value nsVal;
    if (!global->getProperty("ns", &nsVal))
    {
        // 不存在则创建一个 JS 对象，相当于 var ns = {};
        se::HandleObject jsobj(se::Object::createPlainObject());
        nsVal.setObject(jsobj);

        // 将 ns 对象挂载到 global 对象中，名称为 ns
        global->setProperty("ns", nsVal);
    }
    se::Object* ns = nsVal.toObject();

    // 创建一个 Class 对象，开发者无需考虑 Class 对象的释放，其交由 ScriptEngine 内部自动处理
    auto cls = se::Class::create("SomeClass", ns, nullptr, _SE(js_SomeClass_constructor)); // 如果无构造函数，最后一个参数可传入 nullptr，则这个类在 JS 中无法被 new SomeClass()出来

    // 为这个 Class 对象定义成员函数、属性、静态函数、析构函数
    cls->defineFunction("foo", _SE(js_SomeClass_foo));
    cls->defineProperty("xxx", _SE(js_SomeClass_get_xxx), _SE(js_SomeClass_set_xxx));

    cls->defineFinalizeFunction(_SE(js_SomeClass_finalize));

    // 注册类型到 JS VirtualMachine 的操作
    cls->install();

    // JSBClassType 为 Cocos 引擎绑定层封装的类型注册的辅助函数，此函数不属于 ScriptEngine 这层
    JSBClassType::registerClass<ns::SomeClass>(cls);

    // 保存注册的结果，便于其他地方使用，比如类继承
    __jsb_ns_SomeClass_proto = cls->getProto();
    __jsb_ns_SomeClass_class = cls;

    // 为每个此 Class 实例化出来的对象附加一个属性
    __jsb_ns_SomeClass_proto->setProperty("yyy", se::Value("helloyyy"));

    // 注册静态成员变量和静态成员函数
    se::Value ctorVal;
    if (ns->getProperty("SomeClass", &ctorVal) && ctorVal.isObject())
    {
        ctorVal.toObject()->setProperty("static_val", se::Value(200));
        ctorVal.toObject()->defineFunction("static_func", _SE(js_SomeClass_static_func));
    }

    // 清空异常
    se::ScriptEngine::getInstance()->clearException();
    return true;
}
```

### 如何绑定 CPP 接口中的回调函数？

```c++
static bool js_SomeClass_setCallback(se::State& s)
{
    const auto& args = s.args();
    int argc = (int)args.size();
    if (argc >= 1)
    {
        ns::SomeClass* cobj = (ns::SomeClass*)s.nativeThisObject();

        se::Value jsFunc = args[0];
        se::Value jsTarget = argc > 1 ? args[1] : se::Value::Undefined;

        if (jsFunc.isNullOrUndefined())
        {
            cobj->setCallback(nullptr);
        }
        else
        {
            assert(jsFunc.isObject() && jsFunc.toObject()->isFunction());

            // 如果当前 SomeClass 是可以被 new 出来的类，我们 使用 se::Object::attachObject 把 jsFunc 和 jsTarget 关联到当前对象中
            s.thisObject()->attachObject(jsFunc.toObject());
            s.thisObject()->attachObject(jsTarget.toObject());

            // 如果当前 SomeClass 类是一个单例类，或者永远只有一个实例的类，我们不能用 se::Object::attachObject 去关联
            // 必须使用 se::Object::root，开发者无需关系 unroot，unroot 的操作会随着 lambda 的销毁触发 jsFunc 的析构，在 se::Object 的析构函数中进行 unroot 操作。
            // js_cocos2dx_EventDispatcher_addCustomEventListener 的绑定代码就是使用此方式，因为 EventDispatcher 始终只有一个实例，
            // 如果使用 s.thisObject->attachObject(jsFunc.toObject);会导致对应的 func 和 target 永远无法被释放，引发内存泄露。

            // jsFunc.toObject()->root();
            // jsTarget.toObject()->root();

            cobj->setCallback([jsFunc, jsTarget](int counter){

                // CPP 回调函数中要传递数据给 JS 或者调用 JS 函数，在回调函数开始需要添加如下两行代码。
                se::ScriptEngine::getInstance()->clearException();
                se::AutoHandleScope hs;

                se::ValueArray args;
                args.push_back(se::Value(counter));

                se::Object* target = jsTarget.isObject() ? jsTarget.toObject() : nullptr;
                jsFunc.toObject()->call(args, target);
            });
        }

        return true;
    }

    SE_REPORT_ERROR("wrong number of arguments: %d, was expecting %d", argc, 1);
    return false;
}
SE_BIND_FUNC(js_SomeClass_setCallback)
```

SomeClass 类注册后，就可以在 JS 中这样使用了：

```js
 var myObj = new ns.SomeClass();
 myObj.foo();
 ns.SomeClass.static_func();
 cc.log("ns.SomeClass.static_val: " + ns.SomeClass.static_val);
 cc.log("Old myObj.xxx:" + myObj.xxx);
 myObj.xxx = 1234;
 cc.log("New myObj.xxx:" + myObj.xxx);
 cc.log("myObj.yyy: " + myObj.yyy);

 var delegateObj = {
     onCallback: function(counter) {
         cc.log("Delegate obj, onCallback: " + counter + ", this.myVar: " + this.myVar);
         this.setVar();
     },

     setVar: function() {
         this.myVar++;
     },

     myVar: 100
 };

 myObj.setCallback(delegateObj.onCallback, delegateObj);

 setTimeout(function(){
    myObj.setCallback(null);
 }, 6000); // 6 秒后清空 callback
```

Console 中会输出：

```
SomeClass::foo
SomeClass::static_func
ns.SomeClass.static_val: 200
Old myObj.xxx:0
New myObj.xxx:1234
myObj.yyy: helloyyy
setCallback(cb)
Delegate obj, onCallback: 1, this.myVar: 100
Delegate obj, onCallback: 2, this.myVar: 101
Delegate obj, onCallback: 3, this.myVar: 102
Delegate obj, onCallback: 4, this.myVar: 103
Delegate obj, onCallback: 5, this.myVar: 104
Delegate obj, onCallback: 6, this.myVar: 105
setCallback(nullptr)
```

### 如何使用 cocos2d-x bindings 这层的类型转换辅助函数？

类型转换辅助函数位于`cocos/scripting/js-bindings/manual/jsb_conversions.hpp/.cpp`中，其包含：

#### se::Value 转换为 C++ 类型

```c++
bool seval_to_int32(const se::Value& v, int32_t* ret);
bool seval_to_uint32(const se::Value& v, uint32_t* ret);
bool seval_to_int8(const se::Value& v, int8_t* ret);
bool seval_to_uint8(const se::Value& v, uint8_t* ret);
bool seval_to_int16(const se::Value& v, int16_t* ret);
bool seval_to_uint16(const se::Value& v, uint16_t* ret);
bool seval_to_boolean(const se::Value& v, bool* ret);
bool seval_to_float(const se::Value& v, float* ret);
bool seval_to_double(const se::Value& v, double* ret);
bool seval_to_long(const se::Value& v, long* ret);
bool seval_to_ulong(const se::Value& v, unsigned long* ret);
bool seval_to_longlong(const se::Value& v, long long* ret);
bool seval_to_ssize(const se::Value& v, ssize_t* ret);
bool seval_to_std_string(const se::Value& v, std::string* ret);
bool seval_to_Vec2(const se::Value& v, cocos2d::Vec2* pt);
bool seval_to_Vec3(const se::Value& v, cocos2d::Vec3* pt);
bool seval_to_Vec4(const se::Value& v, cocos2d::Vec4* pt);
bool seval_to_Mat4(const se::Value& v, cocos2d::Mat4* mat);
bool seval_to_Size(const se::Value& v, cocos2d::Size* size);
bool seval_to_Rect(const se::Value& v, cocos2d::Rect* rect);
bool seval_to_Color3B(const se::Value& v, cocos2d::Color3B* color);
bool seval_to_Color4B(const se::Value& v, cocos2d::Color4B* color);
bool seval_to_Color4F(const se::Value& v, cocos2d::Color4F* color);
bool seval_to_ccvalue(const se::Value& v, cocos2d::Value* ret);
bool seval_to_ccvaluemap(const se::Value& v, cocos2d::ValueMap* ret);
bool seval_to_ccvaluemapintkey(const se::Value& v, cocos2d::ValueMapIntKey* ret);
bool seval_to_ccvaluevector(const se::Value& v, cocos2d::ValueVector* ret);
bool sevals_variadic_to_ccvaluevector(const se::ValueArray& args, cocos2d::ValueVector* ret);
bool seval_to_blendfunc(const se::Value& v, cocos2d::BlendFunc* ret);
bool seval_to_std_vector_string(const se::Value& v, std::vector<std::string>* ret);
bool seval_to_std_vector_int(const se::Value& v, std::vector<int>* ret);
bool seval_to_std_vector_float(const se::Value& v, std::vector<float>* ret);
bool seval_to_std_vector_Vec2(const se::Value& v, std::vector<cocos2d::Vec2>* ret);
bool seval_to_std_vector_Touch(const se::Value& v, std::vector<cocos2d::Touch*>* ret);
bool seval_to_std_map_string_string(const se::Value& v, std::map<std::string, std::string>* ret);
bool seval_to_FontDefinition(const se::Value& v, cocos2d::FontDefinition* ret);
bool seval_to_Acceleration(const se::Value& v, cocos2d::Acceleration* ret);
bool seval_to_Quaternion(const se::Value& v, cocos2d::Quaternion* ret);
bool seval_to_AffineTransform(const se::Value& v, cocos2d::AffineTransform* ret);
//bool seval_to_Viewport(const se::Value& v, cocos2d::experimental::Viewport* ret);
bool seval_to_Data(const se::Value& v, cocos2d::Data* ret);
bool seval_to_DownloaderHints(const se::Value& v, cocos2d::network::DownloaderHints* ret);
bool seval_to_TTFConfig(const se::Value& v, cocos2d::TTFConfig* ret);

//box2d seval to native convertion
bool seval_to_b2Vec2(const se::Value& v, b2Vec2* ret);
bool seval_to_b2AABB(const se::Value& v, b2AABB* ret);

template<typename T>
bool seval_to_native_ptr(const se::Value& v, T* ret);

template<typename T>
bool seval_to_Vector(const se::Value& v, cocos2d::Vector<T>* ret);

template<typename T>
bool seval_to_Map_string_key(const se::Value& v, cocos2d::Map<std::string, T>* ret)
```

#### C++ 类型转换为 se::Value

```c++
bool int8_to_seval(int8_t v, se::Value* ret);
bool uint8_to_seval(uint8_t v, se::Value* ret);
bool int32_to_seval(int32_t v, se::Value* ret);
bool uint32_to_seval(uint32_t v, se::Value* ret);
bool int16_to_seval(uint16_t v, se::Value* ret);
bool uint16_to_seval(uint16_t v, se::Value* ret);
bool boolean_to_seval(bool v, se::Value* ret);
bool float_to_seval(float v, se::Value* ret);
bool double_to_seval(double v, se::Value* ret);
bool long_to_seval(long v, se::Value* ret);
bool ulong_to_seval(unsigned long v, se::Value* ret);
bool longlong_to_seval(long long v, se::Value* ret);
bool ssize_to_seval(ssize_t v, se::Value* ret);
bool std_string_to_seval(const std::string& v, se::Value* ret);

bool Vec2_to_seval(const cocos2d::Vec2& v, se::Value* ret);
bool Vec3_to_seval(const cocos2d::Vec3& v, se::Value* ret);
bool Vec4_to_seval(const cocos2d::Vec4& v, se::Value* ret);
bool Mat4_to_seval(const cocos2d::Mat4& v, se::Value* ret);
bool Size_to_seval(const cocos2d::Size& v, se::Value* ret);
bool Rect_to_seval(const cocos2d::Rect& v, se::Value* ret);
bool Color3B_to_seval(const cocos2d::Color3B& v, se::Value* ret);
bool Color4B_to_seval(const cocos2d::Color4B& v, se::Value* ret);
bool Color4F_to_seval(const cocos2d::Color4F& v, se::Value* ret);
bool ccvalue_to_seval(const cocos2d::Value& v, se::Value* ret);
bool ccvaluemap_to_seval(const cocos2d::ValueMap& v, se::Value* ret);
bool ccvaluemapintkey_to_seval(const cocos2d::ValueMapIntKey& v, se::Value* ret);
bool ccvaluevector_to_seval(const cocos2d::ValueVector& v, se::Value* ret);
bool blendfunc_to_seval(const cocos2d::BlendFunc& v, se::Value* ret);
bool std_vector_string_to_seval(const std::vector<std::string>& v, se::Value* ret);
bool std_vector_int_to_seval(const std::vector<int>& v, se::Value* ret);
bool std_vector_float_to_seval(const std::vector<float>& v, se::Value* ret);
bool std_vector_Touch_to_seval(const std::vector<cocos2d::Touch*>& v, se::Value* ret);
bool std_map_string_string_to_seval(const std::map<std::string, std::string>& v, se::Value* ret);
bool uniform_to_seval(const cocos2d::Uniform* v, se::Value* ret);
bool FontDefinition_to_seval(const cocos2d::FontDefinition& v, se::Value* ret);
bool Acceleration_to_seval(const cocos2d::Acceleration* v, se::Value* ret);
bool Quaternion_to_seval(const cocos2d::Quaternion& v, se::Value* ret);
bool ManifestAsset_to_seval(const cocos2d::extension::ManifestAsset& v, se::Value* ret);
bool AffineTransform_to_seval(const cocos2d::AffineTransform& v, se::Value* ret);
bool Data_to_seval(const cocos2d::Data& v, se::Value* ret);
bool DownloadTask_to_seval(const cocos2d::network::DownloadTask& v, se::Value* ret);

template<typename T>
bool Vector_to_seval(const cocos2d::Vector<T*>& v, se::Value* ret);

template<typename T>
bool Map_string_key_to_seval(const cocos2d::Map<std::string, T*>& v, se::Value* ret);

template<typename T>
bool native_ptr_to_seval(typename std::enable_if<!std::is_base_of<cocos2d::Ref,T>::value,T>::type* v, se::Value* ret, bool* isReturnCachedValue = nullptr);

template<typename T>
bool native_ptr_to_seval(typename std::enable_if<!std::is_base_of<cocos2d::Ref,T>::value,T>::type* v, se::Class* cls, se::Value* ret, bool* isReturnCachedValue = nullptr)

template<typename T>
bool native_ptr_to_seval(typename std::enable_if<std::is_base_of<cocos2d::Ref,T>::value,T>::type* v, se::Value* ret, bool* isReturnCachedValue = nullptr);

template<typename T>
bool native_ptr_to_seval(typename std::enable_if<std::is_base_of<cocos2d::Ref,T>::value,T>::type* v, se::Class* cls, se::Value* ret, bool* isReturnCachedValue = nullptr);

template<typename T>
bool native_ptr_to_rooted_seval(typename std::enable_if<!std::is_base_of<cocos2d::Ref,T>::value,T>::type* v, se::Value* ret, bool* isReturnCachedValue = nullptr);

template<typename T>
bool native_ptr_to_rooted_seval(typename std::enable_if<!std::is_base_of<cocos2d::Ref,T>::value,T>::type* v, se::Class* cls, se::Value* ret, bool* isReturnCachedValue = nullptr);


// Spine conversions
bool speventdata_to_seval(const spEventData& v, se::Value* ret);
bool spevent_to_seval(const spEvent& v, se::Value* ret);
bool spbonedata_to_seval(const spBoneData& v, se::Value* ret);
bool spbone_to_seval(const spBone& v, se::Value* ret);
bool spskeleton_to_seval(const spSkeleton& v, se::Value* ret);
bool spattachment_to_seval(const spAttachment& v, se::Value* ret);
bool spslotdata_to_seval(const spSlotData& v, se::Value* ret);
bool spslot_to_seval(const spSlot& v, se::Value* ret);
bool sptimeline_to_seval(const spTimeline& v, se::Value* ret);
bool spanimationstate_to_seval(const spAnimationState& v, se::Value* ret);
bool spanimation_to_seval(const spAnimation& v, se::Value* ret);
bool sptrackentry_to_seval(const spTrackEntry& v, se::Value* ret);

// Box2d
bool b2Vec2_to_seval(const b2Vec2& v, se::Value* ret);
bool b2Manifold_to_seval(const b2Manifold* v, se::Value* ret);
bool b2AABB_to_seval(const b2AABB& v, se::Value* ret);
```

辅助转换函数不属于 `Script Engine Wrapper` 抽象层，属于 cocos2d-x 绑定层，封装这些函数是为了在绑定代码中更加方便的转换。每个转换函数都返回 `bool` 类型，表示转换是否成功，开发者如果调用这些接口，需要去判断这个返回值。

以上接口，直接根据接口名称即可知道具体的用法，接口中第一个参数为输入，第二个参数为输出参数。用法如下：

```c++
se::Value v;
bool ok = int32_to_seval(100, &v); // 第二个参数为输出参数，传入输出参数的地址
int32_t v;
bool ok = seval_to_int32(args[0], &v); // 第二个参数为输出参数，传入输出参数的地址
```

#### （IMPORTANT）理解 native_ptr_to_seval 与 native_ptr_to_rooted_seval 的区别

**开发者一定要理解清楚这二者的区别，才不会因为误用导致 JS 层内存泄露这种比较难查的 bug。**

- `native_ptr_to_seval` 用于 `JS 控制 CPP 对象生命周期` 的模式。当在绑定层需要根据一个 CPP 对象指针获取一个 `se::Value` 的时候，可调用此方法。引擎内大部分继承于 `cocos2d::Ref` 的子类都采取这种方式去获取 `se::Value`。记住一点，当你管理的绑定对象是由 JS 控制生命周期，需要转换为 seval 的时候，请用此方法，否则考虑用 `native_ptr_to_rooted_seval`。
- `native_ptr_to_rooted_seval` 用于 `CPP 控制 JS 对象生命周期` 的模式。一般而言，第三方库中的对象绑定都会用到此方法。此方法会根据传入的 CPP 对象指针查找 cache 的 `se::Object`，如果不存在，则创建一个 rooted 的 `se::Object`，即这个创建出来的 JS 对象将不受 GC 控制，并永远在内存中。开发者需要监听 CPP 对象的释放，并在释放的时候去做 `se::Object` 的 unroot 操作，具体可参照前面章节中描述的 `spTrackEntry_setDisposeCallback` 中的内容。

更多关于手动绑定的内容可参考 [使用 JSB 手动绑定](https://docs.cocos.com/creator/manual/zh/advanced-topics/jsb-manual-binding.html)。

## 自动绑定

### 配置模块 ini 文件

配置方法与 1.6 中的方法相同，主要注意的是：1.7 中废弃了 `script_control_cpp`，因为 `script_control_cpp` 字段会影响到整个模块，如果模块中需要绑定 cocos2d::Ref 子类和非 cocos::Ref 子类，原来的绑定配置则无法满足需求。1.7 中取而代之的新字段为 `classes_owned_by_cpp`，表示哪些类是需要由 CPP 来控制 JS 对象的生命周期。

1.7 中另外加入的一个配置字段为 `persistent_classes`，用于表示哪些类是在游戏运行中一直存在的，比如：`SpriteFrameCache`、`FileUtils`、`EventDispatcher`、`ActionManager`、`Scheduler`。

其他字段与 1.6 一致。

具体可以参考引擎目录下的 `tools/tojs/cocos2dx.ini` 等 ini 配置。

### 理解 ini 文件中每个字段的意义

```bash
# 模块名称
[cocos2d-x] 

# 绑定回调函数的前缀，也是生成的自动绑定文件的前缀
prefix = cocos2dx

# 绑定的类挂载在 JS 中的哪个对象中，类似命名空间
target_namespace = cc

# 自动绑定工具基于 Android 编译环境，此处配置 Android 头文件搜索路径
android_headers = -I%(androidndkdir)s/platforms/android-14/arch-arm/usr/include -I%(androidndkdir)s/sources/cxx-stl/gnu-libstdc++/4.8/libs/armeabi-v7a/include -I%(androidndkdir)s/sources/cxx-stl/gnu-libstdc++/4.8/include -I%(androidndkdir)s/sources/cxx-stl/gnu-libstdc++/4.9/libs/armeabi-v7a/include -I%(androidndkdir)s/sources/cxx-stl/gnu-libstdc++/4.9/include

# 配置 Android 编译参数
android_flags = -D_SIZE_T_DEFINED_

# 配置 clang 头文件搜索路径
clang_headers = -I%(clangllvmdir)s/%(clang_include)s

# 配置 clang 编译参数
clang_flags = -nostdinc -x c++ -std=c++11 -U __SSE__

# 配置引擎的头文件搜索路径
cocos_headers = -I%(cocosdir)s/cocos -I%(cocosdir)s/cocos/platform/android -I%(cocosdir)s/external/sources

# 配置引擎编译参数
cocos_flags = -DANDROID

# 配置额外的编译参数
extra_arguments = %(android_headers)s %(clang_headers)s %(cxxgenerator_headers)s %(cocos_headers)s %(android_flags)s %(clang_flags)s %(cocos_flags)s %(extra_flags)s

# 需要自动绑定工具解析哪些头文件
headers = %(cocosdir)s/cocos/cocos2d.h %(cocosdir)s/cocos/scripting/js-bindings/manual/BaseJSAction.h

# 在生成的绑定代码中，重命名头文件
replace_headers=CCProtectedNode.h::2d/CCProtectedNode.h,CCAsyncTaskPool.h::base/CCAsyncTaskPool.h

# 需要绑定哪些类，可以使用正则表达式，以空格为间隔
classes = 

# 哪些类需要在 JS 层通过 cc.Class.extend，以空格为间隔
classes_need_extend = 

# 需要为哪些类绑定属性，以逗号为间隔
field = Acceleration::[x y z timestamp]

# 需要忽略绑定哪些类，以逗号为间隔
skip = AtlasNode::[getTextureAtlas],
       ParticleBatchNode::[getTextureAtlas],

# 重命名函数，以逗号为间隔
rename_functions = ComponentContainer::[get=getComponent],
                   LayerColor::[initWithColor=init],

# 重命名类，以逗号为间隔
rename_classes = SimpleAudioEngine::AudioEngine,
                 SAXParser::PlistParser,


# 配置哪些类不需要搜索其父类
classes_have_no_parents = Node Director SimpleAudioEngine FileUtils TMXMapInfo Application GLViewProtocol SAXParser Configuration

# 配置哪些父类需要被忽略
base_classes_to_skip = Ref Clonable

# 配置哪些类是抽象类，抽象类没有构造函数，即在 js 层无法通过 var a = new SomeClass();的方式构造 JS 对象
abstract_classes = Director SpriteFrameCache Set SimpleAudioEngine

# 配置哪些类是始终以一个实例的方式存在的，游戏运行过程中不会被销毁
persistent_classes = SpriteFrameCache FileUtils EventDispatcher ActionManager Scheduler

# 配置哪些类是需要由 CPP 对象来控制 JS 对象生命周期的，未配置的类，默认采用 JS 控制 CPP 对象生命周期
classes_owned_by_cpp =
```

更多关于自动绑定的内容可参考 [使用 JSB 自动绑定](https://docs.cocos.com/creator/manual/zh/advanced-topics/jsb-auto-binding.html)。

## 远程调试与 Profile

默认远程调试和 Profile 是在 debug 模式中生效的，如果需要在 release 模式下也启用，需要手动修改 cocos/scripting/js-bindings/jswrapper/config.hpp 中的宏开关。

```c++
#if defined(COCOS2D_DEBUG) && COCOS2D_DEBUG > 0
#define SE_ENABLE_INSPECTOR 1
#define SE_DEBUG 2
#else
#define SE_ENABLE_INSPECTOR 0
#define SE_DEBUG 0
#endif
```

改为：

```c++
#if 1 // 这里改为 1，强制启用调试
#define SE_ENABLE_INSPECTOR 1
#define SE_DEBUG 2
#else
#define SE_ENABLE_INSPECTOR 0
#define SE_DEBUG 0
#endif
```

### Chrome 远程调试 V8

#### Windows/Mac

- 编译、运行游戏（或在 Creator 中直接使用模拟器运行）

- 用 Chrome 浏览器打开 [devtools://devtools/bundled/js_app.html?v8only=true&ws=127.0.0.1:5086/00010002-0003-4004-8005-000600070008](devtools://devtools/bundled/js_app.html?v8only=true&ws=127.0.0.1:5086/00010002-0003-4004-8005-000600070008)。（若使用的是旧版 Chrome，则需要将地址开头的 `devtools` 改成 `chrome-devtools`）

- 断点调试：

  ![img](../image/Cocos JSB教程/v8-win32-debug.jpg)

- 抓取 JS Heap：

  ![img](../image/Cocos JSB教程/v8-win32-memory.jpg)

- Profile：

  ![img](../image/Cocos JSB教程/v8-win32-profile.jpg)

#### Android/iOS

- 保证 Android/iOS 设备与 PC 或者 Mac 在同一个局域网中
- 编译，运行游戏
- 用 Chrome 浏览器打开 [devtools://devtools/bundled/js_app.html?v8only=true&ws=xxx.xxx.xxx.xxx:6086/00010002-0003-4004-8005-000600070008](devtools://devtools/bundled/js_app.html?v8only=true&ws=xxx.xxx.xxx.xxx:6086/00010002-0003-4004-8005-000600070008)，其中 `xxx.xxx.xxx.xxx` 为局域网中 Android/iOS 设备的 IP 地址。（若使用的是旧版 Chrome，则需要将地址开头的 `devtools` 改成 `chrome-devtools`）
- 调试界面与 Windows 相同

## Q & A

### se::ScriptEngine 与 ScriptingCore 的区别，为什么还要保留 ScriptingCore?

在 1.7 中，抽象层被设计为一个与引擎没有关系的独立模块，对 JS 引擎的管理从 ScriptingCore 被移动到了 se::ScriptEngine 类中，ScriptingCore 被保留下来是希望通过它把引擎的一些事件传递给封装层，充当适配器的角色。

ScriptingCore 只需要在 AppDelegate 中被使用一次即可，之后的所有操作都只需要用到 se::ScriptEngine。

```c++
bool AppDelegate::applicationDidFinishLaunching()
{
    ...
    ...
    director->setAnimationInterval(1.0 / 60);

    // 这两行把 ScriptingCore 这个适配器设置给引擎，用于传递引擎的一些事件，
    // 比如 Node 的 onEnter, onExit, Action 的 update，JS 对象的持有与解除持有
    ScriptingCore* sc = ScriptingCore::getInstance();
    ScriptEngineManager::getInstance()->setScriptEngine(sc);

    se::ScriptEngine* se = se::ScriptEngine::getInstance();
    ...
    ...
}
```

### se::Object::root/unroot 与 se::Object::incRef/decRef 的区别?

root/unroot 用于控制 JS 对象是否受 GC 控制，root 表示不受 GC 控制，unroot 则相反，表示交由 GC 控制，对一个 se::Object 来说，root 和 unroot 可以被调用多次，se::Object 内部有_rootCount 变量用于表示 root 的次数。当 unroot 被调用，且_rootCount 为 0 时，se::Object 关联的 JS 对象将交由 GC 管理。还有一种情况，即如果 se::Object 的析构被触发了，如果_rootCount > 0，则强制把 JS 对象交由 GC 控制。

incRef/decRef 用于控制 se::Object 这个 `cpp` 对象的生命周期，前面章节已经提及，建议用户使用 se::HandleObject 来控制 **手动创建非绑定对象** 的方式控制 se::Object 的生命周期。因此，一般情况下，开发者不需要接触到 incRef/decRef。

### 对象生命周期的关联与解除关联

使用 `se::Object::attachObject` 关联对象的生命周期
使用 `se::Object::dettachObject` 解除对象的生命周期。

`objA->attachObject(objB);` 类似于 JS 中执行 `objA.__nativeRefs[index] = objB`，只有当 objA 被 GC 后，objB 才有可能被 GC
`objA->dettachObject(objB);` 类似于 JS 中执行 `delete objA.__nativeRefs[index];`，这样 objB 的生命周期就不受 objA 控制了

### cocos2d::Ref 子类与非 cocos2d::Ref 子类 JS/CPP 对象生命周期管理有何不同？

目前引擎中 cocos2d::Ref 子类的绑定采用 JS 对象控制 CPP 对象生命周期的方式，这样做的好处是，解决了一直以来被诟病的需要在 JS 层 retain，release 对象的烦恼。

非 cocos2d::Ref 子类采用 CPP 对象控制 JS 对象生命周期的方式。此方式要求，CPP 对象销毁后需要通知绑定层去调用对应 se::Object 的 clearPrivateData, unroot, decRef 的方法。JS 代码中一定要慎重操作对象，当有可能出现非法对象的逻辑中，使用 cc.sys.isObjectValid 来判断 CPP 对象是否被释放了。

### 绑定 cocos2d::Ref 子类的析构函数需要注意的事项

如果在 JS 对象的 finalize 回调中调用任何 JS 引擎的 API，可能导致崩溃。因为当前引擎正在进行垃圾回收的流程，无法被打断处理其他操作。 finalize 回调中是告诉 CPP 层是否对应的 CPP 对象的内存，不能在 CPP 对象的析构中又去操作 JS 引擎 API。

那如果必须调用，应该如何处理？

cocos2d-x 的绑定中，如果引用计数为 1 了，我们不使用 release，而是使用 autorelease 延时 CPP 类的析构到帧结束去执行。

```c++
static bool js_cocos2d_Sprite_finalize(se::State& s)
{
    CCLOG("jsbindings: finalizing JS object %p (cocos2d::Sprite)", s.nativeThisObject());
    cocos2d::Sprite* cobj = (cocos2d::Sprite*)s.nativeThisObject();
    if (cobj->getReferenceCount() == 1)
        cobj->autorelease();
    else
        cobj->release();
    return true;
}
SE_BIND_FINALIZE_FUNC(js_cocos2d_Sprite_finalize)
```

### 请不要在栈（Stack）上分配 cocos2d::Ref 的子类对象

Ref 的子类必须在堆（Heap）上分配，即通过 `new`，然后通过 `release` 来释放。当 JS 对象的 finalize 回调函数中统一使用 `autorelease` 或 `release` 来释放。如果是在栈上的对象，reference count 很有可能为 0，而这时调用 `release`，其内部会调用 `delete`，从而导致程序崩溃。所以为了防止这个行为的出现，开发者可以在继承于 cocos2d::Ref 的绑定类中，标识析构函数为 `protected` 或者 `private`，保证在编译阶段就能发现这个问题。

例如：

```c++
class CC_EX_DLL EventAssetsManagerEx : public cocos2d::EventCustom
{
public:
    ...
    ...
private:
    virtual ~EventAssetsManagerEx() {}
    ...
    ...
};

EventAssetsManagerEx event(...); // 编译阶段报错
dispatcher->dispatchEvent(&event);

// 必须改为

EventAssetsManagerEx* event = new EventAssetsManagerEx(...);
dispatcher->dispatchEvent(event);
event->release();
```

### 如何监听脚本错误

在 AppDelegate.cpp 中通过 se::ScriptEngine::getInstance()->setExceptionCallback(...)设置 JS 层异常回调。

```c++
bool AppDelegate::applicationDidFinishLaunching()
{
    ...
    ...
    se::ScriptEngine* se = se::ScriptEngine::getInstance();

    se->setExceptionCallback([](const char* location, const char* message, const char* stack){
        // Send exception information to server like Tencent Bugly.
        // ...
        // ...
    });

    jsb_register_all_modules();
    ...
    ...
    return true;
}
```

# 使用 JSB 手动绑定

## 背景

一直以来，ABCmouse 项目中的整体 JS/Native 通信调用结构都是基于 `callStaticMethod <-> evalString` 的方式。通过 `callStaticMethod` 方法我们可以通过反射机制直接在 JavaScript 中调用 `Java/Objective-C` 的静态方法。而通过 `evalString` 方式，则可以执行 JS 代码，这样便可以进行双端通信。

[![ ](../image/Cocos JSB教程/infrastructure.png)](https://docs.cocos.com/creator/manual/zh/advanced-topics/jsb/infrastructure.png)

新版 ABCmouse 的应用架构：基于 callStaticMethod 与 evalString 进行通信

虽然基于这个方式上层封装接口后，新增业务逻辑会比较方便。但是过度依赖 evalString，往往也会带来一些隐患。举个 Android 侧的例子：

```js
Cocos2dxJavascriptJavaBridge.evalString("window.sample.testEval('" + param + "',JSON.stringify(" + jsonObj + "))");
```

对于常见的参数结构，这样运行是没有问题的，然而基于实际场景的种种情况，我们会发现针对 **引号** 的控制格外重要。如代码所示，为了保证 JS 代码能够被正确执行，我们在拼接字符串时必须明确 `'` 与 `"` 的使用，稍有不慎就会出现 `evalString` 失败的情况。在 Cocos 的官方论坛上，从大量的反馈中我们也能了解这里的确是一个十分容易踩坑的地方。而另一方面，对于我们项目本身而言，过度依赖 `evalString` 所产生的种种不确定因素也往往很难掌控，我们又不能一味地通过 `try/catch` 去解决。所幸的是，经过全局业务排查，目前项目中在绝大多数因此，在查阅官方文档后，我们决定绕过 `evalString`，直接基于 JSB 绑定的方式进行通信。

这里以下载器的接入为例。在我们的项目中，下载器是在 Android 与 iOS 侧分别各自实现。在改造之前的版本中，下载器的调用与回调基于 `callStaticMethod <-> evalString` 的方式。
每次调用下载都需要这样执行：

```js
if(cc.sys.isNative && cc.sys.os == cc.sys.OS_IOS) {
    jsb.reflection.callStaticMethod('ABCFileDownloader', 'downloadFileWithUrl:cookie:savePath:', url, cookies, savePath);
} else if(cc.sys.isNative && cc.sys.os == cc.sys.OS_ANDROID) {
    jsb.reflection.callStaticMethod("com/tencent/abcmouse/downloader/ABCFileDownloader", "downloadFileWithUrl", "(Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;)V", url, cookies, savePath);
}
```

下载成功抑或是失败都需要通过拼接出类似如下的语句执行 JS：

```java
StringBuilder sb = new StringBuilder(JS_STRING_ON_DOWNLOAD_FINISH + "(");
sb.append("'" + success + "',");
sb.append("'" + url + "',");
sb.append("'" + savePath + "',");
sb.append("'" + msg + "',");
sb.append("'" +code + "')");
Cocos2dxJavascriptJavaBridge.evalString(sb.toString());
```

无论是调用抑或是回调都拼接繁琐又容易出错，全部数据不得不转化为字符串 ~~（emmmm也不美观）~~，而且还要考虑到 `evalString` 的执行效率问题。如果只是仅有的少数业务场景在使用尚勉强接受，但是当业务日趋复杂庞大，如果都要这样写，同时又没有详细的文档去规范约束，其后期维护成本可想而知。

而当使用 JSB 改造后，我们调用只需如下寥寥几行代码且无需区分平台，更不必担心上述拼接隐患，相比之下逻辑要清晰许多：

```js
jsb.fileDownloader.requestDownload(url, savePath, cookies, options, (success, url, savePath, msg, code) => {
    // do whatever you want
});
```

那么接下来就以一个最简单的下载器的绑定流程为例，我来带大家学习下 JSB 手动绑定的大致流程。
**（虽然 Cocos 很人性化提供了自动绑定的配置文件，可以通过一些配置直接生成目标文件，减少了很多工作量。但是亲手来完成一次手动绑定的流程会帮助更为全面地了解整个绑定的实现流程，有助于加深理解。另一方面，当存在特殊需要自动绑定无法满足时，手动绑定也往往会更为灵活）**

## 前置

在开始之前，我们需要需要知道有关 ScriptEngine 抽象层、相关 API 等相关知识，这部分内容如果已从 Cocos 文档了解可跳过，直接进行 **实践** 部分。

### 抽象层

![img](../image/Cocos JSB教程/JSB2.0-Architecture.png)

首先先来看一下上图 Cocos 官方提供的一张抽象层架构，在 1.7 版本中，抽象层被设计为一个与引擎没有关系的独立模块，对 JS 引擎的管理从 `ScriptingCore` 被移动到了 `se::ScriptEngine` 类中，`ScriptingCore` 被保留下来是希望通过它把引擎的一些事件传递给封装层，充当适配器的角色。在这个抽象层提供了对 JavaScriptCore、SpiderMonkey、V8、ChakraCore 等多种可选的 JS 执行引擎的封装。JSB 的大部分工作其实就是设定 JS 相关操作的 C++ 回调，在回调函数中关联 C++ 对象。它其实主要包含如下两种类型：

- 注册 JS 函数（包含全局函数，类构造函数、类析构函数、类成员函数，类静态成员函数），绑定一个 C++ 回调
- 注册 JS 对象的属性读写访问器，分别绑定读与写的 C++ 回调

考虑到不同多种 JS 引擎的关键方法的定义各不相同，Cocos 团队使用 **宏** 来抹平这种回调函数定义与参数类型的差异，这里就不展开，详细可阅读文末 Cocos Creator 的相关文档。
**值得一提的是，ScriptEngine 这层设计之初 Cocos 团队就将其定义为一个独立模块，完全不依赖 Cocos 引擎。** 我们开发者完全可以把 **cocos/scripting/js-bindings/jswrapper** 下的所有抽象层源码移植到其他项目中直接使用。

### SE 类型

C++ 抽象层所有的类型都在 `se` 命名空间下，其为 ScriptEngine 的缩写。

- **se::ScriptEngine**

  它是 JS 引擎的管理员，掌管 JS 引擎初始化、销毁、重启、Native 模块注册、加载脚本、强制垃圾回收、JS 异常清理、是否启用调试器。它是一个单例，可通过 `se::ScriptEngine::getInstance()` 得到对应的实例。

- **se::Value**

  可以被理解为 JS 变量在 C++ 层的引用。JS 变量有 `object`、`number`、`string`、`boolean`、`null`、`undefined` 六种类型，因此 `se::Value` 使用 union 包含 `object`、`number`、`string`、`boolean` 4 种有值类型，无值类型: `null`、`undefined` 可由私有变量 `_type` 直接表示。

  如果 `se::Value` 中保存基础数据类型，比如 `number`、`string`、`boolean`，其内部是直接存储一份值副本。`object` 的存储比较特殊，是通过 `se::Object*` 对 JS 对象的弱引用。

- **se::Object**

  继承于 `se::RefCounter` 引用计数管理类，它保存了对 JS 对象的弱引用。我们在绑定回调中如果需要用到当前对象对应的 `se::Object`，只需要通过 `s.thisObject()` 即可获取。其中 s 为 `se::State` 类型。

- **se::Class**

  用于暴露 C++ 类到 JS 中，它会在 JS 中创建一个对应名称的构造函数。Class 类型创建后，不需要手动释放内存，它会被封装层自动处理。`se::Class` 提供了一些 API 用于定义 Class 的创建、静态/动态成员函数、属性读写等等，后面在实践时用到会做介绍。完整内容可查阅 Cocos 文档。

- **se::State**

  它是绑定回调中的一个环境，我们通过 `se::State` 可以取得当前的 C++ 指针、`se::Object` 对象指针、参数列表、返回值引用。

### 宏

前面有提到，抽象层使用宏来抹平不同 JS 引擎关键函数定义与参数类型的不同，不管底层是使用什么引擎，开发者统一使用一种函数的定义。

例如，抽象层所有的 JS 到 C++ 的回调函数的定义为：

```cpp
bool foo(se::State& s)
{
    ...
    ...
}
SE_BIND_FUNC(foo) // 此处以回调函数的定义为例
```

我们在编写完回调函数后，需要记住使用 `SE_BIND_XXX` 系列的宏对回调函数进行包装。目前全部的 `SE_BIND_XXX` 宏如下所示：

- `SE_BIND_PROP_GET`：包装一个 JS 对象属性读取的回调函数
- `SE_BIND_PROP_SET`：包装一个 JS 对象属性写入的回调函数
- `SE_BIND_FUNC`：包装一个 JS 函数，可用于全局函数、类成员函数、类静态函数
- `SE_DECLARE_FUNC`：声明一个 JS 函数，一般在 `.h` 头文件中使用
- `SE_BIND_CTOR`：包装一个 JS 构造函数
- `SE_BIND_SUB_CLS_CTOR`：包装一个 JS 子类的构造函数，此子类使用 `cc.Class.extend` 继承 Native 绑定类
- `SE_FINALIZE_FUNC`：包装一个 JS 对象被 GC 回收后的回调函数
- `SE_DECLARE_FINALIZE_FUNC`：声明一个 JS 对象被 GC 回收后的回调函数
- `_SE`：包装回调函数的名称，转义为每个 JS 引擎能够识别的回调函数的定义，注意，第一个字符为下划线，类似 Windows 下用的 _T("xxx") 来包装 Unicode 或者 MultiBytes 字符串

在我们的简化版例子中，只需要用到 `SE_DECLARE_FUNC`、`SE_BIND_FUNC` 即可。

### 类型转换辅助函数

类型转换辅助函数位于 **cocos/scripting/js-bindings/manual/jsb_conversions.hpp/.cpp** 中，包含了多种 `se::Value` 与 C++ 类型相互转化的方法。

- `bool std_string_to_seval(const std::string& v, se::Value* ret);`
- `bool seval_to_std_string(const se::Value& v, std::string* ret);`
- `bool boolean_to_seval(bool v, se::Value* ret);`
- `bool seval_to_boolean(const se::Value& v, bool* ret);` ... ...

## 实践

在开始之前，我们需要明确一下流程。JSB 绑定简单来讲就是在 C++ 层实现一些类库，然后经过一些特定处理可以在 JS 端进行对应方法调用的过程。因为采用 JS 为主要业务编写语言，使得我们在做一些 Native 的功能时会比较受限，例如文件、网络等等相关操作。

以 Cocos2d-js 文档中 `cc.Sprite` 为例，在 JSB 中 如果使用 `new` 操作符来调用 `cc.Sprite` 的构造函数，实际上在 C++ 层会调用 `js_cocos2dx_Sprite_constructor` 函数。在这个 C++ 函数中，会为这个精灵对象分配内存，并把它添加到自动回收池，然后调用 JS 层的 `_ctor` 函数来完成初始化。在 `_ctor` 函数中会根据参数类型和数量调用不同的 init 函数，这些 init 函数也是 C++ 函数的绑定：

```cpp
#define SE_BIND_CTOR(funcName, cls, finalizeCb) \
    void funcName##Registry(const v8::FunctionCallbackInfo<v8::Value>& _v8args) \
    { \
        v8::Isolate* _isolate = _v8args.GetIsolate(); \
        v8::HandleScope _hs(_isolate); \
        bool ret = true; \
        se::ValueArray args; \
        se::internal::jsToSeArgs(_v8args, &args); \
        se::Object* thisObject = se::Object::_createJSObject(cls, _v8args.This()); \
        thisObject->_setFinalizeCallback(_SE(finalizeCb)); \
        se::State state(thisObject, args); \
        ret = funcName(state); \
        if (!ret) { \
            SE_LOGE("[ERROR] Failed to invoke %s, location: %s:%d\n", #funcName, __FILE__, __LINE__); \
        } \
        se::Value _property; \
        bool _found = false; \
        _found = thisObject->getProperty("_ctor", &_property); \
        if (_found) _property.toObject()->call(args, thisObject); \
    }
```

三层的方法对应关系如下：

| Javascript                        | JSB                                        | Cocos2d-x                                |
| :-------------------------------- | :----------------------------------------- | :--------------------------------------- |
| cc.Sprite.initWithSpriteFrameName | js_cocos2dx_Sprite_initWithSpriteFrameName | cocos2d::Sprite::initWithSpriteFrameName |
| cc.Sprite.initWithSpriteFrame     | js_cocos2dx_Sprite_initWithSpriteFrame     | cocos2d::Sprite::initWithSpriteFrame     |
| cc.Sprite.initWithFile            | js_cocos2dx_Sprite_initWithFile            | cocos2d::Sprite::initWithFile            |
| cc.Sprite.initWithTexture         | js_cocos2dx_Sprite_initWithTexture         | cocos2d::Sprite::initWithTexture         |

这个调用过程的时序如下：

[![ ](../image/Cocos JSB教程/jsb_process.png)](https://docs.cocos.com/creator/manual/zh/advanced-topics/jsb/jsb_process.png)

调用时序图（引自 Cocos2d-js 文档）

和上面的过程类似。首先，我们需要确定接口和字段，我们随便拟定一个最简单的下载器 `FileDownloader`，它所具备的是 `download(url, path, callback)` 接口，而在 `callback` 中我们需要拿到的则是 `code`，`msg`。并且为了方便使用，我们将它挂载在 `jsb` 对象下，这样我们便可以使用如下代码进行简单地调用:

```js
jsb.fileDownloader.download(url, path, (msg, code) => {
    // do whatever you want
});
```

确定接口后，我们可以开始着手码 C++ 部分了。首先来一发 `FileDownloader.h`，作为公共头文件供 Android/iOS 使用。接着 Android/iOS 分别实现各自的具体下载实现即可（此处略过），`reqCtx` 则用于存储回调对应关系：

```cpp
class FileDownloader {
        public:
            typedef std::function<void(const std::string& msg, const int code)> ResultCallback;
            static FileDownloader* getInstance();
            static void destroyInstance();
            void download(const std::string& url,
                                         const std::string& savePath,
                                         const ResultCallback& callback);
            void onDownloadResult(const std::string msg, const int code);
            ... ...
        protected:
            static FileDownloader* s_sharedFileDownloader;
            std::unordered_map<std::string, ResultCallback> reqCtx;
};
```

接下来我们进行最关键的绑定部分。
因为下载器就功能上分类属于 network 模块，我们可以选择将我们的 `FileDownloader` 的绑定实现在 Cocos 源码中现有的 `jsb_cocos2dx_network_auto` 中。在 `jsb_cocos2dx_network_auto.hpp` 中声明 JS 函数：

```cpp
SE_DECLARE_FUNC(js_cocos2dx_network_FileDownloader_download); // 声明成员函数，下载调用
SE_DECLARE_FUNC(js_cocos2dx_network_FileDownloader_getInstance); // 声明静态函数，获取单例
```

随后在 `jsb_cocos2dx_network_auto.cpp` 中来注册 `FileDownloader` 和新声明的这两个函数到 JS 虚拟机中。首先先写好对应的两个方法实现留空，等注册逻辑完成后再来补全：

```cpp
static bool js_cocos2dx_network_FileDownloader_download(se::State &s) { // 方法名与声明时一致
    // TODO
}

SE_BIND_FUNC(js_cocos2dx_network_FileDownloader_download); // 包装该方法

static bool js_cocos2dx_network_FileDownloader_getInstance(se::State& s) { // 方法名与声明时一致
    // TODO
}

SE_BIND_FUNC(js_cocos2dx_network_FileDownloader_getInstance); // 包装该方法
```

现在我们开始编写注册逻辑，新增一个注册方法用于收归 `FileDownloader` 的全部注册逻辑：

```cpp
bool js_register_cocos2dx_network_FileDownloader(se::Object* obj) {
    auto cls = se::Class::create("FileDownloader", obj, nullptr, nullptr);
    cls->defineFunction("download", _SE(js_cocos2dx_network_FileDownloader_download));
    cls->defineStaticFunction("getInstance", _SE(js_cocos2dx_network_FileDownloader_getInstance));
    cls->install();
    JSBClassType::registerClass<cocos2d::network::FileDownloader>(cls);
    se::ScriptEngine::getInstance()->clearException();
    return true;
}
```

我们来看看这个方法里做了些什么重要的事情：

1. 调用 `se::Class::create(className, obj, parentProto, ctor)` 方法，创建了一个名为 `FileDownloader` 的 Class，注册成功后，在 JS 层中可以通过 `let xxx = new FileDownloader();`的方式创建实例。
2. 调用 `defineFunction(name, func)` 方法，定义了一个成员函数 `download`，并将其实现绑定到包装后的 `js_cocos2dx_network_FileDownloader_download` 上。
3. 调用 `defineStaticFunction(name, func)` 方法，定义了一个静态成员函数 `getInstance`，并将其实现绑定到包装后的 `js_cocos2dx_network_FileDownloader_getInstance` 上。
4. 调用 `install()` 方法，将自己注册到 JS 虚拟机中。
5. 调用 `JSBClassType::registerClass` 方法，将生成的 Class 与 C++ 层的类对应起来（内部通过 `std::unordered_map<std::string, se::Class*>` 实现）。

通过以上这几步，我们完成了关键的注册部分，当然不要忘记在 `network` 模块的注册入口添加 `js_register_cocos2dx_network_FileDownloader` 的调用：

```cpp
bool register_all_cocos2dx_network(se::Object* obj)
{
    // Get the ns
    se::Value nsVal;
    if (!obj->getProperty("jsb", &nsVal))
    {
        se::HandleObject jsobj(se::Object::createPlainObject());
        nsVal.setObject(jsobj);
        obj->setProperty("jsb", nsVal);
    }
    se::Object* ns = nsVal.toObject();

    ... ...
    // 将前面生成的 Class 注册 设置为 jsb 的一个属性，这样我们便能通过
    // let downloader = new jsb.FileDownloader();
    // 获取实例
    js_register_cocos2dx_network_FileDownloader(ns);
    return true;
}
```

完成这一步，我们的 Class 已经成功绑定，现在回来继续完善刚才留空的方法。

首先是 `getInstance()`：

```cpp
static bool js_cocos2dx_network_FileDownloader_getInstance(se::State& s)
{
    const auto& args = s.args();
    size_t argc = args.size();
    CC_UNUSED bool ok = true;
    if (argc == 0) {
        cocos2d::network::FileDownloader* result = cocos2d::network::FileDownloader::getInstance(); // C++ 单例
        ok &= native_ptr_to_seval<cocos2d::network::FileDownloader>((cocos2d::network::FileDownloader*)result, &s.rval());
        SE_PRECONDITION2(ok, false, "js_cocos2dx_network_FileDownloader_getInstance : Error processing arguments");
        return true;
    }
    SE_REPORT_ERROR("wrong number of arguments: %d, was expecting %d", (int)argc, 0);
    return false;
}
```

前面提到，我们可以通过 `se::State` 获取到 C++ 指针、`se::Object` 对象指针、参数列表、返回值引用。梳理逻辑如下：

1. `args()` 获取 JS 带过来的全部参数（`se::Value` 的 vector）；
2. 参数个数判断，因为这里的 `getInstance()` 并不需要额外参数，因此参数为 0；
3. `native_ptr_to_seval()` 用于在绑定层根据一个 C++ 对象指针获取一个 `se::Value`，并赋返回值给 `rval()` 至 JS 层；

到这里，`getInstance()` 的绑定层逻辑已全部完成，我们已经可以通过 `let downloader = jsb.FileDownloader.getInstance()` 获取实例了。

接着是 `download()`：

```cpp
static bool js_cocos2dx_network_FileDownloader_download(se::State &s) {
    cocos2d::network::FileDownloader *cobj = (cocos2d::network::FileDownloader *) s.nativeThisObject();
    SE_PRECONDITION2(cobj, false,
                     "js_cocos2dx_network_FileDownloader_download : Invalid Native Object");
    const auto &args = s.args();
    size_t argc = args.size();
    CC_UNUSED bool ok = true;
    if (argc == 3) {
        std::string url;
        std::string path;
        ok &= seval_to_std_string(args[0], &url); // 转化为std::string url
        ok &= seval_to_std_string(args[1], &path); // 转化为std::string path
        std::function<void(const std::string& msg,
                           const int code)> callback;
        do {
            if (args[2].isObject() && args[2].toObject()->isFunction())
            {
                se::Value jsThis(s.thisObject());
                // 获取 JS 回调
                se::Value jsFunc(args[2]);
                // 如果目标类是一个单例则不能用 se::Object::attachObject 去关联
                // 必须使用 se::Object::root，无需关心 unroot，unroot 的操作会随着 lambda 的销毁触发 jsFunc 的析构，在 se::Object 的析构函数中进行 unroot 操作。
                // 如果使用 s.thisObject->attachObject(jsFunc.toObject);会导致对应的 func 和 target 永远无法被释放，引发内存泄露。
                jsFunc.toObject()->root(); 
                auto lambda = [=](const std::string& msg,
                                  const int code) -> void {
                    se::ScriptEngine::getInstance()->clearException();
                    se::AutoHandleScope hs;
                    CC_UNUSED bool ok = true;
                    se::ValueArray args;
                    args.resize(2);
                    ok &= std_string_to_seval(msg, &args[0]);
                    ok &= int32_to_seval(code, &args[1]);
                    se::Value rval;
                    se::Object* thisObj = jsThis.isObject() ? jsThis.toObject() : nullptr;
                    se::Object* funcObj = jsFunc.toObject();
                    // 执行 JS 方法回调
                    bool succeed = funcObj->call(args, thisObj, &rval);
                    if (!succeed) {
                        se::ScriptEngine::getInstance()->clearException();
                    }
                };
                callback = lambda;
            }
            else
            {
                callback = nullptr;
            }
        } while(false);
        SE_PRECONDITION2(ok, false, "js_cocos2dx_network_FileDownloader_download : Error processing arguments");
        cobj->download(url, path, callback);
        return true;
    }
    SE_REPORT_ERROR("wrong number of arguments: %d, was expecting %d", (int) argc, 3);
    return false;
}
```

1. 通过 `seval_to_std_string` 方法获取转化 C++ 后的 url、path 参数和原始 jsFunc。
2. 手动构造回调 function，将 msg 和 code 转化为 `se::Value`。
3. 通过 `funcObj->call` 执行 JS 方法进行回调。

以上即为一次普通调用的回调的执行过程的绑定。现在我们还剩下一些收尾工作，我们需要将 `FileDownloader` 真正成为单例，在 JS 层无需手动实例化即可使用。 因为下载器属于通用组件，所以我们需要尽早将其实例化并成功挂载，因此我们需要修改 `jsb_boot.js`，这个文件会在 Cocos 引擎初始化时调用，我们在其中补充如下代码：

```js
// FileDownloader
jsb.fileDownloader = jsb.FileDownloader.getInstance();
delete jsb.FileDownloader;
```

最后，考虑到内存释放的风险，我们还需要在 `CCDirector.cpp` 中的 `reset()` 方法中进行相关回收：

```cpp
network::FileDownloader::destroyInstance();
```

================================================

以上就是全部的绑定流程，在分别编译到 Android/iOS 环境后，我们就能够通过 `jsb.fileDownloader.download()` 进行下载调用了。
（PS：一定切记在使用前进行 `CC_JSB` 的宏判断，因为非 JSB 环境下是无法使用的）

## 总结

我们现在来总结一下手动绑定改造的详细流程。一般而言，常用的 JSB 的改造流程大致如下：

- 确定方法接口与 JS/Native 公共字段
- 声明头文件，并分别实现 Android JNI 与 OC 具体业务代码
- 编写抽象层代码，将必要的类与对应方法注册到 JS 虚拟机中
- 将绑定的类挂载在 JS 中的指定对象（类似命名空间）中

# 使用 JSB 自动绑定

尽管 Cocos 官方提供了 `jsb.reflection.callStaticMethod` 方式支持从 JS 端直接调用 Native 端（Android/iOS/Mac）的接口，但是经过大量实践发现此接口在大量频繁调用情况下性能很低下，尤其是在 Android 端，比如调用 Native 端实现的打印 log 的接口，而且会容易引起一些 native crash，例如 `local reference table overflow` 等问题。纵观 Cocos 原生代码的实现，基本所有的接口方法的实现都是基于 JSB 的方式来实现，所以此文主要讲解下 JSB 的自动绑定逻辑，帮助大家能快速实现 `callStaticMethod` 到 JSB 的改造过程。

## 背景

对于用过 Cocos Creator（为了方便后文直接简称 CC）的人来说，`jsb.reflection.callStaticMethod` 这个方法肯定不陌生，其提供了我们从 JS 端调用 Native 端的能力，例如我们要调用 Native 实现的 log 打印和持久化的接口，就可以很方便的在 JavaScript 中按照如下的操作调用即可：

```javascript
if (cc.sys.isNative && cc.sys.os == cc.sys.OS_IOS) {
    msg = this.buffer_string + '\n[cclog][' + clock + '][' + tag + ']' + msg;
    jsb.reflection.callStaticMethod("ABCLogServuce", "log:module:level:", msg, 'cclog', level);
    return;
} else if (cc.sys.isNative && cc.sys.os == cc.sys.OS_ANDROID) {
    msg = this.buffer_string + '\n[cclog][' + clock + '][' + tag + ']' + msg;
    jsb.reflection.callStaticMethod("com/example/test/CommonUtils", "log", "(ILjava/lang/String;Ljava/lang/String;)V", level, 'cclog', msg);
    return;
}
```

尽管使用很简单，一行代码就能实现跨平台调用，稍微看下其实现就可以知道，在 C++ 层 Android 端是通过 jni 的方式实现的，IOS 端是通过运行时的方式动态调用，但是为了兼顾通用性和支持所有的方法，Android 端没有对 jni 相关对象做缓存机制，就会导致短时间大量调用时出现很严重的性能问题，之前我们遇到的比较多的情况就是在下载器中打印 log，某些应用场景短时间内触发大量的下载操作，就会出现 `local reference table overflow` 的 crash，甚至在低端机上导致界面卡顿无法加载出来的问题。

修复此问题就需要针对 log 调用进行 JSB 的改造，同时还要加上 jni 的相关缓存机制，优化性能。jSB 绑定说白了就是 C++ 和脚本层之间进行对象的转换，并转发脚本层函数调用到 C++ 层的过程。

JSB 绑定通常有 **手动绑定** 和 **自动绑定** 两种方式。手动绑定方式可以参考 [使用 JSB 手动绑定](https://docs.cocos.com/creator/manual/zh/advanced-topics/jsb-manual-binding.html)。

- 手动绑定方式优点是灵活，可定制型强；缺点就是全部代码要自己书写，尤其是在 js 类型跟 c++ 类型转换上，稍有不慎容易导致内存泄漏，某些指针或者对象没有释放。
- 自动绑定方式则会帮你省了很多麻烦，直接通过一个脚本一键生成相关的代码，后续如果有新增或者改动，也只需要重新执行一次脚本即可。所以自动绑定对于不需要进行强定制，需要快速完成JSB的情况来说就再适合不过了。下面就一步步说明下如何实现自动绑定 JSB。

## 环境配置和自动绑定展示

### 环境配置

自动绑定，说简单点，其实就只要执行一个 python 脚本即可自动生成对应的 `.cpp`、`.h`、`.js` 文件。所以首先要保证电脑有 python 运行环境，这里以 Mac 上安装为例来讲解。

1. 安装 python，强烈建议先安装 [HomeBrew](https://brew.sh/)，然后直接命令行运行：

   ```shell
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    brew install python
   ```

2. 通过 pip 安装 python 的一些依赖库：

   ```shell
    sudo easy_install pip
    sudo pip install PyYAML
    sudo pip install Cheetah
   ```

3. 安装 NDK，涉及到 c++ 肯定这个是必不可少的，建议安装 [14b](https://dl.google.com/android/repository/android-ndk-r14b-darwin-x86_64.zip) 版本，然后在 `~/.bash_profile` 中设置 `PYTHON_ROOT` 和 `NDK_ROOT` 这两个环境变量，因为在后面执行的 python 文件里面就会直接用到这两个环境变量：

   ```shell
    export NDK_ROOT="/Users/mycount/Library/Android/sdk/ndk-r14b"
    export PYTHON_BIN="/usr/bin/python"
   ```

Window 下直接参考上面需要安装的模块直接安装就好了，最后也要记得配置环境变量。

### 自动绑定展示

这里演示的是 cocos 引擎下面也即⁨ `build/⁨jsb-default⁩/frameworks⁩/cocos2d-x/cocos⁩/scripting⁩/js-bindings/⁨auto⁩` 目录下的文件（如下图所示）是怎么自动生成的：

![img](../image/Cocos JSB教程/auto-file.png)

其实从这些文件名的开头也能看出，这些文件命名都是有某些特定规律的，那么这些文件是怎么生成的呢？首先打开终端，先 cd 到`build/jsb-default/frameworks/cocos2d-x/tools/tojs` 目录下，然后直接运行 `./genbindings.py`：

![img](../image/Cocos JSB教程/generate-file.png)

大概运行一分钟左右后，会出现如下的提示，说明已经顺利生成完了：

![img](../image/Cocos JSB教程/generate-file-complete.png)

当然一般大家第一次运行后很可能会失败，出现如下面类似的报错：

![img](../image/Cocos JSB教程/error-1.png)

![img](../image/Cocos JSB教程/error-2.png)

一般都是因为配置的 NDK 版本太高导致，最开始我是用 NDK16b 就出现了问题，换成 NDK14b 后就 OK 了。

经过上面的步骤后，`build/⁨jsb-default⁩/frameworks⁩/cocos2d-x/cocos⁩/scripting⁩/js-bindings/⁨auto⁩` 下的文件就全部自动生成出来了，是不是非常方便。

下面再以 js 层通过 jsb 调用 Native 层的 log 方法打印日志为例，详细的告知下如何实现通过自动绑定工具，依据自己写的 c++ 代码，生成对应的自动绑定文件。

## 编写 c++ 层的实现

C++ 作为连接 js 层和 Native 层的桥梁，既然要实现 jsb 调用，那第一步肯定是要先把 C++ 层的头文件和实现准备好，这里我们在 build⁩/jsb-defaul/frameworks⁩/cocos2d-x⁩/cocos⁩ 创建一个 test 文件夹用于存放相关文件：

![img](../image/Cocos JSB教程/store-file.png)

这里先准备 `ABCJSBBridge.h`，里面主要是申明了一个 `abcLog` 的函数，此函数就是供 JS 层调用打 log 的，另外由于打 log 方法肯定在 js 层很多地方都会使用，所以这里采用了一个单例模式，提供了 `getInstance()` 来获取当前类的实例。

```cpp
#include <string>
#include "base/CCConsole.h"

#ifndef PROJ_ANDROID_STUDIO_ABCJSBBRIDGE_H
#define PROJ_ANDROID_STUDIO_ABCJSBBRIDGE_H
#define DLLOG(format, ...)      cocos2d::log(format, ##__VA_ARGS__)
#endif //PROJ_ANDROID_STUDIO_ABCJSBBRIDGE_H

namespace abc
{
    class IABCJSBBridgeImpl;
    class JSBBridge
    {
    public:
        void abcLog(const int level, const std::string& tag, const std::string& msg);
        /**
        * Returns a shared instance of the director.
        * @js _getInstance
        */
        static JSBBridge* getInstance();

        /** @private */
        JSBBridge();
        /** @private */
        ~JSBBridge();
        bool init();

    private:
        std::unique_ptr<IABCJSBBridgeImpl> _impl;
    };
}
```

下面是对应的实现 `ABCJSBBridge.cpp`：

```cpp
#include "ABCJSBBridge.h"

// include platform specific implement class
#if (CC_TARGET_PLATFORM == CC_PLATFORM_MAC || CC_TARGET_PLATFORM == CC_PLATFORM_IOS)

#include "ABCJSBBridge-apple.h"
#define JSBBridgeImpl  JSBBridgeApple

#elif (CC_TARGET_PLATFORM == CC_PLATFORM_ANDROID)

#include "ABCJSBBridge-android.h"
#define JSBBridgeImpl  JSBBridgeAndroid

#endif

namespace abc
{
    // singleton stuff
    static JSBBridge *s_SharedJSBBridge = nullptr;

    JSBBridge::JSBBridge()
    {
        DLLOG("Construct JSBBridge %p", this);
        init();
    }

    JSBBridge::~JSBBridge()
    {
        DLLOG("Destruct JSBBridge %p", this);
        s_SharedJSBBridge = nullptr;
    }

    JSBBridge* JSBBridge::getInstance()
    {
        if (!s_SharedJSBBridge)
        {
            DLLOG("getInstance JSBBridge ");
            s_SharedJSBBridge = new (std::nothrow) JSBBridge();
            CCASSERT(s_SharedJSBBridge, "FATAL: Not enough memory for create JSBBridge");
        }

        return s_SharedJSBBridge;
    }

    bool JSBBridge::init(void)
    {
        DLLOG("init JSBBridge ");
        _impl.reset(new JSBBridgeImpl());
    }

    void JSBBridge::abcLog(const int level, const std::string& tag, const std::string& msg)
    {
        _impl->abcLog(level, tag, msg);
    }
}
```

`CCIABCJSBBridgeIml.h`：

```cpp
#include <string>
#include "base/CCConsole.h"

#ifndef PROJ_ANDROID_STUDIO_CCIABCJSBBRIDGEIML_H
#define PROJ_ANDROID_STUDIO_CCIABCJSBBRIDGEIML_H

#endif //PROJ_ANDROID_STUDIO_CCIABCJSBBRIDGEIML_H

#define DLLOG(format, ...)      cocos2d::log(format, ##__VA_ARGS__)

namespace abc
{
    class IABCJSBBridgeImpl
    {
    public:
        virtual ~IABCJSBBridgeImpl(){}
        virtual void abcLog(const int level, const std::string& tag, const std::string& msg) = 0;
    };
}
```

这里为了方便区分 Android 平台和 iOS 平台的实现，仿照 Cocos 源码其他地方的写法，分别提供了 `ABCJSBBridge-android.h` 和 `ABCJSBBridge-apple.h` 以及对应的实现类，两个平台分别继承 `IABCJSBBridgeImpl` 然后实现内部的虚函数即可。

## JSB 配置脚本编写

为了保持跟官方的一致，我们在 `build/jsb-default/frameworks/cocos2d-x/tools/tojs` 目录下创建 `genbindings_test.py`，里面的内容基本跟 `genbindings.py` 差不多，主要区别有如下几点：

1. 去掉了 `cmd_args` 那段，里面主要是记录了 cocos 自带的一些需要生成 jsb 的文件，因为考虑到项目可能会对 Cocos 源码进行修改，如果这时候把这部分保留的话，当运行脚本后会把我们自带的修改就给覆盖掉了。

2. 取消了定制的 `output_dir` 也就是最终生成的 js，c++ 等绑定文件的路径，而是保持跟 Cocos 一样，也即在 `cocos/scripting/js-bindings/auto`，主要为了方便下一步配置 mk 文件。

   ![img](../image/Cocos JSB教程/cancel-output_dir.png)

这里先说下 `genbindings_test.py` 里面配置的一些参数：

1. **NDK_ROOT 环境变**：指示 NDK 的根目录
2. **PYTHON_BIN 环境变量**：指示 Python 命令的路径
3. **cocosdir**：Cocos 引擎根目录，在用户工程下一般是 `build/jsb-default/frameworks/cocos2d-x/`
4. **jsbdir**：JSB 目录，在用户工程下一般是 `build/jsb-default/frameworks/cocos2d-x/cocos/scripting/js-bindings`
5. **cxx_generator_root**：自动绑定工具路径，在用户工程下一般是 `build/jsb-default/frameworks/cocos2d-x/tools/bindings-generator`
6. **output_dir**：生成的绑定文件存储路径
7. **cmd_args** 和 **custom_cmd_args**：所有配置文件，及其对应的模块名称和输出文件名称

这里自动绑定工具使用 `libclang` 的 python API 对 C++ 头文件进行语法分析。绑定的过程大致如下：

- 创建绑定代码输出文件。
- 递归扫描需要绑定的头文件。
- 通过 `libclang` 的 `clang.cindex` python 模块找到所有需要绑定的类，公共 API 等。
- 按照模版生成类绑定函数，API 绑定函数，绑定注册函数并输出到文件中。

关于 `custom_cmd_args` 参数的配置这里说明下：

```xml
'cocos2dx_test.ini': ('cocos2dx_test', 'jsb_cocos2dx_test_auto'),
配置文件：（模块名称，输出的绑定文件名）
```

这里的配置文件 `cocos2dx_test.ini` 又是用来干嘛的呢？其实就跟 `build/jsb-default/frameworks/cocos2d-x/tools/tojs/` 下的其他 `.ini` 文件类似，主要让自动绑定工具知道哪些 API 要被绑定和以什么样的方式绑定，写法上直接参考 Cocos 已有的 ini 文件，这里展示下 `cocos2dx_test.ini` 的内容：

```shell
[cocos2dx_test]
# the prefix to be added to the generated functions. You might or might not use this in your own
# templates
prefix = cocos2dx_test

# create a target namespace (in javascript, this would create some code like the equiv. to `ns = ns || {}`)
# all classes will be embedded in that namespace
target_namespace = abc

android_headers = -I%(androidndkdir)s/platforms/android-14/arch-arm/usr/include -I%(androidndkdir)s/sources/cxx-stl/gnu-libstdc++/4.8/libs/armeabi-v7a/include -I%(androidndkdir)s/sources/cxx-stl/gnu-libstdc++/4.8/include -I%(androidndkdir)s/sources/cxx-stl/gnu-libstdc++/4.9/libs/armeabi-v7a/include -I%(androidndkdir)s/sources/cxx-stl/gnu-libstdc++/4.9/include
android_flags = -D_SIZE_T_DEFINED_ 

clang_headers = -I%(clangllvmdir)s/%(clang_include)s 
clang_flags = -nostdinc -x c++ -std=c++11 -U __SSE__

cocos_headers = -I%(cocosdir)s -I%(cocosdir)s/cocos -I%(cocosdir)s/cocos/platform/android -I%(cocosdir)s/external

cocos_flags = -DANDROID

cxxgenerator_headers = 

# extra arguments for clang
extra_arguments = %(android_headers)s %(clang_headers)s %(cxxgenerator_headers)s %(cocos_headers)s %(android_flags)s %(clang_flags)s %(cocos_flags)s %(extra_flags)s 

# what headers to parse
headers = %(cocosdir)s/cocos/test/ABCJSBBridge.h

# cpp_headers = network/js_network_manual.h

# what classes to produce code for. You can use regular expressions here. When testing the regular
# expression, it will be enclosed in "^$", like this: "^Menu*$".
classes = JSBBridge

# what should we skip? in the format ClassName::[function function]
# ClassName is a regular expression, but will be used like this: "^ClassName$" functions are also
# regular expressions, they will not be surrounded by "^$". If you want to skip a whole class, just
# add a single "*" as functions. See bellow for several examples. A special class name is "*", which
# will apply to all class names. This is a convenience wildcard to be able to skip similar named
# functions from all classes.
skip = JSBBridge::[init]

rename_functions = 

rename_classes =

# for all class names, should we remove something when registering in the target VM?
remove_prefix = 

# classes for which there will be no "parent" lookup
classes_have_no_parents = JSBBridge

# base classes which will be skipped when their sub-classes found them.
base_classes_to_skip = Clonable

# classes that create no constructor
# Set is special and we will use a hand-written constructor
abstract_classes = JSBBridge
```

其实从里面的注释也讲的非常详细，这里说几个主要的属性及含义：

![img](../image/Cocos JSB教程/ini-file-properties.png)

以上的配置完成后，就可以按照如下命令运行自动生成绑定文件：

![img](../image/Cocos JSB教程/generate-binding-file.png)

然后就会看到在 **build/jsb-default/frameworks/cocos2d-x/cocos/scripting/js-bindings** 下面多出了三个绑定文件：

![img](../image/Cocos JSB教程/binding-file.png)

打开生成的 `jsb_cocos2dx_test_autp.cpp`：

```cpp
#include "scripting/js-bindings/auto/jsb_cocos2dx_test_auto.hpp"
#include "scripting/js-bindings/manual/jsb_conversions.hpp"
#include "scripting/js-bindings/manual/jsb_global.h"
#include "test/ABCJSBBridge.h"

se::Object* __jsb_abc_JSBBridge_proto = nullptr;
se::Class* __jsb_abc_JSBBridge_class = nullptr;

static bool js_cocos2dx_test_JSBBridge_abcLog(se::State& s)
{
    abc::JSBBridge* cobj = (abc::JSBBridge*)s.nativeThisObject();
    SE_PRECONDITION2(cobj, false, "js_cocos2dx_test_JSBBridge_abcLog : Invalid Native Object");
    const auto& args = s.args();
    size_t argc = args.size();
    CC_UNUSED bool ok = true;
    if (argc == 3) {
        int arg0 = 0;
        std::string arg1;
        std::string arg2;
        ok &= seval_to_int32(args[0], (int32_t*)&arg0);
        ok &= seval_to_std_string(args[1], &arg1);
        ok &= seval_to_std_string(args[2], &arg2);
        SE_PRECONDITION2(ok, false, "js_cocos2dx_test_JSBBridge_abcLog : Error processing arguments");
        cobj->abcLog(arg0, arg1, arg2);
        return true;
    }
    SE_REPORT_ERROR("wrong number of arguments: %d, was expecting %d", (int)argc, 3);
    return false;
}
SE_BIND_FUNC(js_cocos2dx_test_JSBBridge_abcLog)

static bool js_cocos2dx_test_JSBBridge_getInstance(se::State& s)
{
    const auto& args = s.args();
    size_t argc = args.size();
    CC_UNUSED bool ok = true;
    if (argc == 0) {
        abc::JSBBridge* result = abc::JSBBridge::getInstance();
        ok &= native_ptr_to_seval<abc::JSBBridge>((abc::JSBBridge*)result, &s.rval());
        SE_PRECONDITION2(ok, false, "js_cocos2dx_test_JSBBridge_getInstance : Error processing arguments");
        return true;
    }
    SE_REPORT_ERROR("wrong number of arguments: %d, was expecting %d", (int)argc, 0);
    return false;
}
SE_BIND_FUNC(js_cocos2dx_test_JSBBridge_getInstance)

bool js_register_cocos2dx_test_JSBBridge(se::Object* obj)
{
    auto cls = se::Class::create("JSBBridge", obj, nullptr, nullptr);

    cls->defineFunction("abcLog", _SE(js_cocos2dx_test_JSBBridge_abcLog));
    cls->defineStaticFunction("getInstance", _SE(js_cocos2dx_test_JSBBridge_getInstance));
    cls->install();
    JSBClassType::registerClass<abc::JSBBridge>(cls);

    __jsb_abc_JSBBridge_proto = cls->getProto();
    __jsb_abc_JSBBridge_class = cls;

    se::ScriptEngine::getInstance()->clearException();
    return true;
}

bool register_all_cocos2dx_test(se::Object* obj)
{
    // Get the ns
    se::Value nsVal;
    if (!obj->getProperty("abc", &nsVal))
    {
        se::HandleObject jsobj(se::Object::createPlainObject());
        nsVal.setObject(jsobj);
        obj->setProperty("abc", nsVal);
    }
    se::Object* ns = nsVal.toObject();

    js_register_cocos2dx_test_JSBBridge(ns);
    return true;
}
```

看到这里是不是感觉很熟悉，跟 Cocos 已有的那些 cpp 完全一样，甚至包括里面的注册函数和类的定义都给全部自动生成了。

## Cocos 编译配置

尽管经过上面一步后我们已经生成出来了绑定文件，但是 js 层还是没法直接使用，因为还需要把生成的绑定文件，配置到 mk 文件中，从而跟其他 c++ 文件一起编译才行，这部分主要就是将最后的 mk 编译配置。

1. 打开 `build/jsb-default/frameworks/cocos2d-x/cocos/Android.mk` 文件，在其中加上最开始实现的 cpp 文件：

   ![img](../image/Cocos JSB教程/100.png)

2. 打开 `build/jsb-default/frameworks/cocos2d-x/cocos/scripting/js-bindings/proj.android/Android.mk`，在其中加上上一步生成的 cpp 文件：

   ![img](../image/Cocos JSB教程/110.png)

3. 打开 `build/jsb-default/frameworks/runtime-src/Classes/jsb_module_register.cpp`，添加引擎启动时调用绑定文件的注册函数，从而将其添加到 js 环境中：

   ![img](../image/Cocos JSB教程/111.png)

4. 打开 `build/jsb-default/frameworks/cocos2d-x/cocos/scripting/js-bindings/script/jsb_boot.js`，在其中增加 js 对象的初始化：

   ![img](../image/Cocos JSB教程/112.png)

上面说到的 `jsb_module_register.cpp` 和 `jsb_boot.js` 其实都是在 Cocos 引擎初始化的时候就会去调用的，关于启动流程感兴趣的可以去看看这篇 [文章](https://gowa.club/Cocos-Creator/Cocos Creator生成项目的启动工作流程.html)。

经过上面这些配置后，最终就可以在 js 层直接像下面这样来进行调用，而不是用 `callStaticMethod` 方式：

![img](../image/Cocos JSB教程/called-injs.png)

## 自动绑定的限制条件

自动绑定依赖于 Bindings Generator 工具，Cocos 官方还单独把这部分拎出来了：[GitHub](https://github.com/cocos-creator/bindings-generator) | [Gitee](https://gitee.com/mirrors_cocos-creator/bindings-generator)。Bindings Generator 工具它可以将 C++ 类的公共方法和公共属性绑定到脚本层。自动绑定工具尽管非常强大，但是还是会有一些限制：

1. 只能够针对类生成绑定，不可以绑定结构体，独立函数等。
2. 不能够生成 `Delegate` 类型的 API，因为脚本中的对象是无法继承 C++ 中的 `Delegate` 类并重写其中的 `Delegate` 函数的。
3. 子类中重写了父类的 API 的同时，又重载了这个 API。
4. 部分 API 实现内容并没有完全体现在其 API 定义中。
5. 在运行时由 C++ 主动调用的 API。

## 总结

总的来说，自动绑定 JSB 只需要要求开发者编写相关的实现 c++ 实现类，一个配置文件，然后执行一条命令就能完整整个的绑定过程。如果没有什么特殊定制，相对于手动绑定来说，效率上还是提高了不少。实际工作做可以依据具体情况先用自动绑定功能，然后再去手动修改生成的绑定文件，达到事倍功半的效果。

