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

基于这些特定的需求，Unity把序列化拆分成两个表达部分。第一部分叫做**File GUID**，标识这个资产的位置，这个GUID是由Unity根据内部算法自动生成的，并且存放在原始文件的同目录、同名但是后缀为.meta的文件里。

这里需要注意一下几点：

* 第一次导入资源的时候Unity会自动生成。
* 在Unity的面板里移动位置，Unity会自动帮你同步.meta文件。
* 在Unity打开的情况下，单独删除.meta，Unity可以确保重新生成的GUID和现有的一样。
* 在Unity关闭的情况下，移动或者删除.meta文件，Unity无法恢复到原有的GUID，也就是说引用会丢失。**这也是导致.meta文件经常冲突的原因，可能不是删除，而是策划直接将资源拷到目录中，而不是通过Unity导入其中。**

确定了资产文件之后，还需要一个Local IDs来表示当前的Objects在资产里的唯一标识。File GUID确保了资产在整个Unity工程里唯一，Local ID确保Objects在资产里唯一，这样就可以通过二者的组合去快速找到对应的引用。

如果说File GUID表示为文件和文件之间的关系，那么Local ID表示的就是文件内部各对象之间的关系。

一个对象通常是由**一个或多个**对象构成，每个记录在&符号后面的数字都是一个Local ID，每一个Local ID也表示这它将来也会被实例化成一个对象。

也就是说，当一个prefab文件要实例化成一个GameObject时，它会自动尝试获取其内部Local ID所指的那个对象。如果这个所指的对象当前还没有被实例化出来，那么Unity会自动实例化这个对象，如此递归，直到所有涉及的对象都被实例化。

Unity还在内部维护了一张资产GUID和路径的映射表，每当有新的资源进入工程，或者删除了某些资源，又或者调整了资源路径，Unity的编辑器都会自动修改这张映射表以便正确地记录资产位置。所以如果.meta文件丢失或者重新生成了不一样的GUID，Unity就会丢失引用，在工程内的表现就是某个脚本显示“Missing”，或者某些贴图材质的丢失导致场景出现粉红色。

### 3. Library的资源位置

前面我们提到了非Unity支持的格式，需要由导入器进行资源转换。之所以要分到这个小节来讲位置是因为它涉及到了File GUID。之所以需要对资源转换和存储，也是为了方便下一次启动的时候不需要再处理资源，比较每次导入资源是巨耗时的操作。简单来讲，所有的转换结果都会存储在Library/metadata/目录下，以File GUID的前两位进行命名的文件夹里。

**注意：原生支持的Assets资源也会有同样的存储过程，只是不需要再用导入器进行转化而已。**

### 4. Instance ID

File GUID和Local ID确实已经能够在编辑器模式下帮助Unity完成它的规划了，与平台无关、快速定位和维护资源位置以及引用关系。但若投入到运行时，则还有比较大的性能问题。也就是说运行时还是需要一个表现更好的系统。于是Unity又弄了一套**缓存**（还记得前面那套缓存嘛？是用来记录GUID和文件的路径关系的）。PersistentManager用来把File GUIDs和Local IDs转化为一个简单的、Session唯一的整数，这些整数就是**Instance ID**。Instance ID很简单，就是一个递增的整数，每当有新对象需要在缓存里注册的时候，简单的递增就行。

PersistentManager会维护Instance ID和File GUID、Local ID的映射关系，定位Object源数据的位置以及维护内存中（如果有）Object的实例。只要系统解析到一个Instance ID，就能快速找到代表这个Instance ID的已加载的对象。如果Object没有被加载，File GUID和Local ID也可以快速地定位到指定的Asset资源从而即时进行资源加载。

**Unity维护的一套将GUID和FileID解析为数据源地址的机制，**这套机制中的信息，来自于：

1. 场景加载时，Unity收集了与该场景关联的资源信息。
2. 项目启动时，Unity收集了所有Resources文件夹下的资源信息。
3. 读取AssetBundle时，Unity获取了AssetBundle文件的头部信息(Header)。

**可以理解为：随着Unity知道更多的信息，这套机制将能够解析并定位更多的GUID和FileID。**

## 三、Assets目录下常见的文件类型

在Unity3d中一般存在这么几种文件：

* 资源文件
* 代码文件
* 序列化文件
* 文本文档
* 非序列化文件
* Meta文件

### 1. 资源文件

