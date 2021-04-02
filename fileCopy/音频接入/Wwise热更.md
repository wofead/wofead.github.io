# Wwise热更新

## 加载`AkTerminator、AkInitializer`两个脚本

将这两个脚本制作成Prefab，音频初始化时实例化这个prefab。优点：所有初始化参数都可以配置并热更。缺点：需要在初始化Wwise时进行prefab的实例化操作。

`AkInitializer`：Wwise的初始化。

`AkTerminator`避免添加多个重复组件，原因是拥有这个标签。`[UnityEngine.DisallowMultipleComponent]`。

## 注册全局声音对象，用于所有2D声音的播放

我们通常会自己管理一个对象播放所有2D声音（音乐、环境、UI）。这个对象不需要同步坐标，也省去了频繁注册注销SoundObject的损耗。这个步骤能够节省大量的CPU资源。

## 加载需要在游戏开始就要用到的SoundBank（比如音量设置、UI、音乐等）

Wwise使用string作为LoadBank的参数，有三种方式可以实现这个需求：

1. 在lua中保存一个字段记录要加载的SoundBank Name，调用C#接口加载。
2. 做一个.asset配置文件，C#从配置文件中读取要加载的SoundBank Name。
3. 将加载SoundBank操作做成prefab，在初始化时实例化prefab。

初始化时要加载的SoundBank通常需要设计师修改，第三种方式对设计师最友好，但是也需要prefab的实例化操作。

## 音量的初始化设置

关于音乐、音效、语音的开关及音量的设置应该保存在客户端本地，并在Wwise播出第一个声音前设置正确的参数。设计师一般会将Mute、Unmute（开关）操作封装成Event，BusVolume（音量）设置封装成RTPC，然后打包为SoundBank交给程序调用。所以这一步骤依赖LoadBank的完成。一般情况下项目使用的LoadBank方法是异步的。所以这里需要把跟这个功能有关的SoundBank单独拆出来，使用同步的方式加载，或者通过回调确定LoadBank已完成。

以上所有操作应该在C#内被封装成一个方法，让lua在合适的时机去调用。

## 关于热更：

Wwise提供了两个API用于设置SoundBank路径。分别是SetBasePath和AddBasePath。SetBasePath设置基础目录，AddBasePath设置后续DLC目录。Wwise允许设置多个DLC目录，LoadBank时会从最后一次AddBasePath的路径依据SoundBank的文件名开始搜索，依次向前最后到SetBasePath的路径，搜索到第一个目标SoundBank后加载。

**（注意，AddBasePath不是全平台都有效。在PC下是无效的，所以不能在PC平台测试。）**

在iOS和Android平台：

SetBasePath的默认路径为：

`Application.streamingAssetsPath/Audio/GeneratedSoundBanks/(Platform)`

AddBasePath的默认路径为：

`Application.persistentDataPath`

一般我们会把这里改为：

`Application.persistentDataPath/Audio/GeneratedSoundBanks/(Platform)`



```c#
#if UNITY_IPHONE            

    string fileNameBase = Application.dataPath.Substring(0, Application.dataPath.LastIndexOf('/'));

    fileName = fileNameBase.Substring(0, fileNameBase.LastIndexOf('/')) + "/Documents/" + FILE_NAME;

#elif UNITY_ANDROID

    fileName = Application.persistentDataPath + "/" + FILE_NAME ;

#else

    fileName = Application.dataPath + "/" + FILE_NAME;

#endif
```

