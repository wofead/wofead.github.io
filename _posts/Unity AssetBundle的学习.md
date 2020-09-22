# Unity的资源管理--Assets

[toc]

## 一、什么是Assets

对于Assets，一般来说我们有两层认知：一层来自于Unity的默认工程目录Assets，一层来自于Unity的打包系统AssetBundles。那么我们就从这两个方面来归一化地去理解Unity的Assets究竟是什么。

### 1. Assets目录

当我们用Unity创建一个工程的时候，可以发现默认目录有四个：

* Assets
* Library
* Packages
* ProjectSetting

一共有4个目录，他们的作用分别是：

**Assets**

Unity工程实际的资源目录，所有项目用到的资源、代码、配置、库等原始资源只有放置在这个文件夹才会被Unity认可和处理。

****

**Library**

存放Unity处理完毕的资源，大部分的资源导入到Assets目录之后，还需要通过Unity转化成Unity认可的文件，转化后的文件会存储在这个目录。

---

**Packages**

这个是2018以后新增的目录，用于管理Unity分离的packages组件。同时，Unity的引擎在工作目录里也是没法对它进行操作的，是一个只读的目录。在以往的版本中是不存在这个目录的，因为Unity的核心代码越来越多，Unity开始了“减负”，将所有能从核心库里剥离的功能都剥离出来，然后以“Packages”的方式进行引用。

既然Packages目录是一个只读的目录，那么如何去管理它呢？Unity提供了一个Package Manager窗口，在Window菜单栏 - > Package Manger中。

打开之后我们可以看到左侧有一大串插件列表，前面的√表示是项目中现在这个插件是最新的，有下载箭头的表示有新版本可供下载使用，什么都没有表示你的项目中没有引用这个插件。

那我们安装好插件之后插件究竟在哪里呢？我们看到Packages目录下，就存放着我们插件。在Unity中，我们看到的就是具体的插件实体，当我们取文件管理器中看的时候发现只是一个manifest.json文件，真正的文件在Library中的packageCache文件夹中，这也就意味着我们在做项目同步的时候，每次这些Packages都是重新下载的。

有两类manifest文件：**project manifests** (`manifest.json`)和**package manifests** (`package.json`)。两者都使用JSON语法和包管理器(Package Manager)沟通，前者描述当前项目有哪此可用的包，后者描述当前包里面有什么。

---

**Packages**

这个目录用于存放Unity的各种项目设定。

---

当你的工程开始有脚本文件的时候，还会增加一个obj的目录用于代码编译。所以当你要迁移一个工程，或者将工程复制给别人的时候，只需要将Assets、Packages以及ProjectSettings三个目录备份即可，至于Library会在Unity打开的时候进行检查和自动转化。当然如果你的工程非常庞大，资源非常多，那么迁移的时候连Library一起拷贝传递会节省大量的转化时间。

**我们在分享Unity项目的时候，只提交这三个目录就可以了。但是在项目开发过程中，还是全部提交会好一点。**

另外，实际的项目开发期间，我们可能要针对不同的平台进行打包，比如PC包和安卓包。那么复制两份工程，一份设置为PC平台，一份设置为安卓平台，效率会远远大于需要时再切换平台，因为你每次切换到不同的平台，Unity都需要全部重新处理一遍内置资源，非常耗时。

一般我们需要三份工程，Android、IOS以及PC，前面两个用来打包，后面那个可以用来调试，发现bug。

### 2. AssetBundles

抛开所有其它的理解，单从英文命名来看，这是一种捆绑包，是对Asset进行归档的格式，概念更趋向于我们使用Zip或者RAR等格式对资源或者目录进行压缩、加密、归档、存储等等。而区别就在于Zip等压缩格式是针对文件的，而AssetBundles则是针对Unity的Asset。但如果再转换一下概念来理解，其实Zip操作归档的是操作系统能识别的文件，而AssetBundles操作归档的则是Unity能识别的文件。这么理解，二者的作用几乎是一致的。

Unity官方的定义：

> AssetBundles是一个包含了特殊平台、非代码形式Assets的归档文件。

这里有几个重要信息，首先它是一个归档文件（即捆绑形式的文件类型），其次它拥有平台的差异性，再次它不包含代码，最后它存储的是Unity的Assets。



**到目前为止，我们一直在提及Assets，那么究竟Assets是什么呢？以及它在Unity整个引擎中占据的位置又是什么呢？**

### 3. Unity资产

Windows操作系统识别文件是通过后缀名实现的。在系统中注册后缀名和对应的处理软件，那么双击文件的时候系统就会调用指定的软件解析和处理文件。如果没有在系统中注册，或者后缀被删除了，那么操作系统将无法识别这个文件。

Unity的Asset也是一样，我们把一个Asset叫做一个资产，可以理解为Unity能够识别的文件。这里其实又包含两种类型，一种是Unity原生支持的格式，比如：材质球；一种是需要经过Unity处理之后才能支持的，比如：FBX。对于需要处理才能支持的格式，Unity都提供了导入器（Importer）。

要注意，所有的资产原始文件都必须要放在Unity工程的Assets目录，然后经过Unity处理之后存放在Library目录下。

## 二、 Assets的识别和引用

作为资产文件，Assets有非常多的类型。比如：材质球、纹理贴图、音频文件、FBX文件、各种动画、配置或者Clip文件等等。我们通常习惯于在Unity里进行拖拽、新增、修改、重命名甚至变更目录等等各式各样的操作，但不管你在Unity引擎里如何操作（不包括删除），那些相关的引用都不会丢失。这是为什么呢？