资源文件指一些创建好并且不再修改的文件。这样的文件一般是美术设计师和音频视频设计师创造的文件，比如FBX文件、贴图文件、音频文件、视频文件和动画文件（虽然动画文件可以被认为是配置文件，不过在由于一般不会去做修改，所以也认为是资源文件）。像这类文件，Unity中都会在导入时进行转化。每一个类型都对应一个AssetImporter，比如AudioImporter、TextureImporter、ModelImport等等。在Unity中点击这样的资源，在Inspector面板会出现设置界面。



### 2. 代码文件

代码文件包括所有的代码文件、代码库文件、Shader文件等。在导入时，Unity会进行一次编译。



### 3. 序列化文件（数据文件）

序列化文件通常是指Unity能够序列化的文件，一般是Unity自身的一些类型，比如Prefab(预制体)、Unity3d(场景)文件、Asset(ScriptableObject)文件、Mat文件(材质球)，这些文件能够在运行时直接反序列化为对应类的一个实例。



### 4. 文本文档

文本文档比较特殊，它不是序列化文件，但是Unity可以识别为TextAsset。很像资源文件，但是又不需要资源文件那样进行设置和转化，比如txt、xml文件等等。



### 5. 非序列化文件

非序列文件是Unity无法识别的文件，比如一个文件夹也会被认为是一个文件，但是无法识别。



### 6.  Meta文件

Meta文件在Unity中的作用非常关键，它有2个作用：

- 定义在它同目录下，同名的非Meta文件的唯一ID：GUID。而对于Unity的序列化文件来说，引用的对象用的就是这个GUID。所以一旦Meta中的GUID变更了，就要注意，它很可能引起一场引用丢失的灾难。
- 存储资源文件的ImportSetting数据。在上文中资源文件是有ImportSetting数据的，这个数据正数存储在Meta文件中。ImportSetting中专门有存储Assetbundle相关的数据。这些数据帮助编辑器去搜集所有需要打包的文件并分门别类。所以每一次修改配置都会修改meta文件。



## 四、Meta文件详解

由于Meta文件的重要性，这里先说说Meta文件的数据结构。Meta文件实质上是一个文本文档，只是采用的是一种叫做YAML的格式来写的（见Description of the Format）。Unity中的序列化文件都是用这个格式类写的，比如Prefab、场景等（后文会继续）。

### 1. GUID

Guid是Meta中最最最重要的数据。这个Guid代表了这个文件，无论这个文件是什么类型（甚至是文件夹）。换句话说，通过GUID就可以找到工程中的这个文件，无论它在项目的什么位置。在编辑器中使用AssetDatabase.GUIDToAssetPath和AssetDatabase.AssetPathToGUID进行互转。

所以在每次svn提交时如果发现有Meta文件变更，一定要打开看一下。看看这个Guid是否被更改。理论上是不需要更改的。



### 2. ImportSetting数据

后面比较重要的数据是ImportSetting数据。根据不同的文件类型，它的数据是不同的ImportSetting数据。只要对照Inspector面板中的条目，都可以看懂每一行的意义。



### 3. FileID(LocalID)

有一个问题是，如果是一个图集，下面有若干个图片，那么，这个GUID怎么对应一个文件呢？是的，对于一个文件下有多个文件的情况，就需要另外一个ID来表示，这就是LocalID。更习惯用Meta文件中的名字FileID。FileID存储方式有2种：

对于资源文件，非序列化文件，由于一般不会去更改源文件，所以FileID存储在meta文件中。

对于序列化文件，自身数据里面会存储自身的FileID，也会记录所有子文件的FileID。

回到本节一开始的问题，如果是图集，因为是图片本身是资源文件，所以会有FileID存储在对应的Meta文件中。

至此就是整个Unity的GUID/LocalID系统的基础了。通过GUID找到任何一个文件，通过FileID找到其中的某个子文件。



## 五、序列化文件详解--Unity文件引用系统

上文已经提到，对于所有的序列化文件，Unity采用的是YAML来书写。所以对于一个Unity3d(场景)文件、prefab文件、材质、控制器等都可以用文本文档软件打开。

为了能够简洁地说明问题，我们在Unity中创建一个新的场景，然后创建2个Cube，一个做成Prefab。

然后使用VScode打开这个场景，我们可以看到其中的数据：

- OcclusionCullingSettings裁剪数据（菜单Window->Occlusion面板中的数据）
- RenderSettings（菜单Window->Lighting->Settings面板中的部分数据）
- LightmapSettings（菜单Window->Lighting->Settings面板中的其他部分数据）
- NavMeshSettings（菜单Window->Navigation面板中的数据）
- 之后就是场景中的物件的数据

