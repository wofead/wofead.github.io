---
layout:     post
title:      Android JNI
subtitle:   NDK与JNI基础
date:       2019-7-23
author:     Jow
header-img: img/lua-home-bg-o.jpg
catalog: 	 true 
tags:
    - Android
    - NDK
    - JNI

---

### 目录
1. 导读
2. NDK
3. JNI
4. JNI原理
5. JNI的引用
6. JNI的简单实用
7. Cmake工具demo的背后原理
8. Java与Native的相互调用
9. JNI中的签名
10. 回顾和总结


> If you feel you are sleepy, please have a rest. Then hard work.

要想搞清楚什么是JNI，我们需要将与之相管的内容全部整理出来：

![](https://i.imgur.com/puooyEc.png)

## 导读
> 在Android开发中，Google提供了两种开发包，SDK和NDK，其中SDK是用来Java开发的，而NDK是用来c/c++开发的，即JNI编程方式，第三方应用可以通过JNI调用自己的c动态库，这就是NDK。

## NDK
NDK（Native Develop Kit):是一套允许你使用原生代码语言，例如（c和c++）实现部分应用的工具集。在开发某些类型应用时，这有助于您重复使用这些语言编写的代码库。一般情况下NDK工具把C/C++编译为.co文件，然后再Java中调用。
1. NDK可以提升应用性能(C/C++)
2. 平台之间移植
3. 使用第三方库
4. 代码保护(不容易被反编译)

