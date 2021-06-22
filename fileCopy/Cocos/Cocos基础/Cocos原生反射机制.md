# 原生反射机制

# 如何在 Android 平台上使用 JavaScript 直接调用 Java 方法

使用 Creator 打包的安卓原生应用中，我们可以通过反射机制直接在 JavaScript 中调用 Java 的静态方法。它的使用方法很简单：

```js
var o = jsb.reflection.callStaticMethod(className, methodName, methodSignature, parameters...)
```

在 `callStaticMethod` 方法中，我们通过传入 Java 的类名，方法名，方法签名，参数就可以直接调用 Java 的静态方法，并且可以获得 Java 方法的返回值。下面介绍的类名和方法签名可能会有一点奇怪，但是 Java 的规范就是如此的。

## 类名

参数中的类名必须是包含 Java 包路径的完整类名，例如我们在 `org.cocos2dx.javascript` 这个包下面写了一个 `Test` 类：

```java
package org.cocos2dx.javascript;

public class Test {

    public static void hello(String msg){
        System.out.println(msg);
    }

    public static int sum(int a, int b){
        return a + b;
    }

    public static int sum(int a){
        return a + 2;
    }

}
```

那么这个 Test 类的完整类名应该是 `org/cocos2dx/javascript/Test`，注意这里必须是斜线 `/`，而不是在 Java 代码中我们习惯的点 `.`。

## 方法名

方法名很简单，就是方法本来的名字，例如 sum 方法的名字就是 `sum`。

## 方法签名

方法签名稍微有一点复杂，最简单的方法签名是 `()V`，它表示一个没有参数没有返回值的方法。其他一些例子：

- `(I)V` 表示参数为一个int，没有返回值的方法
- `(I)I` 表示参数为一个int，返回值为int的方法
- `(IF)Z` 表示参数为一个int和一个float，返回值为boolean的方法

现在有一些理解了吧，括号内的符号表示参数类型，括号后面的符号表示返回值类型。因为 Java 是允许函数重载的，可以有多个方法名相同但是参数返回值不同的方法，方法签名正是用来帮助区分这些相同名字的方法的。

目前 Cocos Creator 中支持的 Java 类型签名有下面 4 种：

| Java类型 | 签名                   |
| -------- | ---------------------- |
| int      | I                      |
| float    | F                      |
| boolean  | Z                      |
| String   | Ljava / lang / String; |

## 参数

参数可以是 0 个或任意多个，直接使用 JS 中的 number、bool 和 string 就可以。

## 使用示例

我们将会调用上面的 Test 类中的静态方法：

```js
// 调用 hello 方法
jsb.reflection.callStaticMethod("org/cocos2dx/javascript/Test", "hello", "(Ljava/lang/String;)V", "this is a message from js");

// 调用第一个 sum 方法
var result = jsb.reflection.callStaticMethod("org/cocos2dx/javascript/Test", "sum", "(II)I", 3, 7);
cc.log(result); //10

// 调用第二个 sum 方法
var result = jsb.reflection.callStaticMethod("org/cocos2dx/javascript/Test", "sum", "(I)I", 3);
cc.log(result); //5
```

在你的控制台会有正确的输出的，这很简单吧。

## 注意

另外有一点需要注意的就是，在 Android 应用中，Cocos 引擎的渲染和 JS 的逻辑是在 GL 线程中进行的，而 Android 本身的 UI 更新是在 App 的 UI 线程进行的，所以如果我们在 JS 中调用的 Java 方法有任何刷新 UI 的操作，都需要在 UI 线程进行。

例如，在下面的例子中我们会调用一个 Java 方法，它弹出一个 Android 的 Alert 对话框。

```c++
// 给我们熟悉的 AppActivity 类稍微加点东西
public class AppActivity extends Cocos2dxActivity {

    private static AppActivity app = null;
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        app = this;
    }

    public static void showAlertDialog(final String title,final String message) {

        // 这里一定要使用 runOnUiThread
        app.runOnUiThread(new Runnable() {
            @Override
            public void run() {
                AlertDialog alertDialog = new AlertDialog.Builder(app).create();
                alertDialog.setTitle(title);
                alertDialog.setMessage(message);
                alertDialog.setIcon(R.drawable.icon);
                alertDialog.show();
            }
        });
    }
}
```

然后我们在 JS 中调用：

```
jsb.reflection.callStaticMethod("org/cocos2dx/javascript/AppActivity", "showAlertDialog", "(Ljava/lang/String;Ljava/lang/String;)V", "title", "hahahahha");
```

这样调用你就可以看到一个 Android 原生的 Alert 对话框了。

## 再加点料

现在我们可以从 JS 调用 Java 了，那么能不能反过来？当然可以！

在你的项目中包含 Cocos2dxJavascriptJavaBridge，这个类有一个 `evalString` 方法可以执行 JS 代码，它位于 `frameworks\js-bindings\bindings\manual\platform\android\java\src\org\cocos2dx\lib` 文件夹下。我们将会给刚才的 Alert 对话框增加一个按钮，并在它的响应中执行 JS。和上面的情况相反，这次执行 JS 代码必须在 GL 线程中进行。

一般来说，目前引擎并未承诺多线程下的安全性，所以在开发过程中需要避免 JS 代码在其他线程被调用，以避免各种内存错误。