### 1. GameObject

展开第一个GameObject数据，可以看到这个的Name就是Main Camera（如果你的第一个GameObject数据不是Main Camera也没关系，下面肯定某一个是Main Camera，展开那个数据也是一样的结构）。

这个物体上有4个组件，一一对应下面的数据。这就是物体内的引用关系。每一个Unity对象都会有一个FileID，然后在需要引用时，使用这些FileID即可。所以在实例化一个这样的GameObject时，只要依照次序，依次创建物体，组件，初始化数据并进行引用绑定即可在场景中生成一个实例。

选中Main Camera，我们在Inspector面板中的右上角点击，然后选择Debug转成Debug模式下的Inspector面板。

在Hierarchy面板中选中Main Camera可以看到，所有的组件的LocalIdentfierInFile的值就是刚刚在Vscode中看到的数据。



这里有一点，我们看到有一个叫做InstanceID的数据。这个是Unity中一个实例的ID。每一个Unity实例都会有一个InstanceID。在运行时，可以使用UnityEngine.Object的GetInstanceID获取。但是要注意的是，每一次运行，相当于重新生成了新的实例，所以这个值是可变的。



### 2. 组件数据

在GameObject之后就是这个GameObject的组件数据（不知道次序会不会乱，理论上不影响）。每一个组件的数据基本上就是这个组件的一堆参数了。可以结合Unity中这个组件的面板来了解每一个数据的意义。

这里有一个问题，比如这里有一个组件是FlareLayer，但是在YAML里面只是一个Behaviour（所有Behaviour组件都看不到类型名字），怎么样才能知道他是一个FlareLayer？

在FileID左边我们看到一个`XXX`数字。对，这个就是FlareLayer。请参考YAML Class ID Reference，每一个Unity类型都有一个对应的数字。



我们创建一个Test脚本，继承MonoBehaviour。里面什么都不写。添加到Main Camera物体上。

然后回到VScode中，可以看到多了一个MonoBehaviour，并且这个里面有一个M_Script数据，指向对应的GUID及其FileID。上文我们已经说了，任何一个文件都可以通过GUID找到，然后通过FileID找到它内部的子文件。所以这样就能识别出这个具体是什么类了。

我们往Test中写2个字段：

* Public int A;
* Public Test RefText;

在Main Camera中，设置Test脚本的A值为111，RefTest设置为自身。

然后我们回到VScode刚刚打开的那个文件中，就可以看到，在MonoBehavior中，有了A和RefText的值。



### 3. Prefab数据

在YAML的最下面有一个数据是Prefab数据。

看起来很复杂，但是实际上，它就保存了最重要的几个数据：

* Modification：每个组件的修改数据列表，但凡修改的数据，都会在这里体现。
* ParentPrefab：表示哪一个Prefab。

所以上面的数据就是GUID为27f1445c35c923741a22e4948c4da980的Prefab，修改后的FileID为4126423848245890的一堆数据和FileID为1854796474342856的Name数据。

通过打开我们制作的Cube的Prefab文件及其meta文件，我们可以看到，Meta文件中的GUID就是那个，而Prefab中存在4126423848245890（Transform）和1854796474342856（GameObject）



## 六、 AssetBundle

AssetBundle是Unity官方推荐的资源加载方式。上边我们已经简单的介绍了，接下来看看如何生成AssetBundle，加载以及卸载等。

### 1. AssetBundle的生成

生成AssetBundle有很多种方式，在此仅简单说一下比较常用的方式，使用BuildPipeline生成AssetBundle文件。

每一次调用BuildPipleLine.BuildAssetBundles时，将会生成一批AssetBundle文件，具体数量根据传递AssetBundleBuild数组决定，**每一个AssetBundleBuild对象将对应一个AssetBundle及一个同名+.manifest后缀文件**。其中AssetBundle文件的后缀用户自行设置，比如".unity3d"，".ab"等等；而.manifest文件是给人看的，里面有这个AssetBundle的基本信息以及非常关键的资源列表。

除了AssetBundleBuild数组所定的AssetBundle外，还将额外在output路径下生成的一对与output文件夹同名的文件及一个同名.manifest后缀文件。这个同名文件可厉害了，它记录了这批次AssetBundle之间的相互依赖关系。当然.manifest文件还是给人看的，我们可以用它分析资源间的依赖关系，但是在项目实际运行时，Unity并不会关心它。