### 1. Assets和Objects

在进行后面的阐述之前，先统一一下概念，包括如果在后面章节提到，都会遵循这里统一的概念。Assets这里以及后续的内容都指Unity的资产，可以意指为Unity的Projects窗口里看到的单个文件(或者文件夹)。而Objects这里我们指的是从UnityEngine.Object继承的对象，它其实是一个可以序列化的数据，用来描述一个特定的资源的实例。它可以代表任何Unity引擎所支持的类型，比如：Mesh、Sprite、AudioClip or AnimationClip等等。

大多数的Objects都是Unity内置支持的，但有两种除外：

**ScriptableObject**

用来提供给开发者进行自定义数据格式的类型。从该类继承的格式，都可以像Unity的原生类型一样进行序列和反序列化，并且可以从Unity的Inspector窗口进行操作。

---

**MonoBehaviour**

提供了一个指向MonoScript的转换器。MonoScript是一个Unity内部的数据类型，它不是可执行代码，但是会在特定的命名空间和程序集下，保持对某个或者特殊脚本的引用。

---

Assets和Objects之间是一对多的关系，比如一个Prefab我们可以认为是一个Asset，但是这个Prefab里可以包含很多个Objects，比如：如果是一个UGUI的Prefab，里面就可能会有很多个Text、Button、Image等组件。

### 2. File GUIDs 和 Local IDs

熟悉Unity的人都知道，UnityEngine.Objects之间是可以互相引用的。这就会存在一个问题，这些互相引用的Objects有可能是在同一个Assets里，也有可能是在不同的Assets里。比如：UGUI的一个Image需要引用一张Sprite Atlas里的Sprite，这就要求Unity必须有健壮的资源标识，能稳定地处理不同资源的引用关系。除此之外，Unity还必须考虑这些资源标识应该与平台无关，不能让开发者在切换平台的时候还需要关注资源的引用关系，毕竟它自己是一个跨平台部署的引擎。

基于这些特定的需求，Unity把序列化拆分成两个表达部分。第一部分叫做File GUID，标识这个资产的位置，这个GUID是由Unity根据内部算法自动生成的，并且存放在原始文件的同目录、同名但是后缀为.meta的文件里。

这里需要注意一下几点：

* 第一次导入资源的时候Unity会自动生成。
* 在Unity的面板里移动位置，Unity会自动帮你同步.meta文件。
* 在Unity打开的情况下，单独删除.meta，Unity可以确保重新生成的GUID和现有的一样。
* 在Unity关闭的情况下，移动或者删除.meta文件，Unity无法恢复到原有的GUID，也就是说引用会丢失。**这也是导致.meta文件经常冲突的原因，可能不是删除，而是策划直接将资源拷到目录中，而不是通过Unity导入其中。**

确定了资产文件之后，还需要一个Local IDs来表示当前的Objects在资产里的唯一标识。File GUID确保了资产在整个Unity工程里唯一，Local ID确保Objects在资产里唯一，这样就可以通过二者的组合去快速找到对应的引用。

Unity还在内部维护了一张资产GUID和路径的映射表，每当有新的资源进入工程，或者删除了某些资源，又或者调整了资源路径，Unity的编辑器都会自动修改这张映射表以便正确地记录资产位置。所以如果.meta文件丢失或者重新生成了不一样的GUID，Unity就会丢失引用，在工程内的表现就是某个脚本显示“Missing”，或者某些贴图材质的丢失导致场景出现粉红色。

### 3. Library的资源位置

前面我们提到了非Unity支持的格式，需要由导入器进行资源转换。之所以要分到这个小节来讲位置是因为它涉及到了File GUID。之所以需要对资源转换和存储，也是为了方便下一次启动的时候不需要再处理资源，比较每次导入资源是巨耗时的操作。简单来讲，所有的转换结果都会存储在Library/metadata/目录下，以File GUID的前两位进行命名的文件夹里。

**注意：原生支持的Assets资源也会有同样的存储过程，只是不需要再用导入器进行转化而已。**

### 4. Instance ID

File GUID和Local ID确实已经能够在编辑器模式下帮助Unity完成它的规划了，与平台无关、快速定位和维护资源位置以及引用关系。但若投入到运行时，则还有比较大的性能问题。也就是说运行时还是需要一个表现更好的系统。于是Unity又弄了一套缓存（还记得前面那套缓存嘛？是用来记录GUID和文件的路径关系的）。PersistentManager用来把File GUIDs和Local IDs转化为一个简单的、Session唯一的整数，这些整数就是Instance ID。Instance ID很简单，就是一个递增的整数，每当有新对象需要在缓存里注册的时候，简单的递增就行。

PersistentManager会维护Instance ID和File GUID、Local ID的映射关系，定位Object源数据的位置以及维护内存中（如果有）Object的实例。只要系统解析到一个Instance ID，就能快速找到代表这个Instance ID的已加载的对象。如果Object没有被加载，File GUID和Local ID也可以快速地定位到指定的Asset资源从而即时进行资源加载。



https://blog.uwa4d.com/archives/USparkle_inf_UnityEngine.html

https://docs.unity3d.com/Manual/AssetBundlesIntro.html

https://blog.uwa4d.com/archives/USparkle_Addressable1.html

[浅谈Assets -- Unity资源映射]: https://blog.uwa4d.com/archives/USparkle_Addressable1.html