### NDK到so
![](https://i.imgur.com/iTApdOR.png)

so库，即将C或者C++实现的功能进行打包，将其打包为共享库，让其他程序进行调用，这可以提高代码的复用性。linux生成的共享库以so结尾，在win下是dll结尾。

目前Android系统支持下面其中CPU结构，每一种对应着各自的应用程序二进制接口ABI（Application Binary Interface）：定义了二进制文件（尤其是共享库文件）如何在相应的系统平台运行，从使用的指令集内存对齐以及可用的系统函数库。
1. ARMv5 - armeabi
2. ARMv7 - armeabi-v7a
3. ARMv7 - arm64-v8a
4. x86 - x86
5. MPS - mips
6. MIPS64 - mips64
7. x86_64 - x86_64

## JNI
JNI（Java Native Interface）,即Java本地接口。JNI是Java调用Native 语言的一种特性。通过JNI可以使得Java与C/C++机型交互。由于JNI是JVM中规范的一部分，因此可以是我们可以复用以前用C/C++写的大量代码。JNI是一种弄在Java虚拟机机制下的执行代码的标准机制。
在使用JNI的时候，注意平台的跨建。如果要实现跨平台，就必须将本地代码在不同的操作系统平台下编译出相应的动态库。

![](https://i.imgur.com/SQJYK4M.png)

JNI的命名规则：

```java
JNIExport jstring JNICALL Java_com_example_hellojni_MainActivity_stringFromJNI( JNIEnv* env,jobject thiz ) 
```
jstring 是返回值类型

Java_com_example_hellojni 是包名

MainActivity 是类名

stringFromJNI 是方法名

其中JNIExport和JNICALL是不固定保留的关键字不要修改

如何实现JNI：
1. 在Java中先声明一个native方法
2. 编译Java源文件javac得到.class文件
3. 通过javah -jni命令导出JNI的.h头文件
4. 使用Java需要交互的本地代码，实现在Java中声明的Native方法(如果Java需要与C++交互，那么就用C++实现Java的Native方法)
5. 将本地代码编译成动态库
6. 通过Java命令执行Java程序，最终实现Java调用本地代码。

```java
jdouble Java_pkg_Cls_f__ILjava_lang_String_2 (JNIEnv *env, jobject obj, jint i, jstring s)
{
     const char *str = (*env)->GetStringUTFChars(env, s, 0); 
     (*env)->ReleaseStringUTFChars(env, s, str); 
     return 10;
}
```
* *env：一个接口指针
* obj：在本地方法中声明的对象引用
* i和s：用于传递的参数

JNI有自己的原始数据类型和数据引用类型如下：

![](https://i.imgur.com/OManwWT.png)

## JNI原理
> 在计算机系统中，每一种编程语言都有一个执行环境，执行环境来解释执行执行语言中的语句。

Java语言的执行环境是Java虚拟机（JVM），JVM其实是主机环境中的一个进程，每个JVM都在本地环境中有一个JavaVM结构体，该结构体在创建虚拟机的时候被返回。在JNI环境下创建JVM的函数为JNI_CreateJavaVM。

```java
JNI_CreateJavaVM(JavaVM **pvm, void **penv, void*args);
```
![](https://i.imgur.com/KGoZ9a1.png)

其中JavaVM是JVM在JNI层的代表，JNI全局仅仅有一个JavaVM结构，在里面封装了一些函数指针(or函数表结构)，JavaVm中封装的这些函数指针主要是对JVM操作接口。另外c和c++中的JavaVM的定义有所不同，在c中JavaVM是JNIInvokeInterface_类指针，而在C++中，进行了一次封装，比c中少一个参数。这也是为什么推荐使用c++来编写JNI函数。

JNIEnv：

JINEnv是当前Java线程的执行环境，一个JVM对应一个JavaVM结构体，一个JVM中可能创建多个Java线程，每个线程对应一个JNIEnv结构，它们保存在线程本地存储TLS中。因此不同的线程JNIEnv不同，而不能相互共享使用。
JavaEnv结构也是一个函数表，在本地代码通过JNIEnv函数表来操作Java数据或者调用Java方法。也就是说，只要在本地代码中拿到了JNIEnv结构，就可以在本地代码中调用Java代码。


![](https://i.imgur.com/e5UiHQp.png)

* 调用Java 函数：JNIEnv代表了Java执行环境，能够使用JNIEnv调用Java中的代码
* 操作Java代码：Java对象传入JNI层就是jobject对象，需要使用JNIEnv来操作这个Java对象

JNIEnv的创建：
* C 中——JNIInvokeInterface：JNIInvokeInterface是C语言环境中的JavaVM结构体，调用 (AttachCurrentThread)(JavaVM, JNIEnv*, void) 方法，能够获得JNIEnv结构体
* C++中 ——_JavaVM：_JavaVM是C++中JavaVM结构体，调用jint AttachCurrentThread(JNIEnv** p_env, void* thr_args) 方法，能够获取JNIEnv结构体；


JNIEnv的释放：
* C 中释放：调用JavaVM结构体JNIInvokeInterface中的(DetachCurrentThread)(JavaVM)方法，能够释放本线程的JNIEnv
* C++ 中释放：调用JavaVM结构体_JavaVM中的jint DetachCurrentThread(){ return functions->DetachCurrentThread(this); } 方法，就可以释放 本线程的JNIEnv

JNIEnv和线程的关系：
* JNIEnv只在当前线程有效：JNIEnv仅仅在当前线程有效，JNIEnv不能在线程之间进行传递，在同一个线程中，多次调用JNI层方便，传入的JNIEnv是同样的
* 本地方法匹配多个JNIEnv：在Java层定义的本地方法，能够在不同的线程调用，因此能够接受不同的JNIEnv

JINEnv相关常用的函数：
```java
jobject NewObject(JNIEnv *env, jclass clazz,jmethodID methodID, ...)：
jobject NewObjectA(JNIEnv *env, jclass clazz,jmethodID methodID, const jvalue *args)：
jobject NewObjectV(JNIEnv *env, jclass clazz,jmethodID methodID, va_list args)：

jstring NewString(JNIEnv *env, const jchar *unicodeChars,jsize len)：
ArrayType New<PrimitiveType>Array(JNIEnv *env, jsize length);
jobjectArray NewObjectArray(JNIEnv *env, jsize length,jclass elementClass, jobject initialElement);
jobject GetObjectArrayElement(JNIEnv *env,jobjectArray array, jsize index);
jsize GetArrayLength(JNIEnv *env, jarray array);
```
第一个参数jclass class 代表的你要创建哪个类的对象，第二个参数,jmethodID methodID代表你要使用那个构造方法ID来创建这个对象。只要有jclass和jmethodID，我们就可以在本地方法创建这个Java类的对象。

通过Unicode字符的数组来创建一个新的String对象。

env是JNI接口指针；unicodeChars是指向Unicode字符串的指针；len是Unicode字符串的长度。返回值是Java字符串对象，如果无法构造该字符串，则为null。

## JNI的引用

Java内存管理这块是完全透明的，new一个对象或者释放一个对象，都是Java自己进行管理的，但是从Java虚拟机创建的对象传到C/C++代码就会产生引用，根据Java的垃圾回收机制，只要存在引用就不会对对象进行垃圾回收。

在JNI规范中定义了三种引用：局部引用、全局引用和弱全局引用。

1. 局部引用(Local Reference)

局部引用，也成本地引用，通常是在函数中创建并使用。会阻止GC回收所有引用对象。这个是最常见的引用类型，基本上通过JNI返回来的引用都是局部引用，例如使用newobject,就会返回创建出来的是实例的局部引用，局部引用值在改native函数中有效，会在函数返回的时候自动释放，也可以使用DeleteLocalRef函数手动释放该对象。

2. 全局引用(Global Reference)

全局引用可以跨方法、跨线程使用，直到被开发者显式释放。类似局部引用，一个全局引用在被释放前保证引用对象不被GC回收。和局部应用不同的是，没有俺么多函数能够创建全局引用。能创建全部引用的函数只有NewGlobalRef，而释放它需要使用ReleaseGlobalRef函数
3. 弱全局引用(Weak Global Reference)

是JDK 1.2 新增加的功能，与全局引用类似，创建跟删除都需要由编程人员来进行，这种引用与全局引用一样可以在多个本地代码有效，不一样的是，弱引用将不会阻止垃圾回收期回收这个引用所指向的对象，所以在使用时需要多加小心，它所引用的对象可能是不存在的或者已经被回收。
4. 引用比较

给定两个引用，我们可以通过下面的代码来判断是否是相同的引用。如果obj1和obj2指向相同的对象，则返回JNI_TRUE(或者1)，否则返回JNI_FALSE(或者0).

```java
(*env)->IsSameObject(env, obj1, obj2)
```

attention:有一个特殊的引用需要注意：NULL，JNI中的NULL引用指向JVM中的null对象，如果obj是一个全局或者局部引用，使用(*env)->IsSameObject(env, obj, NULL)或者obj == NULL用来判断obj是否指向一个null对象即可。但是需要注意的是，IsSameObject用于弱全局引用与NULL比较时，返回值的意义是不同于局部引用和全局引用的。代码如下：

```java
jobject local_obj_ref = (*env)->NewObject(env, xxx_cls,xxx_mid); 
jobject g_obj_ref = (*env)->NewWeakGlobalRef(env, local_ref);
// ... 业务逻辑处理
jboolean isEqual = (*env)->IsSameObject(env, g_obj_ref, NULL);
```

## JNI的简单实用
实现JNI的方法用两种，第一种为传统方法，非常的复杂，这里就不做演示了，我们主要介绍第二种，实用CMAKE的方式。

在创建Android项目的时候选择Include C++ Support，里面有三个项目，我们选择第一个就行了：
* C++ Standard：即C++标准，使用下拉列表选择你希望使用的C++的标准，选择Toolchain Default 会使用默认的CMake设置。

* Exceptions Support：如果你希望启用对C++异常处理的支持，请选择此复选框。如果启动此复选框，Android Studio 会将-fexceptions标志添加到模块级build.gradle文件的cppFlags中，Gradle会将其传递到CMake。

* Runtime Type Information Support：如果开发者希望支持RTTI，请选中此复选框。如果启用此复选框，Android Studio 会将-frtti标志添加到模块级build.gradle文件的cppFlags中，Gradle会将其传递到CMake。

Android Studio 将添加cpp和External Build Files 组：

* 在 cpp 文件夹中：可以找到属于项目的所有原生源文件等构建库。对于新项目，Android Studio会创建一个示例C++源文件 native-lib.cpp，并将其置于应用模块src/main/cpp/目录中。这个示例代码提供了一个简单的C++函数stringFromJNI()，此函数可以返回字符串“Hello from C++”

* 在 External Build Files 文件夹中：可以找到CMake或 ndk-build 的构建脚本。与build.gradle文件指示Gradle构建应用一样，CMake和ndk-build需要一个构建脚本来了解如何构原生库。对于新项目，Android Studio 会创建一个CMake 构建脚本CMakeLists.txt，并将其置于模块根目录中。

## Cmake工具demo的背后原理

CMake的入口：build.gradle配置文件中。

在defaultConfig中会有：

```
externalNativeBuild {
    cmake {
        cppFlags ""
    }
}
```
在defaultConfig外面会有

```
 externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
        }
    }
```

> * 在defaultConfig外面的externalNativeBuild里面的cmake指明了CMakeLists.txt的路径(在本项目下，和是build.gradle在同一个目录里面)。
> * 在defaultConfig里面的externalNativeBuild里面的cmake主要填写的是CMake的命令参数。即由arguments中的参数最后转化成一个可执行的CMake的命令

在CMakeLists.txt文件中包含以下四个部分：

![](https://i.imgur.com/AFiIUIf.png)

> * cmake_minimum_required(VERSION 3.4.1)：指定CMake的最小版本
> * add_library：创建一个静态或者动态库，并提供其关联的源文件路径，开发者可以定义多个库，CMake会自动去构建它们。Gradle可以自动将它们打包进APK中。第一个参数——native-lib：是库的名称；第二个参数——SHARED：是库的类别，是动态的还是静态的；第三个参数——src/main/cpp/native-lib.cpp：是库的源文件的路径。
> * target_link_libraries：指定CMake链接到目标库。开发者可以链接多个库，比如开发者可以在此定义库的构建脚本，并且预编译第三方库或者系统库。第一个参数——native-lib：指定的目标库；第一个参数——${log-lib}：将目标库链接到NDK中的日志库。

上面只是CMake的基本功能，CMake里面可以非常强大，有兴趣可以研究一下CMake手册。

## Java与Native的相互调用
1. 注册native函数
2. JNI中的签名
3. native代码反调用Java层代码

![](https://i.imgur.com/uVwX1id.png)

首先是native函数的注册：

* 静态注册：

先有java得到本地方法的声明，然后再通过JNI实现该声明方法
* 动态注册：

先通过JNI重载JNI_OnLoad()实现本地方法，然后直接在Java中调用本地方法。


静态方法注册：首先在Java代码中声明native函数，然后在c文件中实现方法。

```java
public class JniDemo1{
       static {
             System.loadLibrary("native-lib");
        }

        public native String stringFromJNI();
		public native  int integerFromJNI();
}

extern "C" JNIEXPORT jstring JNICALL
Java_com_example_jnitest_MainActivity_stringFromJNI(
        JNIEnv* env,
        jobject /* this */) {
    std::string hello = "Hello from C++";
    return env->NewStringUTF(hello.c_str());
}

extern "C"
JNIEXPORT jint JNICALL
Java_com_example_jnitest_MainActivity_integerFromJNI(JNIEnv *env, jobject instance) {
    int a = 23;
    return a;
}

```

动态方法注册：

> 上面我们介绍了静态注册native方法的过程，就是Java层声明的nativ方法和JNI函数一一对应。以我来说，刚开始做JNI的前期，可能会遵守静态注册的流程：1、编写带有native方法的Java类。 2、编写代码实现C文件中的方法，这样的单调的标准流程，而且还要忍受这么"长"的函数名。那有没有更简单的方式呢？比如让Java层的native方法和任意JNI函数连接起来？答案是有的——动态注册，也就是通过RegisterNatives方法把C/C++中的方法映射到Java中的native方法，而无需遵循特定的方法命名格式。

当我们使用System.loadLibarary()方法加载so库的时候，Java虚拟机就会找到这个JNI_OnLoad函数兵调用该函数，这个函数的作用是告诉Dalvik虚拟机此C库使用的是哪一个JNI版本，如果你的库里面没有写明JNI_OnLoad()函数，VM会默认该库使用最老的JNI 1.1版本。由于最新版本的JNI做了很多扩充，也优化了一些内容，如果需要使用JNI新版本的功能，就必须在JNI_OnLoad()函数声明JNI的版本。同时也可以在该函数中做一些初始化的动作，其实这个函数有点类似于Android中的Activity中的onCreate()方法。该函数前面也有三个关键字分别是JNIEXPORT，JNICALL ，jint。其中JNIEXPORT和JNICALL是两个宏定义，用于指定该函数时JNI函数。jint是JNI定义的数据类型，因为Java层和C/C++的数据类型或者对象不能直接相互的引用或者使用，JNI层定义了自己的数据类型，用于衔接Java层和JNI层

> 与JNI_OnLoad()函数相对应的有JNI_OnUnload()函数，当虚拟机释放的该C库的时候，则会调用JNI_OnUnload()函数来进行善后清除工作。

该函数会有两个参数，其中*jvm为Java虚拟机实例，JavaVM结构体定义一下函数：

```
DestroyJavaVM
AttachCurrentThread
DetachCurrentThread
GetEnv
```

首先加载共享库：

```java
public class JniDemo1{
       static {
             System.loadLibrary("samplelib_jni");
        }
}
```

接下来在c中实现：

```
jint JNI_OnLoad(JavaVM* vm, void* reserved)
```

```c
#include <jni.h>
#include "Log4Android.h"
#include <stdio.h>
#include <stdlib.h>

using namespace std;

#ifdef __cplusplus
extern "C" {
#endif

static const char *className = "com/gebilaolitou/jnidemo/JNIDemo2";

static void sayHello(JNIEnv *env, jobject, jlong handle) {
    LOGI("JNI", "native: say hello ###");
}

static JNINativeMethod gJni_Methods_table[] = {
    {"sayHello", "(J)V", (void*)sayHello},
};

static int jniRegisterNativeMethods(JNIEnv* env, const char* className,
    const JNINativeMethod* gMethods, int numMethods)
{
    jclass clazz;

    LOGI("JNI","Registering %s natives\n", className);
    clazz = (env)->FindClass( className);
    if (clazz == NULL) {
        LOGE("JNI","Native registration unable to find class '%s'\n", className);
        return -1;
    }

    int result = 0;
    if ((env)->RegisterNatives(clazz, gJni_Methods_table, numMethods) < 0) {
        LOGE("JNI","RegisterNatives failed for '%s'\n", className);
        result = -1;
    }

    (env)->DeleteLocalRef(clazz);
    return result;
}

jint JNI_OnLoad(JavaVM* vm, void* reserved){
    LOGI("JNI", "enter jni_onload");

    JNIEnv* env = NULL;
    jint result = -1;

    if (vm->GetEnv((void**) &env, JNI_VERSION_1_4) != JNI_OK) {
        return result;
    }

    jniRegisterNativeMethods(env, className, gJni_Methods_table, sizeof(gJni_Methods_table) / sizeof(JNINativeMethod));

    return JNI_VERSION_1_4;
}

#ifdef __cplusplus
}
#endif
```

```
if (vm->GetEnv((void**) &env, JNI_VERSION_1_4) != JNI_OK) {
    return result ;
}
```

这里调用了GetEnv函数时为了获取JNIEnv结构体指针，其实JNIEnv结构体指向了一个函数表，该函数表指向了对应的JNI函数，我们通过这些JNI函数实现JNI编程。

然后就调用了jniRegisterNativeMethods函数来实现注册，这里面注意一个静态变量gJni_Methods_table。它其实代表了一个native方法的数组，如果你在一个Java类中有一个native方法，这里它的size就是1，如果是两个native方法，它的size就是2，大家看下我这个gJni_Methods_table变量的实现:

```c
static JNINativeMethod gJni_Methods_table[] = {
    {"sayHello", "(J)V", (void*)sayHello},
};
```

我们看到他的类型是JNINativeMethod ，那我们就来研究下JNINativeMethod:

> JNI允许我们提供一个函数映射表，注册给Java虚拟机，这样JVM就可以用函数映射表来调用相应的函数。这样就可以不必通过函数名来查找需要调用的函数了。Java与JNI通过JNINativeMethod的结构来建立联系，它被定义在jni.h中，其结构内容如下：
```c
typedef struct { 
    const char* name; 
    const char* signature; 
    void* fnPtr; 
} JNINativeMethod; 
```

* 第一个变量name，代表的是Java中的函数名
* 第二个变量signature，代表的是Java中的参数和返回值
* 第三个变量fnPtr，代表的是的指向C函数的函数指针

下面我们再来看下jniRegisterNativeMethods函数内部的实现:

首先通过clazz = (env)->FindClass( className);找到声明native方法的类

然后通过调用RegisterNatives函数将注册函数的Java类，以及注册函数的数组，以及个数注册在一起，这样就实现了绑定。

上面在讲解JNINativeMethod结构体的时候，提到一个概念，就是"signature"即签名，这个是什么东西？我们下面就来讲解下。

## JNI的签名

为什么JNI中突然多出了一个概念叫"签名"：

因为Java是支持函数重载的，也就是说，可以定义相同方法名，但是不同参数的方法，然后Java根据其不同的参数，找到其对应的实现的方法。这样是很好，所以说JNI肯定要支持的，那JNI要怎么支持那，如果仅仅是根据函数名，没有办法找到重载的函数的，所以为了解决这个问题，JNI就衍生了一个概念——"签名"，即将参数类型和返回值类型的组合。如果拥有一个该函数的签名信息和这个函数的函数名，我们就可以顺序的找到对应的Java层中的函数了。

如何查看签名呢：可以使用javap命令。

```
javap -s -p MainActivity.class

Compiled from "MainActivity.java"
public class com.example.hellojni.MainActivity extends android.app.Activity {
  static {};
    Signature: ()V

  public com.example.hellojni.MainActivity();
    Signature: ()V

  protected void onCreate(android.os.Bundle);
    Signature: (Landroid/os/Bundle;)V

  public boolean onCreateOptionsMenu(android.view.Menu);
    Signature: (Landroid/view/Menu;)Z

  public native java.lang.String stringFromJNI(); //native 方法
    Signature: ()Ljava/lang/String;  //签名

  public native int max(int, int); //native 方法
    Signature: (II)I    //签名
}
```

签名有哪些：

![](https://i.imgur.com/TORyJIO.png)

![](https://i.imgur.com/6sxxvKn.png)

native代码反调用Java层代码：

首先获取class对象：为了能够在C/C++中调用Java中的类，jni.h的头文件专门定义了jclass类型表示Java中Class类。JNIEnv中有3个函数可以获取jclass。

1. jclass FindClass(const char* clsName)：通过类的名称(类的全名，这时候包名不是用'"."点号而是用"/"来区分的)来获取jclass。比如:

```
jclass jcl_string=env->FindClass("java/lang/String");
```

2. jclass GetObjectClass(jobject obj)：通过对象实例来获取jclass，相当于Java中的getClass()函数
3. jclass getSuperClass(jclass obj)：通过jclass可以获取其父类的jclass对象

获取属性方法：

在Native本地代码中访问Java层的代码，一个常用的常见的场景就是获取Java类的属性和方法。所以为了在C/C++获取Java层的属性和方法，JNI在jni.h头文件中定义了jfieldID和jmethodID这两种类型来分别代表Java端的属性和方法。在访问或者设置Java某个属性的时候，首先就要现在本地代码中取得代表该Java类的属性的jfieldID，然后才能在本地代码中进行Java属性的操作，同样，在需要调用Java类的某个方法时，也是需要取得代表该方法的jmethodID才能进行Java方法操作。

常见的调用Java层的方法如下：

1. GetFieldID/GetMethodID：获取某个属性/某个方法
2. GetStaticFieldID/GetStaticMethodID：获取某个静态属性/静态方法

```c
jfieldID GetFieldID(JNIEnv *env, jclass clazz, const char *name, const char *sig);
jmethodID GetMethodID(JNIEnv *env, jclass clazz, const char *name, const char *sig);
jfieldID GetStaticFieldID(JNIEnv *env, jclass clazz, const char *name, const char *sig);
jmethodID GetStaticMethodID(JNIEnv *env, jclass clazz,const char *name, const char *sig);
```

大家发现什么规律没有？对了，我们发现他们都是4个入参，而且每个入参的都是*JNIEnv *env，jclass clazz，const char *name，const char *sig。关于JNIEnv，前面我们已经讲过了，这里我们就不详细讲解了，JNIEnv代表一个JNI环境接口，jclass上面也说了代表Java层中的"类"，name则代表方法名或者属性名。那最后一个char *sig代表什么？它其实代表了JNI中的一个特殊字段——签名，上面已经讲解过了。


### 构造一个对象

```
jobject NewObject(jclass clazz, jmethodID methodID, ...)
```

比如有我们知道Java类中可能有多个构造函数，当我们要指定调用某个构造函数的时候，会调用下面这个方法:

```
jmethodID mid = (*env)->GetMethodID(env, cls, "<init>", "()V");
obj = (*env)->NewObject(env, cls, mid);
```

* clazz:是需要创建的Java对象的Class对象
* methodID：是传递一个方法ID，想一想Java对象创建的时候，需要执行什么操作？就是执行构造函数。

有人会说这要走两行代码，有没有一行代码的，是有的，如下：

```c
jobject NewObjectA(JNIEnv *env, jclass clazz, 
jmethodID methodID, jvalue *args);
```

这里多了一个参数，即jvalue *args，这里是args代表的是对应构造函数的所有参数的，我们可以应将传递给构造函数的所有参数放在jvalues类型的数组args中，该数组紧跟着放在methodID参数的后面。NewObject()收到数组中的这些参数后，将把它们传给编程任索要调用的Java方法。

上面说到，参数是个数组，如果参数不是数组怎么处理，jni.h同样也提供了一个方法，如下：

```c
jobject NewObjectV(JNIEnv *env, jclass clazz, 
jmethodID methodID, va_list args);
```

这个方法和上面不同在于，这里将构造函数的所有参数放到在va_list类型的参数args中，该参数紧跟着放在methodID参数的后面。

JNI的常用方法API：[https://www.jianshu.com/p/67081d9b0a9c](https://www.jianshu.com/p/67081d9b0a9c)

例子：[https://www.jianshu.com/p/0f34c097028a](https://www.jianshu.com/p/0f34c097028a)

## 回顾和总结

要再c++层调用Java代码，首先要知道classname和方法名称：

* #define CLASS_NAME "com/lizhenda/lib/sdk/SDKHelper"
* SDKHelper::getMyInfo(LuaCallback callback)
* JniHelper::callStaticVoidMethod(CLASS_NAME, "getMyInfo");
* cocos2d::JniHelper::getStaticMethodInfo(t, className.c_str(), methodName.c_str(), signature.c_str())

```c++
t.env->CallStaticVoidMethod(t.classID, t.methodID, convert(t, xs)...);
t.env->DeleteLocalRef(t.classID);
deleteLocalRefs(t.env);
```

其中methodid和classid可以通过JNIEnv中的函数获取，传入classname，methodname以及签名。