![](https://mmbiz.qpic.cn/mmbiz_png/F03VdOKPlCmdicLeMcJWt5NAT4od0OA4cNiaf5JWIy6R9pWWdJ3hINLL4DlacwoN7m3TVKnc5rmdY43Zj3KpOdaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 2. AssetBundle的加载

根据AssetBundle文件所在的位置(本地、远端)，AssetBundle有不同的加载方式，在此仅总结最常用的本地AssetBundle文件加载。

我个人将AssetBundle拆分理解为：**Bundle加载**和**Asset加载**两部分。因为AssetBundle文件可以从功能上分为两大块：

1. 记录文件标记、压缩信息、文件列表的Header部分。
2. 记录资源实际内容的Data部分。

当使用AssetBundle.LoadFromFile或LoadFromFileAsync时，在pc平台及移动平台上，unity**仅会为我们读取AssetBundle的header部分，并不会将bundle的data部分整个读入内存。**



当调用上一步生成的AssetBundle对象读取具体资源时（LoadAsset, LoadAssetAsync, LoadAllAssets），Unity会参考已经缓存的文件列表，找到目标资源在data部分的位置并读入到内存中。



**如果一个资源引用到了其他资源，则必须要先读入被引用资源的AssetBundle文件，否则就会发生引用Miss。**



为了避免上面Miss的情况，在加载资源时，**首先需要将该资源的依赖项全部加载完毕，不过仅需加载依赖资源的AssetBundle文件。**也就是说，我们只要将该依赖AssetBundle的Header部分加载（AssetBundle.LoadFromFile或LoadFromFileAsync）就可以，这样在真正读取Asset时，Unity会自动处理好真实依赖的Asset，我们不用操心。



AssetBundle的依赖关系如何读取呢？加载上面提到的那个很厉害的文件就可以了。

```c#
var bundle = AssetBundle.LoadFromFile(bundleManifestPath);
if(bundle)
{
    //读取AssetBundleManifest资源
    AssetBundleManifest abManifest = bundle.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
    //获取生成的全部AssetBundle，对应一次BuildPipeline.BuildAssetBundles
    string[] assets = abManifest.GetAllAssetBundles();
    //遍历所有AB资源
    foreach(var item in assets)
    {
        //获取AB资源的直接依赖资源A->B->C则返回{B}
        sttring[] dependencies = abMainfest.GetDirectDependencies(item);
        //获取AB资源的全部依赖资源 A->B->C返回{B,c},不推荐使用
        string[] dependencies = abManifest.GetAllDependencies(item);
    }
}
```

非常简单的获取依赖关系的方法，通常会在项目启动时将全部依赖关系保存下来。



### 3. AssetBundle的使用

当AssetBundle被成功加载后，调用该Assebbundle对象的LoadAsset、LoadAllAssets或对应的异步版本即可加载资源，也就是实例化对象。如果这个对象已经被加载过，Unity并不会重复加载，还记得之前所说的映射表么，被加载过的资源就好比挂上了数字牌的钥匙，直接对地址解引用即可。



### 4. AssetBundle的卸载

如果说AssetBundle真的有什么容易出问题的地方，那恐怕就是卸载了。

在这里只说最常用的这个卸载方法吧：

```c#
public void Unload(bool unloadAllLoadedObjects);
```

一个被加载过的AssetBundle可以通过调用Unload来卸载这个Bundle下所有的Asset。但是调用这个函数时传入的参数**对卸载结果影响甚大。**

Unity官方对这个函数的讲解非常详细，配图也非常直观，因此我只是简单总结一下。

**相同点：**

无论传入参数为 true 或是 false，调用Unload都可以Destroy当前AssetBundle对象，释放之前从AssetBundle文件中的Header部分所获取的信息。当然，被释放的AssetBundle对象无法再使用诸如LoadAsset、LoadAllAssets等函数加载资源。

---

**不同点：**

unloadAllLoadedObjects == true：



不仅Destroy了AssetBundle这个对象，而且**这个AssetBundle下包含的所有对象，只要实例化了，有一个算一个，统统释放掉。**

比如你通过ab.LoadAsset(apple)后，将apple设置给go_0的一个Renderer，如果这时候ab.Unload(true)，那go_0就傻了，咋回事儿啊，图咋没了呢？

它的好处是：**不会有重复资源问题的情况发生，每次都处理的干干净净。**

unloadAllLoadedObjects == false：

仅仅Destroy了AssetBundle这个对象，但是并没有释放这个AssetBundle下的任何Asset，因此如果有对象引用了这些Asset，也不会有问题。

它的风险(代价)是：下次再Load这个AssetBundle，并且通过这个AssetBundle重新读取了这个Asset，会在内存中重新创建一份，这样如果之前的Asset没有被释放，那么现在内存中就有两份Asset了。

这种情况如果频繁发生，便意味着内存中有很多资源将“不受控制”，容易引发内存占用过高的问题，而释放这种不受控的资源，仅有两种方式：

1. 当没有对象引用到这些不受控资源时，每次调用Resources.UnloadUnusedAssets，回收之。
2. 加载场景时，如果加载模式没有设置为LoadSceneMode.Additive，则会自动调用Resources.UnloadUnusedAssets。



## 七、资源生命周期

到现在为止我们已经搞清楚了Unity的Asset在编辑器和运行时的关联和引用关系。那么接下来我们还要关注一下这些资源的生命周期，以及在内存中的管理方式，以便大家能更好地管理加载时长和内存占用。

### 1. Object加载

当Unity的应用程序启动的时候，PersistentManager的缓存系统会对项目立刻要用到的数据(比如：启动场景里的这些或者它的依赖项)，以及所有包含在Resources目录的Objects进行初始化。如果在运行时导入了Assets或者从AssetBundles（比如：远程下载下来的）加载Object都会产生新的Instance ID、

另外Object在满足下列条件的情况会自动加载，比如：

* 某个Object的Instance ID被间接引用了
* Object当前没有被加载进内存
* 可以定位到Object的源位置（File GUID 和 Local ID）

另外，如果File GUID和Local ID没有Instance ID，或者有Instance ID，但是对用的Objects已经被卸载了，并且这个Instance ID引用了无效的File GUID和Local ID，那么这个Objects的引用会被保留，但是实际Objects不会被加载。在Unity的编辑器里会显示为：“（Missing）”引用，而在运行时根据Objects类型不一样，有可能会是空指针，有可能会丢失网格或者纹理贴图导致场景或者物体显示粉红色。

### 2. Object卸载

除了加载之外，Objects会在一些特定情况下被卸载。

1. 当没有使用的Asset在执行清理的时候，会自动卸载对应的Object。一般是由切场景或者手动调用了Resources.UnloadUnusedAssets的API的时候触发的。但是这个过程只会卸载那些没有任何引用的Objects。
2. 从Resources目录下加载的Objects可以通过调用Resources.UnloadAsset API进行显式的卸载。但这些Objects的Instance ID会保持有效，并且仍然会包含有效的File GUID 和LocalID。当任何Mono的变量或者其它Objects持有了被Resources.UnloadAsset卸载的Objects的引用之后，这个Object在被直接或者间接引用之后会马上被加载。
3. 从AssetBundles里得到的Objects在执行了AssetBundle.Unload(true) API之后，会立刻自动的被卸载，并且这会立刻让这些Objects的File GUID、Local ID以及Instance ID立马失效。任何试图访问它的操作都会触发一个NullReferenceException。但如果调用的是AssetBundle.Unload(false)API，那么生命周期内的Objects不会随着AssetBundle一起被销毁，但是Unity会中断File GUID 、Local ID和对应Object的Instance IDs之间的联系，也就是说，如果这些Objects在未来的某些时候被销毁了，那么当再次对这些Objects进行引用的时候，是没法再自动进行重加载的。

另外，如果Objects中断了它和源AssetBundle的联系之后，那么再次加载相同Asset，Unity也不会复用先前加载的Objects，而是会重新创建Instance ID，也就是说内存里会有多份冗余的资源。



## 八、 资源管理的注意点

1. 移动Unity资源时，要在Unity编辑器内拖动，不要在操作系统下剪切粘贴。因为这样Unity会为这个文件生成一个新的File GUID及.meta文件，它会打破之前建立好的关系，让所有引用过这个文件的prefab出现miss的情况。
2. 实际上在项目build完成后，就已经不存在File GUID和Local ID的概念了，转而用相对简单方式建立映射，这也是为什么我们在项目运行的过程中无法获取到File GUID的原因，不过原理上它们是一样的。
3. 尽管一个AssetBundle的Header部分非常小，通常只有几十KB，但是Unity并不能保证读入大量AssetBundle的Header部分后资源的加载效率。因此还是按需读取AssetBundle吧。



[浅谈Assets -- Unity资源映射]: https://blog.uwa4d.com/archives/USparkle_Addressable1.html

[Unity资源加载及管理]:https://mp.weixin.qq.com/s/0XFQt8LmqoTxxst_kKDMjw?