```java
alertDialog.setButton("OK", new DialogInterface.OnClickListener() {
    public void onClick(DialogInterface dialog, int which) {
        // 一定要在 GL 线程中执行
        app.runOnGLThread(new Runnable() {
            @Override
            public void run() {
                Cocos2dxJavascriptJavaBridge.evalString("cc.log(\"Javascript Java bridge!\")");
            }
        });
    }
});
```

如果要在 C++ 中调用 `evalString`，我们可以参考下面的方式，确保 `evalString` 在 JS 引擎所在的线程被执行：

```c++
Application::getInstance()->getScheduler()->performFunctionInCocosThread([=](){
    se::ScriptEngine::getInstance()->evalString(script.c_str());
});
```

这样在点击 OK 按钮后，你应该可以在控制台看到正确的输出。`evalString` 可以执行任何 JS 代码，并且它可以访问到你在 JS 代码中的对象。

# 如何在 iOS 平台上使用 Javascript 直接调用 Objective-C 方法

使用 Creator 打包的 iOS / Mac 原生应用中，我们也提供了在 iOS 和 Mac 上 JavaScript 通过原生语言的反射机制直接调用 Objective-C 函数的方法，示例代码如下：

```js
var result = jsb.reflection.callStaticMethod(className, methodName, arg1, arg2, .....);
```

在 `jsb.reflection.callStaticMethod` 方法中，我们通过传入 OC 的类名，方法名，参数就可以直接调用 OC 的静态方法，并且可以获得 OC 方法的返回值。注意仅仅支持调用可访问类的静态方法。

**警告**：苹果 App Store 在 2017 年 3 月对部分应用发出了警告，原因是使用了一些有风险的方法，其中 `respondsToSelector:` 和 `performSelector:` 是反射机制使用的核心 API，在使用时请谨慎关注苹果官方对此的态度发展，相关讨论：[JSPatch](https://github.com/bang590/JSPatch/issues/746)、[React-Native](https://github.com/facebook/react-native/issues/12778)、[Weex](https://github.com/alibaba/weex/issues/2875)

## 类

- 参数中的类名，只需要传入 OC 中的类名即可，与 Java 不同，类名并不需要路径。比如你在工程底下新建一个类 `NativeOcClass`，只要你将他引入工程，那么他的类名就是 `NativeOcClass`，你并不需要传入它的路径。

  ```
  import <Foundation/Foundation.h>
  @interface NativeOcClass : NSObject
  +(BOOL)callNativeUIWithTitle:(NSString *) title andContent:(NSString *)content;
  @end
  ```

## 方法

- JS 到 OC 的反射仅支持 OC 中类的静态方法。

- 方法名比较需要注意。我们需要传入完整的方法名，特别是当某个方法带有参数的时候，需要将它的 **:** 也带上。根据下面的例子，此时的方法名是 **callNativeUIWithTitle:andContent:**，不要漏掉了中间的 **:**。

  ```
  +(BOOL)callNativeUIWithTitle:(NSString *)title andContent:(NSString *)content;
  ```

- 如果是没有参数的函数，那么就不需要 **:**。如下面代码中的方法名是 `callNativeWithReturnString`，由于没有参数，就不需要 **:**，跟 OC 的 method 写法一致。

  ```
  +(NSString *)callNativeWithReturnString;
  ```

## 使用示例

- 下面的示例代码将调用上面 `NativeOcClass` 的方法，在 JS 层我们只需要这样调用：

  ```js
  var ret = jsb.reflection.callStaticMethod("NativeOcClass",
                                           "callNativeUIWithTitle:andContent:",
                                           "cocos2d-js",
                                           "Yes! you call a Native UI from Reflection");
  ```

- 这里是这个方法在 OC 的实现，可以看到是弹出一个原生对话框。并把 `title` 和 `content` 设置成你传入的参数，并返回一个 boolean 类型的返回值。

  ```
  +(BOOL)callNativeUIWithTitle:(NSString *) title andContent:(NSString *)content{
    UIAlertView *alertView = [[UIAlertView alloc] initWithTitle:title message:content delegate:self cancelButtonTitle:@"Cancel" otherButtonTitles:@"OK", nil];
    [alertView show];
    return true;
  }
  ```

- 此时，你就可以在 `ret` 中接收到从 OC 传回的返回值（true）了。

## Objective-C 执行 JS 代码

反过来，我们也可以通过 `evalString` 在 C++ / Objective-C 中执行 JavaScript 代码。

比如：

```c++
Application::getInstance()->getScheduler()->performFunctionInCocosThread([=](){
    se::ScriptEngine::getInstance()->evalString(script.c_str());
});
```

这里需要注意：除非明确当前线程是 **主线程**，否则都需要将函数分发到主线程执行。

## 注意

在 OC 的实现中，如果方法的参数需要使用 float、int、bool 的，请使用如下类型进行转换：

- **float、int 请使用 NSNumber 类型**

- **bool 请使用 BOOL 类型**

- 例如下面代码，我们传入两个浮点数，然后计算它们的合并返回，我们使用 NSNumber 而不是 int、float 去作为参数类型。

  ```
  +(float) addTwoNumber:(NSNumber *)num1 and:(NSNumber *)num2{
      float result = [num1 floatValue]+[num2 floatValue];
      return result;
  }
  ```

- 目前参数和返回值支持 **int**、**float**、**bool**、**string**，其余的类型暂时不支持。