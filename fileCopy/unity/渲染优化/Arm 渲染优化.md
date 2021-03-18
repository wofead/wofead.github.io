# Unity 渲染优化

[toc]

## unity中的设置

在unity的项目设置的质量选项中，有很多对游戏性能产生巨大影响的选项：

1. 像素光源数量  ：是指可以影响给定像素的光源数量。较高的像素光源数量需要执行大量计算。大部分游
   戏能够在使用极少量动态实时光源的同时，最大限度降低对质量的影响。如果光照引发了性能问题，
   请考虑在游戏中使用光照贴图和投影纹理等技术。  
2. 纹理质量：可以给 GPU 带来负载，但通常不会引发性能问题。降低纹理质量会对游戏的视觉质量产生
   不良影响，所以请只在必要的情况下降低纹理质量。在冰穴演示中， 纹理质量设为全分辨率。
   如果纹理引发了性能问题，可尝试使用纹理映射。纹理映射可降低计算和带宽要求，同时不会影响图
   像质量。  
3. 抗锯齿是一项边缘平滑技术，该技术混合了三角形边周围的像素。这显著提高了游戏的视觉质量。有
   多种抗锯齿方式，但是此种情况下采用的是多重采样抗锯齿 (MSAA)。 4x MSAA 在 Mali GPU 上的运
   算量较低，应尽可能使用。  
4. 软粒子：需要渲染到深度纹理或在延迟模式下渲染。这会提高 GPU 负载，但可以获得逼真的粒子效果，
   因此值得采用。在移动平台上，渲染到深度纹理和从深度纹理读取会消耗掉宝贵的带宽，并且使用延迟路
   径进行渲染意味着您不能使用 MASS。请考虑软粒子是否足够重要到需要在游戏中使用。  
5. 各向异性纹理：技术可消除在高梯度下绘制的纹理的失真。这项技术可提高图像质量，但是非常消耗资
   源。除非失真特别明显，否则请避免使用此技术。  
6. 阴影：具有高质量时，计算量较大。如果阴影引发了性能问题，请尝试采用简单的阴影或关闭阴影。如果
   阴影对于您的游戏非常重要，请考虑使用简单的动态阴影技术，例如投影纹理。  
7. 实时反射探测器：选项对运行时性能存在显著的负面影响。  
8. 剔除遮罩  ：在渲染立方体贴图时使用剔除遮罩，从而避免对反射中任何无关几何体进行渲染。  
9. 立方体贴图更新  ：刷新模式选项定义立方体贴图的更新频率，每帧，唤醒，借助脚本。

## unity中分析性能的工具

1. unity分析器
2. unity frame debugger
3. arm mali graphics debugger
4. arm ds-5 streamline

### Unity 分析器

Unity 分析器以一系列图表的形式提供详细的每帧性能数据，帮助您查找游戏中的瓶颈。  通过点击图标，可以看到这一帧的垂直切片信息。

1. CPU使用率分析器：CPU 使用率分析器图表显示了 CPU 使用率的细分情况，重点突出不同组件（如渲染、脚本或物理）的 CPU 使用情况。如果您在图表上选择了某帧，则面板将显示对该帧影响最大的功能的耗费时间、调用数量或内存分配。  **请重点关注耗费更多执行时间或分配过多内存的功能。  **

2. 渲染分析器：渲染分析器图表显示了绘制调用、三角形以及在场景中渲染的顶点的数量。在图表上选择某帧将显示更多有关批处理、纹理和内存消耗的信息。  **请仔细查看绘制调用、三角形以及场景渲染的顶点的数量。这些是移动平台上最为重要的数字。  **

3. 内存分析器：内存分析器图表显示了分配的内存数量以及游戏所用资源（如网格或材质） 的数量。在图表上选择某帧后，将显示资源、图形和音频子系统或分析器数据本身的内存消耗情况。  **移动平台上的内存有限，因此您必须在其生存期内监控游戏需求，并检查占用资源的数量。  **

4. 添加分析器功能  ：添加分析器选项位于分析器窗口左上角的下拉菜单中。使用该选项能够向分析器窗口添加更多图表，例如 CPU 使用率、渲染或内存。  

5. Profiler.BeginSample() 和 Profiler.EndSample() 方法  ：Unity 分析器可让您采用 Profiler.BeginSample() 和 Profiler.EndSample() 方法。您可以在脚本中标记一个区域，然后附上自定义标签，此区域将作为单独的条目出现在分析器层级中。通过执行此操作，您可以获取特定代码的信息，而无需采用深度分析选项，从而节省计算和内存开销。  

   ```
   void Update()
   {
   	Profiler.BeginSample("ProfiledSection");
   	...
   	Profiler.EndSample();
   }
   ```

   

### unity frame debugger

Frame Debugger 是一款分析工具，您可以在每一帧基础上追踪绘制调用  .Frame Debugger 可以从窗口菜单中选用。  

## 优化列表

1. 使用协程代替Invoke(),Monobehaviour.Invoke() 方法可快速便捷地在时间延迟的情况下调用类中的方法，但是它存在局限性：它使用c#反射查找调用方法，这比直接调用方法的速度更慢；没有针对方法签名的编译时检查，无法提供附加参数。

   ```c#
   public void Function()
   {
   ...
   }
   Invoke("Function", 3.0f);
   
   public IEnumerator Function(float delay)
   {
   	yield return new WaitForSeconds(delay);
   }
   StartCoroutine(Function(3.0f));
   ```

   

2. 使用协程轻松进行更新：如果您的游戏需要每隔一段特定时间执行操作，请尝试在 MonoBehaviour.Start() 回调函数中启
   动协程，以此代替 MonoBehaviour.Update() 回调函数（每帧都要执行操作）。  

   ```c#
   void Update()
   {
   	//Perform an action every fram
   }
   
   IEnumerator Start()
   {
   	while(true){
   	//Do something every quarter of second
   	yield return new waitForSecondds(0.25f);
   	}
   }
   ```

   3. 使标记避免硬编码字符串:由于标记的硬编码值会限制游戏的可扩展性和健壮性，因此应避免使用。例如，如果您直接通过字符
      串引用名称，则无法轻松修改标记名称，并且很可能会出现拼写错误。  

      ```c#
      if(gameObject.CompareTag("Player"))
      {
      	[...]
      }
      -- 可以这样做
      public class Tags
      {
          public const string Player = "Player";
      }
      
      if(gameObject.CompareTags(Tags.Player))
      {
      	[...]
      }
      
      ```

      

4. 通过更改固定时间步长减少物理计算量：您可以通过更改固定时间步长降低物理计算的计算量。通常，大部分物理计算发生在固定时间步长内，您可以增加或减小此步长。    增加时间步长会减小应用程序处理器上的负载，但是会降低物理计算的准确性。  您可以从主菜单访问时间管理器： 编辑 > 项目设置 > 时间。  

5. 删除空的回调函数，如果代码包含Awake、Start或Update等函数的空定义，则将其删除。

6. 避免在每帧中都使用GameObject.Find()。GameObject.Find() 函数用于循环访问场景中的每个对象。如果它在代码中使用的位置不正确，
   则会导致主线程大小显著增加。  例如不要再Update中使用Find函数，而是在Awake中使用，然后缓存，或者使用FindWithTag函数。

7. 使用StringBuilder类连接字符串，这样可以避免由加号产生的临时的字符串。

8. 使用GameObject.CompareTag()方法代替标记属性。使用GameObject.CampareTag()方法代替GameObject.tag属性。CompareTag()方法的速度更快，并且不会分配额外的内存。

   ```c#
   GameObject mainCamera = GameObject.Find("Main Camera");
   if(mainCamera.tag == "MainCarema"){
       //action
   }
   
   if(mainCarema.CompareTag("MainCamera")){
       //action
   }
   ```

   9. 对象次的使用。
   10. 缓存组件检索。缓存 GameObject.GetComponent<Type>() 返回的组件实例。涉及的函数调用非常消耗性能。Gameobject.camera、ameobject.rendener 或 Gameobject.transform 等属性是对应GameObject.GetComponent<Camera>()、 Gameobject.GetComponent<Renderer>() 和Gameobject.GetComponent<Transform>() 的快捷方式  。
   11. 使用 OnBecameVisible() 和 OnBecameInvisible() 回调函数  ：如果与 MonoBehaviour.OnBecameVisible() 和 MonoBehaviour.OnBecameInvisible()  等回调函数相关的游戏对象出现在或消失在屏幕上，则这些回调函数将通知您的脚本。  如果某一游戏对象未在屏幕上渲染，这些调用能够让您禁用大量计算性代码例程或特效。  
   12. 使用 sqrMagnitude 比较向量幅度  ，使用 Vector3.sqrMagnitude 代替Vector3.Distance() 或 Vector3.magnitude。  
   13. 使用内置数组  ：如果您事先知道数组大小，则使用内置数组。  ArrayList 和 List 类的灵活性更高，它们会随插入元素的增加而增大，但是速度要比内置数组慢。  
   14. 使用平面作为碰撞目标  ：如果场景只需要与平面物体（如地板或墙壁）进行粒子碰撞，则可将粒子系统碰撞模式更改为Planes。将设置更改为使用平面可降低所需的计算量。    
   15. 使用复合的原始碰撞器（而非网格碰撞器）  ：网格碰撞器以对象的实际几何数据为基础。它们在碰撞检测方面具备极高的准确性，但是计算量非
       常大。  您可以将盒子、舱体或球体等形状合并为模拟原始网格形状的复合碰撞器。这可使您获得相似的结果， 同时还可大幅降低计算开销。  

   

## GPU优化

### 杂项GPU优化

   1. 使用静态批处理：静态批处理是一种常见的优化技术，可以减少绘制调用数量，从而降低应用程序处理器的使用率。动态批处理可由Unity 以透明的方式执行，但是无法运用至大量顶点组成的对象，因为计算开销过大。 静态批处理可在大量顶点组成的对象上运行，但是经过批处理的对象在渲染过程中不得移动、旋转或扩大。若要使 Unity 能够集合要进行静态批处理的对象，请在“检视面板”中将它们标记为静态。  
   2. 使用 4x MSAA  ：ARM Mali GPU 能够以极低的计算开销执行 4x 多重采样抗锯齿。您可以在“Unity 质量设置”中启用 4xMSAA。  
3. 使用细节层次  ：Unity 引擎可以使用细节层次 (LOD) 技术根据与摄像机的距离渲染同一对象的不同网格。当对象更接近摄像机时，几何结构更为清晰。随着对象远离摄像机，细节层次降低。当处于最远距离时，您可以使用平面公告板。      
   4. 避免在自定义着色器中使用数学函数 ：在编写自定义着色器时，请避免使用计算量大的内置数学函数  ，例如：pow、exp、log、cos、sin、tan。
### 光照贴图和灯光探测器

运行时光照计算的计算成本很高。一项用于降低计算要求的常见技巧称为光照贴图，它预先进行光照计算并将 它们烘焙为名为光照贴图的纹理。   这意味着您会丧失完全动态光照环境的灵活性，但您可以生成质量非常高的图像，而不会影响性能。  在静态光照贴图中烘焙生成的光照 ：

* 将接受光照的几何体设置为静态。
* 将灯光中的烘焙选项设置为已烘焙，而不是运行时。

* 在光照贴图窗口的"场景"选项卡中，选中已烘焙 GI选项。

**查看生成的光照贴图：**

* 选择几何体
* 选择窗口>光照
* 按对象按钮
* 选择预览选项中的红别强度光照贴图。

**构建光照贴图  :**

若要为光照贴图准备对象，您必须拥有:

1. 带有光照贴图 UV 的场景模型  
2. 模型必须标记为静态光照贴图  
3. 模型范围内必须存在光源  
4. 光源的烘焙类型必须设为“已烘焙”

**对象：**单击对象按钮可更改与您在层级中所选对象的光照贴图相关的设置。借此可修改影响光照贴图流程的对象设置。选择一个光源，便可更改诸多选项：  

* 仅烘焙：可在烘焙时启用光源并在运行时禁用光源。  
* 已烘焙：如果选中了已烘焙 GI，将烘焙该光源。  
* 实时：光源可用于预计算的实时 GI，也可在没有 GI 时使用。  
* 仅实时：可在烘焙时禁用光源并在运行时启用光源。  
* 混合：光源已烘焙，但它在运行时仍然存在，为非静态对象提供直接光照。  

将大部分光源设为已烘焙可确保运行时的计算量相对较低。  

**场景:**场景选项卡包含应用于整个场景的设置。您可以在此选项卡中启用和禁用预计算实时GI 和已烘焙 GI功能。  

**环境光照**部分中提供了诸多选项，您可用于定义影响环境光照的多个因素， 如天空盒、环境光源类型
以及环境光强度等：  

* 反射反弹次数选项是性能方面最重要的选项。 反射反弹次数定义反射对象之前相互反射的次数，即视野覆盖这些对象的探测器的烘焙次数。如果反射探测器在运行时更新，此选项对性能有很大的负面影响。只有反射对象在探测器中清晰可见时，才可将反射次数设为高于一的值。  
* 在预计算实时 GI 选项卡中， CPU 使用率选项定义在运行时评估GI 上花费的处理器时间量。较高的 CPU 使用率值可以加快光照的反应速度，但可能会影响帧率。在多处理器系统中，对性能的影响比较小。  
* 已烘焙 GI 选项卡中的一个选项可以设置要压缩的光照贴图纹理。压缩光照贴图纹理可以降低对存储空间和带宽的要求，但压缩处理可增加纹理的失真。  
* 在常规 GI 选项卡中，请谨慎使用定向模式选项。如果您无法针对双重光照贴图使用延迟光照，则另外一种方法是使用定向光照贴图。这能够使您在没有实时光照的情况下使用法线贴图和镜面反射光照。如果必须保存法线贴图但未提供双重光照贴图，则可使用定向光照贴图。移动设备通常就属于这种情况。当定向模式设为定向时，将创建一个额外的光照贴图，以存储射入光线的主要方向。因此，这一模式要求大约两倍的存储空间。  
* 在定向镜面模式中，会为镜面反射和法线贴图存储额外的数据。这时存储要求将提高四倍。  
* 光照贴图选项卡可用于设置和查找供场景使用的光照贴图资源文件。要访问光照贴图快照方框，必须取消选中持续烘焙选项。  

**使用定向光照贴图**  :如果您无法针对双重光照贴图使用延迟光照，则另外一种方法是使用定向光照贴图。这能够使您在没有实时光照的情况下使用法线贴图和镜面反射光照。  

**针对游戏中的动态对象使用灯光探测器  :**您可以使用灯光探测器向进行光照贴图的场景添加一些动态光照。  

### ASTC 纹理压缩  



ASTC 纹理压缩是 OpenGL 和 OpenGL ES 图形 API 的官方扩展。ASTC 可以减小应用程序所需的内存以及 GPU 需要的内存带宽。ASTC 提供的纹理压缩质量高、比特率低，而且控制选项也很多。它具有下列特性：  

* 比特率从 8 位每像素 (bpp) 到小于 1 bpp 不等。这可使您微调文件大小与质量的平衡值。  
* 支持 1 至 4 个颜色通道。  
* 同时支持低动态范围 (LDR) 和高动态范围 (HDR) 图像。  
* 支持 2D 和 3D 图像。  
* 支持选择不同的特性组合。  

ASTC 设置窗口中有多个块大小选项。您可以从中选用，选择与资源最匹配的块大小。块大小越大，提供的压缩率越高。为显示细节度不高的纹理选择较大的块大小，如距离摄像机较远的对象。为显示细节度较高的纹理选择较小的块大小，如距离摄像机较近的对象。  

**为 ASTC 纹理选择恰当的格式  :**纹理压缩算法具有不同的通道格式，通常为RGB 和 RGBA。 ASTC 支持多种其他格式，但是这些格式不会出现在Unity 中。通常，每种纹理用于不同的用途，例如：标准纹理、法线贴图、镜面反射、 HDR、 alpha 和查找纹理。为获得尽可能好的结果，所有这些纹理类型都需要不同的压缩格式。  

### 纹理映射  

纹理映射技术与能够同时提高游戏视觉质量和性能的纹理相关。  

纹理映射是不同大小的纹理的预计算版本。每个生成的纹理称为一个层级，它们的宽度和高度是前一个纹理的一半。 Unity 能够自动生成完整的层级，从原始尺寸的第一层级到 1x1 像素版本。  

若要生成纹理映射，请执行以下操作：  

1. 在项目窗口中选择某一纹理。  
2. 将纹理类型更改为高级。  
3. 在检视面板中启用生成纹理映射选项。  

如果纹理没有纹理映射层级，当具有纹理的表面覆盖的区域（以像素表示）小于纹理尺寸时， GPU 会将纹理缩小至适合更小的区域。但是，此过程中会丧失部分准确性，即便使用滤波器插值像素颜色。  

如果纹理具有纹理映射层级， GPU 将从最接近对象大小的层级中提取像素数据以渲染纹理。这可提高图像质量并降低带宽，因为GPU 为获得更高质量已经离线扩展了层级，并且GPU 仅从恰当的层级中提取纹理数据。多级渐进纹理 (mipmapping) 的劣势在于它额外需要 33% 的内存来存储纹理数据。  

**纹理映射和 GUI 纹理  :**您通常不需要对 2D UI 中所用的纹理进行纹理映射。 UI 纹理通常不需要扩大即可在屏幕上渲染，它们仅使用纹理映射链中的第一层级。  

若要更改此设置，请在项目窗口中选择某一纹理，然后进入检视面板并查看纹理类型。将类型设置为编辑器 GUI
和传统 GUI，或者将类型设置为高级并禁用生成纹理映射选项。  

### 天空盒  

天空盒经常用于游戏和其他应用程序中，有多种方法可运行天空盒。  

您可以通过使用单个立方体贴图渲染摄像机的背景来绘制天空盒。  

这需要一个立方体贴图纹理和一个绘制调用函数。与其他方法相比，该方法使用的内存、内存带宽和绘制调用函数较少。  

若要构建天空盒，请使用此方法：  

1. 选择摄像机
2. 确保清除标记设置为天空盒
3. 选择或添加天空盒组件

天空盒组件对每个材质都拥有一个点。该材质是 Unity 在每帧开始时用于绘制摄像机背景的材质  。

您使用的材质是包含全部所需信息的材质。使用移动 > 天空盒着色器创建材质，再使用该材质填充天空盒的六张图像。材质预览将显示一张图像。  

当您完成此材质后，将其拖入天空盒组件中。您的天空盒将在后台进行正确渲染，不会出现明显的裂缝或不必要的绘制调用。  

### 阴影

阴影有助于增加场景的透视度和真实感。没有阴影，有时很难告知对象的深度，尤其是它们与周围对象相似时。

阴影算法可能会非常复杂，渲染高分辨率的精确阴影时尤为如此。请确保在游戏中为阴影选择了相应级别的复杂度和分辨率。  

Unity 的编辑 > 项目设置 > 质量下包含多个阴影选项，它们可影响游戏的性能：  

* 硬阴影/硬加软阴影  ：软阴影看起来更为真实，但是计算时间较长。  
* 阴影距离  ：阴影距离选项可定义与出现阴影的摄像机的距离。增大阴影距离会增加可见阴影的数量，从而加大计算量。增大阴影距离还会增加用于阴影贴图中阴影的像素元数量，从而被动地提高阴影分辨率。  



您可以使用阴影距离较小且分辨率较高的硬阴影。这会在距离摄像机的适当范围内产生不是很复杂且质量较高的阴影  。

进行光照贴图的对象不会产生实时阴影，您在场景中烘焙的静态阴影越多， GPU 执行的实时计算就越少。  

**谨慎使用实时阴影  **

请考虑使用场景对象的网格渲染器组件。如果您不想使用它们投射或接收阴影，请相应地禁用投射阴影和接收
阴影选项。这可降低渲染阴影的计算量。  

### 遮挡剔除

遮挡剔除流程可在从摄像机角度看对象被遮挡时禁用对象渲染。这使得渲染的对象较少，从而节省GPU 处理时间  

但是，当对象完全退出摄像机视锥体时， Unity 将自动执行遮挡剔除，根据您的应用程序风格，可能仍存在不可见和无须渲染的其他对象。  

Unity 包含称之为 Umbra 的遮挡剔除系统。  

用于遮挡剔除的设置取决于您的游戏风格。在设置不恰当的场景中使用遮挡剔除会降低性能，所以请务必谨慎选择设置。  

### 指定渲染顺序  

如果按照任意顺序渲染多个对象，则可能会出现一个对象在渲染之后被其前面的另一对象遮挡。也就是说，渲染被遮挡对象需要的计算全都浪费。  

有各种各样的软件和硬件技术来减少被遮挡对象造成的计算浪费；但是，您可以引导这一流程，因为您清楚玩家如何探索这一场景。  

Unity 为您提供着色器内或材质内的排队选项，让您能够指定渲染的顺序。这可以在着色器中设置，这样具有使用该着色器的材质的对象可以一起渲染。在这一渲染组中，渲染的顺序是随机的，但透明等一些情形中除外。  

**使用渲染顺序提升性能  :**在冰穴演示中，洞穴覆盖了大部分屏幕，其着色器成本高昂。尽可能避免渲染它的组成部分可以提升性能  

利用 Unity Frame Debugger 和 ARM Mali Graphics Debugger 等其他工具查看帧缓冲区组成后，您会发现它已包含了渲染顺序优化。这些工具可帮助您查看渲染顺序。  

若要打开 Unity Frame Debugger，请选择菜单选项窗口 > Frame Debugger。这很有用处，因为一些东西在编辑器模式中似乎正确，但在您执行它们时却可能不能正确工作。例如，如果您有仅运行时设置或您要渲染来自另一摄像机的纹理，就可能会出现这种情形。以播放模式启动演示并定位摄像机后，您可以启用 Frame
Debugger，获取由 Unity 执行的绘制序列。  

在冰穴演示中，向下滚动绘制调用可显示洞穴率先得到渲染。然后，对象渲染到场景中，将已渲染的部分洞穴挡住。再例如，一些场景中的反光水晶被洞穴遮挡。在这些情形中，设置较高的渲染顺序值可以减少计算量， 因为不会为被遮挡的水晶执行片段着色器。  



### 使用深度预通道

通过为对象设置渲染顺序来避免过度绘制很有用处，但并不总是能够为每个对象指定渲染顺序。  

例如， 如果有一组具有计算成本高昂的着色器的对象，而且摄像机可以绕着它们自由旋转，那么某些之前位于后面的对象有可能会移到前面。这时，如果为这些对象设定了静态渲染顺序，一些对象即便被遮挡也会最后绘制。如果对象挡住了自身的一部分，也会出现这种问题。  

在这样的情形中，您可以使用深度预通道来减少过度绘制。深度预通道渲染几何体，但不在帧缓冲区中写入颜色。这将使用最近可见对象的深度来初始化各个像素的深度缓冲区。在此预通道后，几何体被正常渲染，但会使用Early-Z 技术，只有参与构成最终场景的对象才会被实际渲染。此技术需要额外的顶点着色器计算，因为要为每个对象计算顶点着色器两次，一次用于填充深度缓冲区，另一次用于实际的渲染。如果您的游戏受片段约束，并且您的顶点着色器中有额外容量，此技术很有用。  

在 Unity 中，若要为带有自定义着色器的对象执行渲染预通道，请为着色器添加额外的通道：  

```
Pass{
	ZWrite On
	ColorMask 0
}
```

在添加这一通道后， Frame Debugger 将显示对象被渲染了两次。第一次渲染时，颜色缓冲区中没有变化。  

### 资源优化

**禁用静态纹理读取/写入  **，如果你不动态修改纹理，请确保检视面板中的读写选项已被禁用。

**合并网格以减少绘制调用量**：为减少渲染所需的绘制调用数量，你可以使用Mesh.CombineMeshes()方法将多个网格合并为一个网格。如果所有网格的材质相同，请将mergeSubMeshes参数设置为true。以便它可以分局合并组中的每个网格生成单一子网格。

把多个网格合并为单个较大的网格将帮助您：  

* 创建更高效的遮挡器。  
* 将多个基于图块的资源转变为大型、无缝、实心的单一资源。  

网格合并脚本对性能优化很有帮助，但是这取决于您的场景构成。较大网格在视图中的停留时间长于较小网格，请进行试验，以获取正确的网格大小。  

应用此技术的一种方法是在层级中创建空的游戏对象，使其成为您想要合并的所有网格的父网格，然后为其附加一个脚本。  

**请勿在非动画 FBX 网格模型上导入动画数据  **：当导入不包含任何动画数据的 FBX 网格时，您可以在导入设置的“装备”选项卡中将动画类型设置为
无。这样设置后，将网格置于层级时 Unity 不会生成未使用的动画组件。
**避免读取/写入网格**  ：如果运行时修改了模型，则 Unity 在保留原始数据的同时会在内存中另外保存一份网格数据副本。如果运行时未修改模型（即使准备缩放），也请在导入设置的模型选项卡中禁用读取/写入已启用选项。这样便无需另外保存一份副本，因而可节省内存  

**使用纹理贴图集  ：**您可以使用纹理贴图集减少一组对象所需的绘制调用数量。  

纹理贴图集是指合并成一个大型纹理的纹理组。多个对象可通过一组合适的坐标重复使用此纹理。这有助于 Unity 对共享相同材质的对象采用自动批处理。  

设置对象的 UV 纹理坐标时，请避免更改其材质的 mainTextureScale 和 mainTextureOffset属性。这会创建新的独特材质，该材质无法与批处理一同运行。请通过 MeshFilter 组件获取网格数据并使用 Mesh.uv 属性更改每个像素元的坐标。  



**衡量 Unity 着色器  **：Unity 着色器使用编程语言 C for Graphics (Cg) 编写。 Cg 以 C 编程语言为基础，但进行了一些修改，因而更适用于 GPU 编程。  

在构建过程中， Unity 将 Cg 转换为 OpenGL、 OpenGL ES 或 DirectX。

检索 OpenGL ES 着色器代码  ：

1. 选择您要在 Unity 中分析的着色器。  
2. 选择 OpenGLES30 或 OpenGLES20 作为您要为之生成程序的自定义平台。  
3. 单击编译并显示按钮。  

顶点和片段会话通常由 #ifdef VERTEX 或 #ifdef FRAGMENT 分隔。如果使用 #pragma multi_compile <FEATURE_OFF> <FEATURE_ON> 等选项，文件中将生成多个着色器变体。  

通常会存在多个 VERTEX 和 FRAGMENT 部分。 Unity 启动您的应用程序时，会单独编译各个变体。启用一项功能时，会选中相关的变体。  

由于代码已转换为OpenGL ES，您可以将顶点和片段着色器代码复制到两个独立的文件，并使用Mali OfflineShader Compiler 进行编译。  

使用下列选项之一编译着色器：  

* -v 用于顶点着色器。  
* -f 用于片段着色器。  

**分析统计信息  **：Mali Offline Shader Compiler 产生的统计信息提供着色器所要求的每一顶点或像素的周期数的测量结果  。

结果分成三行：

* 合集
* 最短路径
* 最长路径

最短和最长路径通过检查代码中是否采取分叉的效应来衡量。这提供对执行周期数最小和最大值的估计。从算术而言，第一行中的测量结果除以算术流水线的数目。该数值为一、二或三，具体取决于 Mali GPU。第二和第三行用于加载/存储和纹理流水线。它们没有考虑缓存遗漏，所以最好将这些数字乘以 1.5，从而获得
更加实际的估计。  

**针对 Mali™ GPU 流水线进行优化  **：Mali Offline Shader Compiler 提供每一流水线中使用的周期数。着色器在流水线中速度最慢，其周期数最高。将流水线中速度最慢的作为优化目标，优化您的着色器。  

Mali GPU 包含三种类型的处理流水线：  

* 算术流水线
* 加载/存储流水线
* 纹理流水线

这些流水线全部并行运行。着色器通常使用所有这三种流水线。  

**算术流水线：**所有算术运算消耗算术流水线中的周期。  

下面列出了您可以降低算术流水线使用量的多种方法  ：

* 避免使用复杂算术，例如：矩阵求逆函数、球模运算符、触发、行列式、正弦、余弦
* 对于整数操作数，使用移位等运算计算除法、取模和乘法
* 为避免计算移项，如果其中一个矩阵已移项，更改矩阵-向量或矩阵-矩阵乘法中操作数的顺序。  也可以将负载移到其他流水线中，从而降低对算术管道的负载： 。
* 将矩阵作为统一变量传递，而不要计算它们。这可使用加载/存储流水线。  
* 使用纹理来存储代表正弦或余弦等函数的一组预计算值。  这可将负载移到纹理流水线。  

**加载/存储流水线：  **加载/存储流水线用于读取统一变量、写入变量，以及访问着色器中的缓冲区，如统一缓冲区对象或着色器存储缓冲区对象。  

如果应用程序属于加载/存储流水线束缚型，可尝试下列技巧：  

* 使用纹理而非缓冲区对象来读取着色器中的数据。
* 使用算术运算计算数据
* 压缩或减少统一变量和变量

**纹理流水线：**纹理访问使用纹理流水线中的周期，也会使用内存带宽。使用大纹理可能是不利的，因为缓存遗漏几率更高，
而且这样可能导致多个线程因为等待数据而停滞。  

要提高纹理流水线的性能，可进行如下尝试：

**使用纹理映射：**纹理映射可提高缓冲命中率，因为它根据纹理坐标变化选用分辨率最佳的纹理。  

**使用纹理压缩：**这也有益于降低内存带宽并提高缓存命中率。每一压缩的块包含多个像素元，所以其访问变得更容易缓存。  

**避免三线性或各向异性滤波  **:三线性和各向异性滤波将增加获取像素元所需的运算数。若非绝对必要，请避免使用这些技术。  



### 减少流水线周期的其他技巧

**避免寄存器溢出  **：Mali Offline Shader Compiler 可指示您的着色器是否溢满寄存器。造成寄存器溢出的原因通常是线程中包含大量变量，它们无法完整装入寄存器集中。  

寄存器溢出通常由包含大量以下对象的线程造成：  

* 输入统一变量
* 变量
* 临时变量

如果变量使用高精度，也可能会发生寄存器溢出。  

寄存器溢出会强制 Mali GPU 从内存读取一些统一变量，这可加重加载/存储单元的负载并降低性能。要解决此问题，可尝试减少您向着色器提供的统一变量数量并降低其精度。  

**降低变量和统一变量的精度  **：编写自定义着色器时，您可以指定利用 32 位浮点数或 16 位半浮点数指定统一变量和变量的浮点精度。精度确定了最小值和最大值，以及变量可以表示的数值的粒度。  

使用半浮点数有几个好处：  

* 带宽用量减少。  
* 算术流水线中使用的周期数减少，因为着色器编译器可以优化您的代码，使用更多并行化。  
* 要求的统一寄存器数量减少，这反过来又降低了寄存器溢出风险。

**将世界空间法线贴图用于静态对象  **：您可以使用正切空间法线贴图来提高模型的细致度，而不必增加几何细节。您可以将正切空间法线贴图用于动画对象而不必修改它们，因为它们具有对网格中每一三角形的局部性。  

但遗憾的是，这需要在着色器中进行更多算术运算，从而获得正确的结果。对于静态对象，这些计算通常是不必要的。  

您可以选择使用局部空间法线贴图或世界空间法线贴图。使用局部空间法线贴图可减少着色器中执行的计算数量，但是必须对采样的法线应用模型上的变换。世界空间法线贴图不需要任何变换，但它们是静态的，并且对象无法移动。在冰穴演示中，洞穴和其他高质量对象是静态的；使用世界空间法线贴图可以大幅减少着色器需要的ALU 运算数量。大多数常见的 3D 建模工具可以创建世界空间法线贴图，或者您可以通过在离线进程中的代码来生成它们。  

## 高级图形技术

### 自定义着色器

在 Unity 中，编写着色器的方法通常有两种：  

**表面着色器：**着色器受到光线和阴影影响时，通常使用表面着色器。 Unity 为您执行与光照模型相关的作业，能够让你编写更紧凑的着色器。  

**顶点和片段着色器  **:顶点和片段着色器最为灵活，但是您必须执行所有事项。 Unity ShaderLab 比顶点和片段着色器的功能更多，但它们是图形流水线的主要编程部分，您需要在图形流水线中完成所有着色。因此，了解如何编写自定义顶点和片段着色器非常重要。  

#### 着色器结构

下列代码显示了一个非常简单的顶点和片段着色器，它包含顶点或片段着色器所需的大部分元素。

着色器示例以 Cg 编写。 Unity 还支持适用于着色器片段的 HLSL 语言  

```
Shader "Custom/cttTextured"
{
	Properties
	{
	_AmbientColor("Ambient Color", Color) = (0.2,0.2,0.2,1.0)
	_MainTex("Base (RBG)", 2D) = "white" {}
	}

	SubShader
	{
		Pass
		{
			CGPROGRAM
			#pragma target 3.0
			#pragma glsg
			#pragma vertex vert
			#pragma fragment frag
			#include "UnityCG.cginc"
			
			//User-specified uniforms
			uniform float4 _AmbientColor;
			uniform sampler2D _MainTex;
		}
		struct vertexInput
		{
			float4 vertex: POSITION;
			float4 texCoord:TEXCOORD0;
		};
		struct vertexOutput
		{
			float4 vertex: SV_POSITION;
			float4 tex:TEXCOORD0;
		};
		// Vertxt shader
		vertexOutput vert(vertexInput input)
		{
			vertxtOutput output;
			output.tex = input.texCoord;
			output.pos = mul(UNITY_MATRIX_MVP, input.vertex);
			return output;
		}
		//Fragment shader
		float4 frag(vertexOutput input):COLOR
		{
			float4 texColor = tex2D(_MainTex, float2(input.tex));
			return _AmbientColor + texColor;
		}
		ENDCG
	}
	Fallback "Diffuse"
}
```

首个关键字是着色器路径/名称前的“Shader”。当您设置材质时，路径用于定义下拉菜单中显示着色器的类别。 此示例中的着色器显示在下列菜单中的自定义着色器类别下方。  

Properties{} 块列出了在检视面板中可见的着色器参数，以及您可以进行交互的参数。  

Unity 中的每个着色器均包含一系列子着色器。当 Unity 渲染网格时，它将查找要使用的着色器，并选择可在图形卡上运行的第一个子着色器。通过这种方式，可以在支持不同着色器型号的不同图形卡上正确运行着色器。 此特性非常重要，因为 GPU 硬件和 API 正在不断发展。例如，您可以 Mali Midgard GPU 为目标编写主着色器，从而利用 OpenGL ES 3.0 的最新特性，并同时在单独的子着色器中为支持 OpenGL ES 2.0 及更低版本的图形卡编写替代着色器。  

Pass 块会使对象的几何结构一次性渲染。着色器可包含一个或多个 pass 参数。您可在旧硬件上使用多个 pass参数，或者用其实现各种特效。  

如果Unity 无法在能够正确渲染几何结构的着色器主体中找到子着色器，则它将回滚到另一个在 Fallback 语句后定义的着色器。在此示例中，着色器为“漫反射”内置着色器。  

Cg 程序片段在 CGPROGRAM 和 ENDCG 之间编写 .

#### 编译指令

您将编译指令作为 #pragma 语句传递。编译指令可指示待编译的着色器功能。  

每个编译指令必须至少包含一个用于编译顶点和片段着色器的指令： #pragma vertex name, #pragma fragment name.  

默认情况下， Unity 将着色器编译为着色器模型 2.0。 #pragma target 指令可将着色器编译为其他能力级别。如果着色器变得较大，您将得到下列类型的错误：  

Shader error in 'Custom/MyShader': Arithmetic instruction limit of 64 exceeded; 83arithmetic instructions needed to compile program;  

如果出现这种情况，您必须添加 #pragma target 3.0 语句，将着色器模型 2.0 更改为着色器模型 3.0。着色器模型 3.0 拥有更高的指令限制。  

如果将多个变量从顶点着色器传递至片段着色器，可能会得到下列错误：  Shader error in 'Custom/MyShader': Too many interpolators used (maybe you want
\#pragma glsl?) at line 75.  

如果出现这种情况，请添加编译指令 #pragma glsl。该指令可将 Cg 或 HLSL 代码转换成 GLSL。\#pragma only_renderers 指令  

Unity 支持多种渲染平台，如 gles、 gles3、 opengl、 d3d11、 d3d11_9x、 xbox360、 ps3 和 flash。默认情况下，着色器可在所有上述平台上编译，除非您使用 #pragma only_renderers 指令明确限制此数字，该指令后跟有渲染器 API 且两者之间留有空格。  

如果您以移动设备为目标，则只需将着色器编译限制为 gles 和 gles3。您还必须添加 Unity 编辑器使用的opengl 和 d3d11 渲染器：  

\#pragma only_renderers gles gles3 [opengl, d3d11]  

#### include语句

可以在着色器中添加 Include 文件以利用 Unity 预定义的变量和辅助函数。  

您可以在 C:\Program Files\Unity\Editor\Data\CGIncludes 中找到可用include 语句。 例如，在 include 语句 UnityCG.cginc 中，您可找到若干个用于许多标准着色器的辅助函数和宏。 若要使用它们，请在着色器中声明 include。  

有许多 Unity 内置变量可用于着色器。 它们位于 include 文件 UnityShaderVariables.cginc 中。 您不需要将此文件包含在着色器中，因为 Unity 会自动执行此操作。多个有用的转换矩阵和幅度可直接用于着色器。必须了解上述内容，以避免重复工作。例如，在考虑如何将矩阵传递至着色器、摄像机位置、 投射参数或灯光参数前，请先检查 include 语句是否已提供矩阵。  

为提高性能，有时更偏向于在 CPU 中进行运算并将结果传递至 GPU，而非在顶点着色器中对每个顶点进行运算。例如，矩阵统一变量乘法运算就是这种情况。这就是为什么 Unity 可用作统一多个复合矩阵的内置工具。下表显示了一些重要的 Unity 着色器内置值：  

#### OpenGL ES 3.0 图形流水线  

知道可编程顶点着色器和片段着色器在图形流水线中的位置十分重要。  

![graph](..\img\armImprove\graph.png)

**原语汇编：**在原语汇编期间，顶点将汇编至几何原语中。产生的原语将剪切至裁剪区域并发送至光栅化程序。  

**光栅化：**针对每个生成的片段计算顶点着色器的输出值。该流程称为插值（用来填充图像变换时像素之间的空隙）。光栅化过程中，原语将转换为一组二维片段，这些片段将被发送至片段着色器。  

**转换反馈  ：**片段着色器采用通用编程方法在片段被发送至下一阶段前对其进行运算。  

**逐片段操作：  **在逐片段操作中，可以在每个片段上应用多项功能和测试：像素所有权测试、裁剪测试、模板和深度测试以及混合和抖动。因此，在逐片段阶段，片段将被弃置或者片段颜色、模板或深度值将写入屏幕坐标中的帧缓冲区。  

#### 顶点着色器

顶点着色器示例针对几何体的每个顶点运行一次。顶点着色器旨在将对象局部坐标中给定的每个顶点的 3D 位 置转换为屏幕空间中的投影 2D 位置，并计算 Z 缓冲区的深度值。  

  转换位置预期出现在顶点着色器的输出中。如果顶点着色器未返回值，控制台将显示下列错误：  

```
Shader error in 'Custom/ctTextured': '' : function does not return a value: vert at line 36
```

在此示例中，顶点着色器接收局部空间中的顶点坐标和纹理坐标以作为输入。顶点坐标通过作为Unity 内置值的模型视图投影矩阵 UNITY_MATRIX_MVP 从局部空间转换至屏幕空间：  

```
output.pos = mul(float4(input.normal, 0.0), _World2Object)
```

纹理坐标作为变量传递至片段着色器，但这并不意味着它们未被转换。

法线将以其它方式从对象空间转换到世界空间。为保证在执行不均匀缩放操作红，法线仍为三角形的法线，必须乘以转换矩阵的位置。若要应用转置操作，请在乘法算数中颠倒乘数因子的顺序。局部矩阵到世界矩阵  的逆矩阵是内置 World2Object Unity 矩阵。它是 4x4 矩阵，因此您必须通过 3 个分量的法线输入向量构建4 个分量。  

```
float4 normalWorld = mul(flaot4(input.normal, 0.0), _World2Object);
```

构建四个分量时，您要添加零作为第四个分量。这对于在第四个维度空间中正确处理矢量转换必不可少；而对于坐标而言，第四个分量必须是一个单位。  

如果世界坐标中已提供法线，则可以跳过法线转换流程。这将节省顶点着色器的工作量。如果对象网格可能由Unity 内置着色器进行处理，则避免此提示，因为此种情况下，法线将出现在对象坐标中。  

大多数图形效果在片段着色器中运行，但是您也可以在顶点着色器中运行一些效果。顶点置换贴图也称为置换贴图，是一项众所周知的技术，能够让您使用纹理对多边形网格进行变形，从而增加表面细节，例如在地形生成过程中使用高度贴图。若要在顶点着色器中获取此纹理，也称之为置换贴图，您必须添加编译指令 #pragma target3.0，因为它仅位于着色器模型 3.0 中。根据着色器模型 3.0，至少必须在顶点着色器内访问 4 个纹理单元。如果您强制编辑器使用 OpenGL 着色器，则还必须添加 #pragma glsl 指令。如果您未声明此指令，产生的错误消息会建议执行此操作：  

```
Shader error in 'Custom/ctTextured': function "tex2D" not supported in this profile (maybe you want #pragma glsl?) at line 57
```

在顶点着色器中，您还可以使用“过程动画”技术将顶点做成动画。您可将支持您修改顶点坐标的着色器中的时间变量用作时间函数。网格蒙皮是另一种在顶点着色器中实现的功能。 Unity 使用此技术，将与人物骨骼相关的网格的顶点做成动画。  

#### 顶点着色器输入

顶点着色器的输入和输出借助结构定义。在此示例的输入结构中，您仅发布了顶点属性位置和纹理坐标。您可以使用下列语义定义更多的属性作为输入，例如另一组纹理坐标、对象坐标中的法线、颜色和切线。  

```glsl
struct vertexInput
{
	float4 vertex:POSITION;
	float4 tangent:TANGENT;
	float3 normal:NORMAL;
	float4 texcoord:TEXCOORD0;
	float4 texcoord1:TEXCOORD1;
	fixed4 color:COLOR;
};
```

语义是着色器输入或输出随附的字符串，提供有关参数使用的信息。您必须为在着色器阶段之间传递的所有变量指定语义。  

如果您使用了不正确的语义，如 float3 tangent2 :TANGENTIAL，则会得到下列类型的错误：  

```
Shader error in 'Custom/ctTextured': unknown semantics "TANGENTIAL" specified for "tangent2" at line 32
```

为了提高性能，请仅在您确实需要的输入结构中指定参数。 Unity 拥有一些适用于最常用输入参数组合的预定义输入结构： appdata_base、 appdata_tan 和 appdata_full。 UnityCG.cginc include 文件中对这些结构进行了说明。前述顶点输入结构示例与 appdata_full 对应。在这种情况下，您无需声明结构，只需声明 include 文件。  

#### 顶点着色器输出和变量

顶点着色器输出在必须包含顶点转换坐标的输出结构中定义。在下列示例中，输出结构非常简单，但是您可以添加其他亮度。

下列代码列出了受Unity支持的语义：

```
struct vertexOutput
{
	float4 pos:SV_POSITION;
	float tex:TEXCOORD0;
	float4 texSpecular:TEXCOORD1;
	float3 vertexInWorld:TEXCOORD2;
	float3 viewDirInWorld:TEXCOORD3;
	float3 normalInWorld:TEXCOORD4;
	float3 vertexToLightInWorld:TEXCOORD5;
	float4 vertexInScreenCoords:TEXCOORD6;
	float4 shadowsVertexInScreenCoords:TEXCOORD7;
};
```

转换顶点坐标使用语义 SV_POSITION 进行定义。两个纹理、多个向量以及在不同空间中调用语义TEXCOORDn 的坐标也会传递至片段着色器。  

通常情况下， TEXCOORD0 为 UV 保留，而 TEXCOORD1 为光照贴图 UV 保留。但是从技术角度而言，你可以在 TEXCOORD0 到 TEXCOORD7 中存入任何内容，传递给片段着色器使用。必须注意，每个插值器（即每个语义） 只能处理最多 4 个浮点。将较大的变量（如矩阵）放入多个插值器中。这意味着，如果您将待传递的矩阵定义为变量： float4x4 myMatrix : TEXCOORD2， Unity 将使用 TEXCOORD2 至 TEXCOORD5 的插值器。  

默认情况下， Unity会对从顶点着色器发送到片段着色器的所有内容进行线性插值。对于通过顶点 V1、 V2 和 V3定义的三角形中的每个像素，位于顶点着色器和片段着色器之间的图形流水线中的光栅化程序将使用重心坐标λ1、 λ2 和 λ3 计算像素坐标，以作为顶点坐标的线性插值。  



![insertValue](..\img\armImprove\insertValue.png)

相同的插值适用于从顶点着色器传递至片段着色器的所有变量。这是一款功能非常强大的工具，因为它配备硬件线性插值器。例如，如果您拥有一个平面，并且想要将颜色作为与中心 C 距离的函数，请将中心 C 的坐标传递至顶点着色器，计算从顶点到 C 之间的平方距离，然后将该量度传递至片段着色器。距离值将自动内插至每个三角形的每个像素中。  

由于值可以线性内插，因此可以执行逐顶点计算并在片段着色器中重复利用这些计算，也就是说，可在片段着色器中线性插值的值可以在顶点着色器中进行计算。这能够大幅提高性能，因为顶点着色器运行的数据集远小于片段着色器运行的数据集。  

您必须谨慎使用变量，尤其是在移动平台上，因为性能和内存带宽消耗对众多游戏的成功至关重要。变量越多， 顶点获取的带宽以及片段着色器变量读取的带宽也就越多。使用变量时，请寻求一个合理的平衡。  

#### 片段着色器

片段着色器是原语光栅化后的图形流水线阶段。

对于原语覆盖的像素的每个示例，都会生成一个片段。系统将针对每个生成的片段执行片段着色器代码。由于片段的数量多于顶点数量，因此你必须注意在片段着色器中执行的运算量。

在片段着色器中，你除了可以获取窗口空间的片段坐标外，你还可以获取从顶点着色器中的到的每个顶点输出值的插值。

在着色器示例中，片段着色器接收来自顶点着色器的内插纹理坐标，并执行纹理查找以获取在这些坐标处的颜色。它将此颜色和环境颜色合并在一起，从而产生最终的输出颜色。通过声明片段着色器 float4 frag (vertexOutput input) :COLOR，很明显可以产生片段颜色。您可以在片段着色器中执行此操作，以达到预期效果。最终步骤是将正确颜色分配至片段。  

#### 像着色器提供数据

任何在 Pass 块中被声明为统一变量的数据均可用于顶点着色器和片段着色器。  

由于统一变量无法在着色器中进行修改，因此可以将统一变量视作一种全局常数变量。  

您可以通过以下方式向着色器提供此统一变量：  

* 使用Properties块
* 通过脚本编程

Properties块能够让你在检视面板中以交互方式定义统一变量。任何在Properties块中，声明的变量都在出现在材质检视面板中，并且会带有变量名称。

下列打码显示了与材质ctSphereMat相关的着色器示例Properties块：

```glsl
Properties
{
	_AmbientColor("Ambient Color", Color) = (0.2,0.2,0.2,1.0)
	_MainTex("Base (RGB)", 2D) = "white" {}
}
```

在 Properties 块中声明的名称为 Ambient Color 和 Base (RGB) 的变量 _AmbientColor 和_MainTex 分别出现在带有这些名称的材质检视面板中。  

当您处于着色器开发阶段时，使用 Properties 块将数据传递至着色器尤为有用，因为您可以在运行时以交互方式更改数据并查看效果。  

您可以将下列类型的变量放入 Properties 块  ：

* 浮点数
* 颜色
* 纹理2D
* 立方体贴图
* 长方形
* 向量

如果在之前计算中需要使用数据，或者需要在指定时间点传输数据，则Properties块不是一种非常有用的数据传递方法。

向着色器传递数据的另一种方法是根据脚本编程。

材质类显示了多种将材质相关数据传递至着色器的方法。如下：

![metarialMethod](..\img\armImprove\metarialMethod.png)

在下列代码中，主摄像机渲染完场景前，辅助摄像机 shwCam 会渲染阴影到将与主摄像机渲染通道合并的纹理。  

对于阴影纹理投影流程，必须以方便的方式转换顶点。阴影摄像机投影矩阵 (shwCam. projectionMatrix)、 世界到局部转换矩阵 (shwCam.transform.worldToLocalMatrix) 和渲染的阴影纹理(m_ShadowsTexture) 将传递至着色器。  

这些值可在着色器中用作为统一变量，其名称为 _ShwCamProjMat、 _ShwCamViewMat 和m_ShadowsTexture。  

下列代码显示了如何借助 shwMats 列表包含的材质将矩阵和纹理发送至着色器。  

```
// Called before object is rendered
public void OnWillRenderObject
{
	//Perform different checks.
	...
	CreateShadowsTexturn();
	//Set up shadows camrea shwCam
	...
	//Pass matrices to the shader
	for(int i = 0; i < shwMats.Count; i++)
	{
		shwMats[i].SetMatrix("_ShwCamProjMat", shwCam.projectionMatrix);
		shwMats[i].SetMatrix("_ShwCamViewMat", shwCam.transform.worldToLocalMatrix);
	}
	//Render shadows texture
	shwCam.Render();
	for(int i = 0; i < shwMats.Count; i ++)
	{
		shwMats[i].SetTextrue("_ShadowsTex", m_ShadowsTexture);
	}
	s_RenderingShadows = false;
}
```

#### 调试Unity中的着色器

在Unity中，无法以处理传统代码的方式调试着色器。但是，你可以用片段着色器的输出使待调试的值形象化。然后，你必须解析产生的图像。

例如可以反射地板，提供的地板反射面的着色器ctReflLocalCubemap.shader 的输出  。

![room1](..\img\armImprove\room1.png)

在下例片段着色器中，输出颜色已替换为标准化的局部修正正反射向量：

```glsl
return float4(normalize(localCorrRefDirWS), 1.0);
```

它使标准化为颜色的反射向量分量（而非反射图像）得以形象化。

地板的红色区域指示反射向量拥有很强的 X 分量，也就是说，它大部分都指向 X 轴方向。红色区域显示来自带窗墙壁的反射。  

蓝色区域表示指向 Z 轴的反射向量（即来自右侧墙壁的反射）最为显著。  

在黑色区域中，向量主要指向 -Z，但是颜色只能拥有正值分量，因为负值分量已固定为 0。  

下图显示了将片段输出颜色替换为标准化的局部修正反射向量的结果  ：

![room2](..\img\armImprove\room2.png)

最初可能很难在调试时解析各颜色的含义，因此请尝试专注于单一颜色分量。例如，您只能返回标准化局部修正反射向量的 Y 分量：  

```glsl
float3 normalLocalCorrReflDirWS = normalize(localCorrReflDirWS);
return float4(0, normLocalCorrReflDirWS.y, 0 ,1)
```

在这种情况下，输出只是主要来自摄像机上方屋顶的反射。也就是说，房间中指向 Y 轴的部分。房间墙壁的反射来自 X、 Z 和 -Z 方向，所以它们以黑色渲染。

  下图显示了使用单一颜色的着色器调试：

![room3](..\img\armImprove\room3.png)

检查使用颜色调试的幅度是否介于 0 和 1 之间，因为所有其他值已自动固定。所有负值均指定为 0，而所有大于 1 的值均指定为 1。  

### 使用局部立方体贴图实施反射

对于在移动设备上渲染反射，基于局部立方体贴图的反射是很有用的技巧。  

Unity 版本 5 和更高版本将基于局部立方体贴图的反射实施为反射探测器。您可以将它们与其他类型的反射组合，如运行时在您的自定义着色器中渲染的反射。  

#### 反射实施的历程  

图形开发人员一直在尝试寻找计算量较小的反射实现方法。  

第一批解决方案中的其中一个便是球形贴图。此方案无需通过计算量大的射线跟踪或光照计算来模拟对象的反射或光照。  

![te1](..\img\armImprove\te1.png)

此方法有多种劣势，但是主要问题在于，将图片贴图至球体时会出现失真。 1999 年，立方体贴图与硬件加速可结合使用。立方体贴图解决了与球形贴图相关的图像失真、视角依赖性以及低效计算等问题。  

![cube5](..\img\armImprove\cube5.png)

立方体贴图使用立方体的六个面作为贴图形状。环境投射到立方体的每个面并存储为六个正方纹理，或展开为单个纹理的六个区域。可以使用六个不同的摄像机方位从指定位置渲染场景，以生成立方体贴图， 90 度的视锥体代表每个立方体表面。源图像将被直接采样。对中间环境贴图重新采样不会产生任何失真。  

![ref](..\img\armImprove\ref.png)

若要实现基于立方体贴图的反射，请评估反射向量 R 并用该向量从立方体贴图 _Cubemap 中获取像素元，方法是使用可用纹理查找函数 texCUBE()：  

```glsl
float4 color = texCUBE(_Cubemap, R);
```

法线 N 和视线向量 D 可从顶点着色器传输至片段着色器。片段着色器从立方体贴图中获取纹理颜色：  

```glsl
float3 R = reflect(D,N);
float4 color = texCUBE(_Cubemap, R);
```

此方法只能从立方体贴图位置不重要的远距离环境中复制反射。这种简单有效的技术主要用于室外光照，例如增加天空的反射效果。  

下图展示不正确的反射：

![arm1](..\img\armImprove\arm1.png)

如果在局部环境中使用此技术，则会产生不正确的反射。反射不正确的原因在于，在表达式 float4 color = texCUBE(_Cubemap, R); 中，没有与局部几何结构绑定。例如，如果您走过反射地板并从同一角度看它， 您将始终看到相同的反射。反射向量始终相同，并且该表达式始终得出相同的结果。这是因为视线向量的方向未发生变化。在真实的世界中，反射取决于观察角度和位置。  

#### 使用局部立方体贴图生成正确的反射  

此问题的解决方案涉及在程序中与局部几何结构绑定以计算反射。  

此解决方案在《GPU 精粹：实时图形编程的技术、技巧和技艺》（作者： Randima Fernando（丛书编辑））进行了介绍  。

下图显示了使用包围球体进行局部修正：  

![mod](..\img\armImprove\mod.png)

包围球体用作界定待反射场景的代理区域。它不使用反射向量 R 从立方体贴图中获取纹理，而是使用新向量R’。为构建此新向量，您需要在反射向量 R 方向的局部点 V 中找到射线包围球体的相交点 P。再根据生成立方体贴图的立方体贴图 C 的中心创建相交点 P 的新向量 R’。然后使用此向量获取立方体贴图的纹理。  

```
float3 R = reflect(D, N);
Find intersection point P
Find vector R' = CP
float4 col = texCUBE(_Cubemap, R');
```

此方法将在近球形的对象表面中产生较好的结果，但是平面反射表面的反射将会失真。此方法的另一缺点在于，用于计算与包围球体的相交点的算法需要解一元二次方程式，此过程非常复杂。  

下图显示了使用包围盒进行局部修正：

![modi](..\img\armImprove\modi.png)

**无限立方体贴图**:

* 主要用于室外，代表远距离环境的光照
* 立方体贴图位置不重要。

**本地立方体贴图：**

* 代表有限局部环境的光照
* 立方体贴图的位置重要。
* 这些立方体贴图的光照仅在立方体贴图的创建位置才正确
* 必须运用局部修正来调整立方体贴图的固有无限性，以使其适应局部环境。

#### 着色器实施

点着色器用于计算作为内插值传输至片段着色器的三个幅度：  

* 顶点位置
* 观察方向
* 法线

这个值位于世界坐标中。

下列代码显示了使用局部立方体贴图的反射的着色器实施，它适用于 Unity。  

```glsl
vertexOutput vert(vertexInput input)
{
	vertexOutput output;
	output.tex = input.texcoord;
	// Transform vertex coordinates from local to world
	float4 vertexWorld = mul(_Object2World, input.vertex);
	//Transform normal to world coordinates.
	float4 normalWorld = mul(float4(input.normal, 0.0), _World2Object);
    // Final vertex output position. output.pos = mul(UNITY_MATRIX_MVP, intput.vertex);
    //----------Local correction -----------
    output.vertexInWorld = vertexWorld.xyz;
    output.viewDirInWorld = vertexWorld.xyz - _WorldSpaceCameraPos;
    output.normalInWorld = normalWorld.xyz;
    return output;
}
```

区域盒中的相交点以及反射向量在片段着色器中计算。您将构建新的局部修正反射向量并用该向量从局部立方体贴图中获取反射纹理。然后，您将结合纹理和反射生成输出颜色：  

```glsl
float4 frag(vertexOutput input):COLOR
{
	float4 reflColor = float4(1,1,0,0);
	//Find reflected vector in WS
	float3 viewDirWS = normalize(input.viewDirInWorld);
	float3 normalWS = normalize(input.normalInWorld);
	float3 reflDirWS = relect(viewDirWS, normalWS);
	//Working in World Coordinate System.
	float3 localPosWs = input.vertexInWorld;
	float3 intersectMaxPointPlanes = (_BBoxMax - localPosWS)/ refDirWS;
	float3 intersectMinPoitPlanes = (_BBoxMin - localPosWS)/ refDirWS;
	//Looking only for intersections in the forward direction of the ray.
	float3 largestParams = max(intersectMaxPointPlanes, intersectMinPointPlanes);
	//Smalleset value of the ray parameters gives us the intersection.
	float distToIntersect = min(min(largestParams.x, largestParams.y), largestParams.z);
	//Find the position of the intersection point
	float3 intersectPointWS = localPosWS + reflDirWS * disToIntersect;
	//Get local corrected reflection point
	float3 localCorrReflDirWS = intersectPositionWS - EnvoCubeMapPos;
	//Lookup the environment reflection texture with the right vector
	reflColor = texCUBE(_Cube, localCorrReflDirWS);
	//Lookup the texrure color
	float4 texColor = tex2D(_MainTex, float2(input.tex));
	return _AmbientColor + texColor * _reflAmount * reflColor;
}
```

在前述片段着色器代码中，幅度 _BBoxMax 和 _BBoxMin 是包围区域的最大点和最小点。变量_EnviCubeMapPos 是立方体贴图的创建位置。请通过下列脚本将这些值传输至着色器：  

```c#
[ExecuteInEditMode]
public class InfoToReflMaterial:MonoBehaviour
{
    //The proxy volume used for local reflection calculations.
    public GameObject boundingBox;
    void Start()
    {
        Vector3 bboxLength = boundingBox.transform.localScale;
        Vector3 centerBox = boundingBox.tranform.positionl;
        
        //Min and max BBox points in world coordinates.
        Vector3 BMin = centerBBox - bboxLenght / 2;
        Vector3 BMax = centerBBox + bboxLenght / 2;
        
        //Pass the values to the material.
        gameObject.renderer.shareMaterial.SetVector("_BBoxMin", BMin);
        gameObject.renderer.shareMaterial.SetVector("BBoxMax", BMax);
        gameObject.renderer.shareMaterial.SetVector("_EnviCubeMapPos", centerBBox);
    }
}
```

将 `_AmbientColor、_ReflAmount`，主纹理以及立方体贴图纹理的值传输至着色器，作为属性块的统一变量：

```glsl
Shader "Custom/ctReflLocalCubemap"
{
	Properties
	{
		_MainTex("Baes (RGB)", 2D) = "white" {}
		_Cube("Reflection Map", Cube) = ""{}
		_AmbientColor("Ambient Color", Color) = (1,1,1,1)
		_ReflAmount("Reflection Amount", Float) = 0.5
	}
    SubShader
    {
        Pass
        {
            CGPROGRAM
            #pragma glsl
            #pragma vertex vert
            #pragma fragment frag
            #include "UnityCG.cginc"
            // User-specified uniforms
            uniform sampler2D _MainTex;
            uniform samplerCUBE _Cube;
            uniform float4 _AmbientColor;
            uniform float _ReflAmount;
            uniform float ToggleLocalCorrection;
            // ---Passed from script InfoRoReflmaterial.cs ---
            uniform float3 _BBoxMin;
            uniform float3 _BBoxMax;
            uniform float3 _EnviCubeMapPos;
            
            struct vertexInput
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float4 texcoord :TEXCOORD0;
            };
           	struct vertexOutput
            {
                float4 pos:SV_POSITION;
                float4 tex:TEXCOORD0;
                float3 vertexInWorld:TEXCOORD1;
                float3 viewDirInWorld:TEXCOORD2;
                float3 noramlInWorld:TEXCOORD3;
			};
            Vertex shader {}
            Fragment shader{}
            ENDCG
        }
    }
}
```

计算包围区域相交点的算法基于使用参数表示局部位置或片段的反射射线。  

#### 过滤立方体贴图  

使用局部立方体贴图实现反射的其中一项优势就是立方体贴图为静态。也就是说，它在开发时生成，而非在运行时生成。这样便能够对立方体贴图图像应用任何过滤来实现某一效果。  

CubeMapGen 是 AMD 提供的用于对立方体贴图应用过滤的工具。您可从 AMD 开发人员网站上获取CubeMapGen.

若要将 Unity 中的立方体贴图图像导出至 CubeMapGen，您必须单独保存每张立方体贴图图像。  此工具不仅能够创建立方体贴图，而且还可选择性地单独保存每张立方体贴图图像。 



您必须将此工具的脚本存储于 Asset 目录的 Editor 文件夹中。   

如何使用立方体贴图编辑器工具：  

1. 创建立方体贴图
2. 从"游戏对象"菜单中启动烘焙立方体贴入工具。
3. 提供立方体贴图和摄像机渲染位置。
4. 如果您计划对立方体贴图应用过滤。请选择性的保存每张图像。

您可以使用 CubeMapGen 为立方体贴图单独加载每张图像。  

从选择立方体表面下拉菜单中选择要加载的表面，然后按下加载立方体表面按钮。加载完所有表面后，可旋转立方体贴图并检查其是否正确。  

CubeMapGen 在过滤类型下拉菜单中拥有众多不同的过滤选项。选择您需要的过滤设置，然后按下过滤立方体贴图，应用过滤器。过滤过程可能会花费数分钟，具体取决于立方体贴图的大小。由于没有撤消选项，因此在应用任何过滤之前，请将立方体贴图保存为独立图像。如果过滤结果与您期待的不一样，您可以重新加载立方体贴图并尝试调整参数  。

按照下列步骤将立方体贴图图像导入 CubeMapGen：  

1. 选中复选框，在烘焙立方体贴图时保存独立图像。
2. 启动CubeMapGen工具，并按照下表所示关系加载立方体贴图图像。
3. 立方体贴图保存为单个dds 或立方体交叉图像。由于无法撤消，因此这使您能够在使用过滤器进行试验的情况下重新加载立方体贴图。  
4. 根据需要对立方体贴图应用过滤器，直到结果满意为止。  
5. 将立方体贴图保存为独立图像。  

#### 射线与盒求交算法  

直线方程：
$$
y = mx + b
$$
该方程的向量形式
$$
\vec r = \vec o + t * \vec D
$$
其中o代表原点，D代表方向矢量，t代表参数。

一个包围盒可以通过最小点 A 和最大点 B 定义轴对齐包围盒 AABB。  

AABB 定义了一组与坐标轴平行的直线。可使用下列方程定义每条直线：  
$$
x = A_x; y = A_y; z = A_z \\
x = B_x; y = B_y; z = B_z \\
$$
若要找到射线与其中一条直线的相交点，只需要这两个方程相等。例如：
$$
O_x + t_x * D_x = A_x
$$
你可以将解答方程写成：
$$
t_{Ax} = (A_x - O_x) / D_x
$$
以相同方式求出这两个相交点的所有可能解：
$$
t_{Ax} = (A_x - O_x) / D_x \\
t_{Ay} = (A_y - O_y) / D_y \\
t_{Az} = (A_z - O_z) / D_z \\
t_{Bx} = (B_x - O_x) / D_y \\
t_{By} = (B_y - O_y) / D_y \\
t_{Bz} = (B_z - O_z) / D_z \\
$$
向量形式的方程为：
$$
t_A = (A - O) / D \\
t_B = (B - O) / D
$$
这可找出直线与立方体表面定义的平面相交的位置，但是它无法保证相交点位于立方体上。  

下图显示了射线与盒相交点的 2D 表示：  

![xj](..\img\armImprove\xj.png)

若要找到哪种答案确实是与盒的相交点，您需要用参数 t 中较大的值来确认最小平面上的相交点。  
$$
t_{min} = (t_{Ax} > t_{Ay})? t_{Ax}:t_{Ay}
$$
您需要用参数 t 中较小的值来确认最大平面上的相交点。  
$$
t_{min} = (t_{Ax} > t_{Ay})? t_{Ax}:t_{Ay}
$$
如果您未得到任何相交点，也必须考虑这些情况。  

下图显示了无相交点的射线与盒：  

![xj2](..\img\armImprove\xj2.png)

如果您保证反射面已被BBox 包围，即反射线的来源位于BBox 中，则始终存在两个与该盒相交的相交点，并
且处理不同情况的流程也会简化  

下图显示了 BBox 中的射线与盒相交点：  

![xj3](..\img\armImprove\xj3.png)

#### 用于使编辑器脚本生成立方体贴图的源代码  

本节提供的源代码可以供编辑器脚本生成立方贴图  。

```c#
using UnityEngine;
using FairyGUI;
using UnityEditor;
using System.IO;
/// <summary>
/// This script must be placed in the Editor folder.
/// The script renders the scene into a cubemap and optionally
/// saves each cubemap image individually
/// The script is available in the Editor mode from the
/// Game Object menu as "Bake Cubemap" option.
/// Be sure the camera for plane is enough to render the scene.
/// </summary>
public class BakeStaticCubemap:ScriptableWizard
{
    public Transform renderPosition;
    public Cubemap cubemap;
    //Camera setting
    public int cameraDepth = 24;
    public LayerMask cameraLayerMask = -1;
    public Color cameraBackgroundColor;
    public float cameraNearPlane = 0.1f;
    public float cameraFarPlane = 2500.0f;
    public bool cameraUseOcclusion = true;
    //Cube setting.
    public FilterMode cubemapFilterMode = FilterMode.Trilinear;
    //Quality setting.
    public int antiAliasing = 4;
    public bool createIndividualImages = false;
    //The folder where individual cubemap images will be saved
    static string imageDirectory = "Assets/CubemapImages";
    static string[] cubemapImage = new string[] { "front+Z", "right+X", "back-Z", "left-X", "top+Y", "bottom-Y" };
    static Vector3[] eulerAngles = new Vector3[] {new Vector3(0.0f, 0.0f, 0.0f),
                                                  new Vector3(0.0f, -90.0f, 0.0f),
                                                  new Vector3(0.0f, 180.0f, 0.0f),new Vector3(0.0f, 90.0f, 0.0f),
                                                  new Vector3(-90.0f, 180.0f, 0.0f),new Vector3(90.0f, 90.0f, 0.0f),
                                                 };
    private void OnWizardUpdate()
    {
        helpString = "Set the positon to render from and the cubemap to back";

        if (renderPosition != null && cubemap != null)
        {
            isValid = true;
        }
        else
        {
            isValid = false;
        }
    }

    private void OnWizardCreate()
    {
        //Create temporary camera for rendering.
        GameObject go = new GameObject("CubemapCam", typeof(Camera));
        Camera camera = go.GetComponent<Camera>();
        //Camera setting
        camera.depth = cameraDepth;
        camera.backgroundColor = cameraBackgroundColor;
        camera.cullingMask = cameraLayerMask;
        camera.nearClipPlane = cameraNearPlane;
        camera.farClipPlane = cameraFarPlane;
        camera.useOcclusionCulling = cameraUseOcclusion;
        //Cubemap setting
        cubemap.filterMode = cubemapFilterMode;
        //Set antialiasing
        QualitySettings.antiAliasing = antiAliasing;

        //Place the camera on the render position
        go.transform.position = renderPosition.position;
        go.transform.rotation = Quaternion.identity;

        //Bake the cubemap
        camera.RenderToCubemap(cubemap);
        //Rendering individual images
        if (createIndividualImages)
        {
            if (!Directory.Exists(imageDirectory))
            {
                Directory.CreateDirectory(imageDirectory);
            }
            RenderIndividualCubemapImages(go);
        }

        //Destroy the camera after rendering.
        DestroyImmediate(go);
    }

    void RenderIndividualCubemapImages(Camera camera)
    {
        camera.backgroundColor = Color.black;
        camera.clearFlags = CameraClearFlags.Skybox;
        camera.fieldOfView = 90;
        camera.aspect = 1.0f;
        camera.gameObject.transform.rotation = Quaternion.identity;

        //Render individual images
        for (int camOrientation = 0; camOrientation < eulerAngles.Length; camOrientation++)
        {
            string imageName = Path.Combine(imageDirectory, cubemap.name + "_"
                                            + cubemapImage[camOrientation] + ".png");
            camera.transform.eulerAngles = eulerAngles[camOrientation];
            RenderTexture renderTex = new RenderTexture(cubemap.height, cubemap.height, cameraDepth);
            camera.targetTexture = renderTex;

            Texture2D img = new Texture2D(cubemap.height, cubemap.height, TextureFormat.RGB24, false);
            img.ReadPixels(new Rect(0, 0, cubemap.height, cubemap.height), 0, 0);
            RenderTexture.active = null;
            GameObject.DestroyImmediate(renderTex);
            byte[] imgBytes = img.EncodeToPNG();
            File.WriteAllBytes(imageName, imgBytes);
            AssetDatabase.ImportAsset(imageName, ImportAssetOptions.ForceUpdate);
        }
        AssetDatabase.Refresh();
    }

    [MenuItem("GameObject/Bake Cubemap")]
    static void RenderCubemap()
    {
        ScriptableWizard.DisplayWizard("Bake Cubemap", typeof(BakeStaticCubemap), "Bake!");
    }
}
```

### 组合放射

基于局部立方体贴图的放射技巧实现了基于静态局部立方体贴图渲染高质量、高效率的反射。但是，如果对象是动态的，静态局部立方体贴图便不再有效，该技巧也无法发挥作用。

你可以将静态反射与动态生成的反射相组合，从而解决此问题。

如果反射表面是平面，您可以使用镜像摄像机生成动态反射。

要创建镜像摄像机，可计算在运行时渲染反射的主摄像机的位置和指向。  

相对于反射平面，镜像主摄像机的位置和指向  。

在镜像过程中，新的反射摄像机最终表现为其轴在相对的指向。与现实中的镜子一样，左右的反射被颠倒。这意味着反射摄像机使用相反的卷绕渲染几何体。  

要正确渲染几何体，您必须逆转几何体卷绕，然后再渲染反射。完成了反射渲染时，恢复原始卷绕。  

构建镜像反射变换矩阵。使用此矩阵计算反射摄像机的位置和世界至摄像机变换矩阵。  

将反射矩阵变换应用到主摄像机的位置和世界至摄像机矩阵。这将为您提供反射摄像机的位置和世界至摄像机矩阵。  

反射摄像机的投影矩阵必须和主摄像机的投影矩阵相同。  

反射摄像机将反射渲染到纹理。  

为获得良好的结果，在渲染之前必须先正确设置此纹理：  

* 使用纹理映射
* 将过滤模式设置为三线性
* 使用多重采样

确保纹理大小与反射表面的面积成正比。纹理越大，反射的像素化程度越低。

#### 组合反射着色器实施

您可以组合着色器中的静态环境反射和动态平面反射。  

着色器必须融合平面反射，在运行时通过反射摄像机渲染。要达到此目的，来自反射摄像机的纹理_ReflectionTex 作为统一变量传递到片段着色器，再使用 lerp() 函数与平面反射结果组合。  

除了与局部修正相关的数据外，顶点着色器还要使用内置函数 ComputeScreenPos() 计算顶点的屏幕坐标。

它将这些坐标传递给片段着色器：  

```glsl
vertexOutput vert(vertexInput input)
{
	vertexOutput output;
	output.tex = input.texcoord;
	//Transform  vertex coordinates from local to world.
	float4 vertexWorld = mul(_Object2World, input.vertex);
	//Transform normal to world coordinates.
	float4 normalWorld = mul(float4(input.normal,0.0),_World2Object);
	// Local correction
	output.vertexInWorld = vertexWorld.xyz;
	output.viewDirInWorld = vertexWorld.xyz - _WorldSpaceCameraPos;
	output.normalInWorld = normalWorld.xyz;
	// Planar reflections
	output.vertexInScreenCoords = ComputeScreenPos(output.pos);
	return output;
}
```

平面反射渲染至纹理，让片段着色器能够访问片段的屏幕坐标。为此，需要将顶点屏幕坐标作为变量传递到片段着色器。

在片段着色器中：

* 向反射向量应用局部修正。
* 从局部立方体贴图检索环境反射的颜色 staticRefIColor。

下列代码显示了如何将使用局部立方体贴图技巧的静态环境反射与运行时使用镜像摄像机技巧渲染的动态平面反射相组合：  

```
float4 frag(vertexOutput input):COLOR
{
	float4 staticReflColor = float4(1,1,1,1);
	
	//Find reflected vector in WS.
	float3 viewDirWS = normalize(input.viewDirInWorld);
	float3 normalWS = normalize(input.normalInWorld);
	float3 reflDirWS = reflect(viewDirWS, normalWS);
	
	//Working in world Coordinate System
	float3 localPosWS = input.vertexInWorld;
	float3 intersectMaxPointPlanes = (_BBoxMax - localPosWS) / reflDirWS;
	float3 intersectMinPointPlanes = (_BBoxMin - localPosWS) / refDirWS;
	
	//Look only for intersections in the forward direction of the ray.
	float3 largestParams = max(intersectMaxPointPlanes, intersectMinPointPlanes);
	
	//Smallest value of the ray parameters gives us the intersction.
	float distToIntersect = min(min(largestParams.x, largestParams.y), largestParams.z);
	
	//Find the position of the intersection point.
	float3 intersectPositionWS = localPosWS + reflDirWS * distToIntersect;
	//Get local corrected reflection vector.
	float3 localCorrReflDirWS = intersectPositionWS - _EnviCubeMapPos;
	
	//Lookup the environment reflection texture with the right vector.
	float4 staticReflColor = tex2Dproj(_ReflectionTex, UNITY_PROJ_COORD(input.vertexInScreenCOoords));
	//Revert the blending with the background color of the reflection camera 
	dynReflColor.rgb /= (dynReflColor.a < 0.00392)?1:dynReflColor.a;
	//Combine static environment reflections with synamic planar reflections
	float3 combinedRefl = lerp(staticReflColor.rgb, dynReflColor.rgb,synReflColor.a);
	//Lookup the texture color.
	float4 texColor = tex2D(_MainTex, float2(input.tex));
	return _AmbientColor + texColor * _ReflAmount * combiendRefl;
}
```

从平面运行时反射纹理 _ReflectionTex 提取纹理颜色 dynReflColor。  在着色器中将 _ReflectionTex 声明为统一变量。 

 在 Property 块中声明 _ReflectionTex。这可使您能够查看它在运行时的外观，从而帮助您在开发游戏期间进行调试  。

为查找纹理，可投射纹理坐标，即：将纹理坐标除以坐标向量的最后一个分量。您可以使用 Unity 内置函数UNITY_PROJ_COORD() 进行此操作。  

使用 lerp() 函数组合静态环境反射和动态平面反射。组合以下所列：  

* 反射颜色
* 反射表面的纹理颜色
* 环境颜色分量

#### 组合远距离环境的反射

在渲染静态和动态对象的反射时，您可能也必须考虑来自远距离环境的反射。例如，通过局部环境中某一窗户可见的天空反射。  

在这种情形中，你必须组合三种不同的反射：

* 来自静态环境的反射，它使用的是局部立方体贴图技巧。
* 来自动态对象的平面反射，它使用的是镜像摄像机技巧。
* 来自天空盒的反射，它使用的是标准立方体贴图技巧。反射向量在从立方体贴图获取纹理前不需要修正。

要整合来自天空盒的反射，可使用反射向量 reflDirWS 从天空盒立方体贴图获取像素元。将天空盒立方体贴图纹理作为统一变量传递到着色器。  

为确保天空盒仅可从窗户可见，请在为反射烘焙静态立方体贴图时在 alpha 通道渲染场景的透明度。  

如果是不透明几何体，分配值一；如果没有几何体或几何体全部透明，分配值零。例如，在alpha 通道中使用零渲染与窗户对应的像素。  

将天空盒立方体贴图 _Skybox 作为统一变量传递到着色器。  

在shader代码 //Lookup the planar runtime texture 前插入几行代码：

```glsl
float4 skyboxReflColor = texCUBE(_Skybox, reflDirWS);
staticReflColor = lerp(skyboxReflColor.rgb, staticReflColor.rgb, staticReflColor.a);
```

此代码将静态反射与来自天空盒的反射组合。  

### 基于局部立方体贴图的动态软阴影

此技巧使用局部立方体贴图保存代表静态环境透明度的纹理。此技巧在生成高质量软阴影时效率非常高。  

#### 关于基于局部立方体贴图的动态软阴影  

在您的场景中，既存在移动的对象，也有房间等静态环境。通过使用此技巧，您不必对每一帧渲染静态几何体到阴影贴图。这可让您通过纹理来表示阴影。  

立方体贴图可以很好地逼近许多种类的静态局部环境，例如冰穴演示中的洞穴等不规则形状。 Alpha 通道也可表示进入房间的光线量。  

运动的对象通常是除了房间外的一切。这些对象包括：  

* 太阳
* 摄像机
* 动态对象

通过用立方体纹理来表示整个房间，您可以在一个片段着色器内访问环境的任意像素元。例如，这意味着太阳可以在任意位置上，你也可以根据从立方体贴图获取的值计算照射片段上的光线量。

Aipha通道或透明度可以表示进入房间的光线量。在你的场景中，将立方体贴图纹理附加到片段着色器上，这些着色器渲染你要为其添加阴影的静态和动态对象。

#### 生成阴影立方体贴图

首先着手于您要向其应用来自环境外部光源的阴影的局部环境。例如，房间、洞穴或笼子。  

此技巧类似于基于局部立方体贴图的反射。  

创建阴影立方体贴图的方式与创建反射立方体贴图相同，但您还必须添加alpha 通道。 Alpha 通道或透明度可以表示进入房间的光线量。  

算出您要从中渲染立方体贴图六个面的位置。在大多数情形中，这是局部环境包围盒的中心。您需要此位置来生成立方体贴图。还必须将此位置传递到着色器，从而计算局部修正向量，以便从立方体贴图获取正确的像素元。  

决定了立方体贴图中心的位置后，您可以将所有面渲染到立方体贴图纹理，再记录局部环境的透明度或alpha通道。一个区域的透明度越高，进入环境中的光线越多。如果没有几何体，则完全透明。必要时，您可以使用RGB 通道存储彩色阴影的环境颜色，如有色玻璃、反射或折射。  

#### 渲染阴影

在世界空间中构建从顶点或片段到一个/多个光源的向量 PiL， 然后使用此向量获取立方体贴图阴影。  

在获取每个像素元前，必须对 PiL 向量应用局部修正。 ARM 建议在片段着色器中进行局部修正，从而获得更加准确的阴影。  

要计算局部修正，您必须计算片段至光源向量与环境包围盒的交叉点。使用此交叉点构建从立方体贴图原点位置C 到交叉点 P 的另一向量。这可为您提供用于获取像素元的最终向量 CP。  

您需要下列输入参数来计算局部修正：  

* _EnviCubeMapPos 立方体贴图原点位置。
* _BboxMax 环境包围盒的最大点。
* _BboxMin环境包围盒的最小点。
* Pi世界空间中的片段位置。
* PiL 世界空间中正规化片段至光源向量。

计算输出值CP。这是修正后的片段至光源向量，你要用它从阴影立方体贴图获取像素元。

下列示例代码演示了如何正确计算 CP 向量：  

```glsl
// Working in World Coordinate System.
vec3 = intersectMaxPointPlanes = (_BBoxMax - Pi) / PiL;
vec3 = intersectMinPointPlanes = (_BBoxMin - Pi) / Pil;
//Looking only for intersections in the forward direction of the ray.
vec3 largestRagParams = max(intersectMaxPointPlanes, intersectMinPointPlanes);
//Smallest value of the ray Parameters gives us the intersection.
float dist = min(min(largestRayParams.x, largestRayParams.y), largestRayParams.z);
//Find the positon of the intersection point.
vec3 intersectPositonWS = pi + piL * dist;
//Get the local corrected vector.
CP = intersectPositionWS - _EnviCubeMapPos;
```

使用CP向量从立方体贴入获取像元素元。像原素元的alpha通道提供关于必须向片段应用多少光线或阴影的信息：

```glsl
float shadow = texCUBE(cubemap, CP).a;
```

此技巧可以在场景中生成有效的阴影，但您可以通过两个额外的步骤来提高阴影的质量：  

* 阴影的背面
* 平滑

#### 阴影中的背面

立方体贴图阴影技巧不使用深度信息来应用阴影。这意味着某些面在应当要处于阴影中时会得到错误的照明。  

只有表面朝向光源相反的方向时才会出现此问题。要解决此问题，可检查法线向量与片段至光源向量 PiL 之间的角度。如果角度数超出 -90 到 90 度范围，该表面处于阴影中。  

下列代码片段进行此检查：  

```
if(dot(PiL,N)<0)
	shadow = 0.0;
```

以上代码导致每个三角形从亮硬切换到至暗。若要获得平滑过渡，可使用下列公式：

shadow *= max(dot(PiL, N) , 0.0);

其中：

* shadow是从阴影立方体贴图获取的alpha值。
* PiL是世界空间中的正规化片段至光源向量。

**平滑**

此阴影技巧可以在您的场景中提供逼真的软阴影。  

1. 生成纹理映射，并为立方体贴图纹理设置三线性过滤。
2. 测量片段至交叉点向量的长度。
3. 将该长度乘以一个系数。

该系数是环境中最大距离与纹理映射层级数的正规化子。  您可以使用包围区域和纹理映射层级数自动计算它。您必须按照自己的场景自定义该系数。这可让您调整相关的设置，使它适合您的环境，从而改善视觉质量。例如，冰穴项目中使用的系数为 0.08。  

您可以重新利用为局部修正所进行的计算的结果。重新利用代码片段中局部修正的 dist，作为从片段位置到片段至光源向量与包围盒交叉点的线段的长度：  

```glsl
float texLod = dist;
```

将texLod乘以距离系数：

texLod *= distanceCoeficient;

要实施柔和度，使用Cg函数texCUBElod()或GLSL函数textureLod()获取纹理的正确纹理映射层级。

构造一个vec4，其中XYZ代表方向矢量，W分量则代表LOD。

```glsl
CP.w = texLod;
shadow = texCUBElod(cubemap,CP).a;
```

#### 组合立方体贴图阴影与阴影贴图

要利用动态内容完善阴影，必须组合使用立方体贴图阴影和传统的阴影贴图技巧。这需要额外的工作，但值得一试，因为您只需要将动态对象渲染到阴影贴图。  

#### 立方体贴图阴影技巧的结果

使用传统的技巧时，渲染阴影的成本比较高，因为它涉及从每个阴影投射光源的视角渲染整个场景。此处介绍的立方体贴图阴影技巧可以提高性能，因为它大部分是预烘焙的。  

此技巧还独立于输出分辨率。它在 1080p、 720p 和其他分辨率上产生相同的视觉质量。  

柔和度过滤是在硬件中计算的，所以平滑几乎没有计算上的成本。阴影越平滑，此技巧的效用越佳。这是因为较小的纹理映射层级数产生的数据要少于传统的阴影贴图技巧。传统的技巧需要较大的内核才能使阴影足够平滑，从而在视觉上更吸引人。这需要很高的内存带宽，所以会降低性能。  

通过立方体贴图阴影技巧获得的质量可能会超越您的预期。它提供逼真的柔和度，阴影稳定，没有闪光的边缘。 由于光栅化和锯齿效应，使用传统的阴影贴图技巧时可能会看到闪光的边缘。不过，所有抗锯齿算法都不能完全修复此问题。  

立方体贴图阴影技巧没有闪光问题。边缘稳定，即使您使用的分辨率远低于渲染目标所用的分辨率。您可以使用比输出低四倍的分辨率，而且没有失真或多余的闪光。使用低四倍的分辨率也可节省内存带宽，因而能提升性能。  

此技巧可用于市面上支持着色器的任何设备，比如支持 OpenGL ES 2.0 或更高版本的设备。如果您已清楚使用基于局部立方体贴图技术的反射的位置和时机，您就可以轻松在实施中应用此阴影技巧。  

该技巧无法用于场景中的一切对象。例如，动态对象从立方体贴图接收阴影，但它们无法预烘焙到立方体贴图纹理。对于动态对象，请使用阴影贴图来生成阴影，并与立方体贴图阴影技巧搭配使用。  

### 基于局部立方体贴图的折射  

您可以使用局部立方体贴图实施高质量折射，也可以在运行时将它们与反射组合。  

#### 关于折射

游戏开发人员经常寻觅高效的方法，在游戏中实施夺人眼球的视觉特效。以移动平台为目标开发时，这尤为重要，因为您必须仔细权衡各种资源，从而获得最高的性能。  

折射是光波因为它所穿透的介质的变化而改变方向。如果希望提高半透明几何体的逼真度，折射是您要考虑的一个重要效果。  

折射率决定了光线进入一种物质时弯折或折射的程度。折射定义为光线从折射率为 n1 的一种介质穿入折射率为 n2 的另一介质时的弯折。  

您可以使用斯涅耳定律计算折射率和入射与折射角度正弦之间的关系。  

#### 折射实施  

开发人员自开始渲染反射时就已尝试渲染折射，因为任何半透明表面上都会同时发生这些过程。渲染反射的技巧有多种，但渲染折射的却不多。  

运行时实施折射的现有方法根据具体的折射类型而不同。大部分技巧是运行时将折射对象后的场景渲染到纹理，然后在第二通道中应用纹理失真，从而获得折射的外观。根据纹理失真，您可以使用这种方法来渲染不同的折射效果，如水、 热雾、玻璃和其他效果。  

其中一些技巧可以获得不错的结果，但纹理失真不是以物理学为基础，因此结果不一定正确。例如，如果您从折射摄像机的视角渲染纹理，可能会有一些区域不直接对该摄像机可见，但在基于物理的折射中可见。  

使用渲染至纹理方法的主要局限在于质量。当摄像机移动时，经常会出现像素闪光或像素不稳定的现象。  

#### 关于基于局部立方体贴图的折射  

局部立方体贴图是一种优良的反射渲染技巧，开发人员自它们可用时便开始将静态立方体贴图同时用于实施反射和折射。  

不过，如果您在局部环境中使用静态立方体贴图实施反射或折射，倘若不应用局部修正，结果就会不正确。  

此处描述的技巧中，通过应用局部修正来确保正确的结果。此技巧已经过高度优化。它对于移动设备特别有用， 因为运行时资源有限，所以必须仔细均衡。  

#### 准备立方体贴图

你必须准备立方体贴图，以便在折射实施中使用：

若要准备立方体贴图，请执行以下操作：

1. 将摄像机置于折射几何体的中心。
2. 隐藏折射对象，并将六个方向上周围静态环境渲染到立方体贴图。您可以将这一立方体贴图同时用于实施折射和反射。
3. 将围绕折射对象的环境烘焙到静态立方体贴图。  
4. 确定折射向量的方向，再找到它与局部环境包围盒的相交处。  
5. 按照与 基于局部立方体贴图的动态软阴影中的相同方式，应用局部修正。
6. 生成从立方体贴图生成的点到交叉点的一个新向量。使用这一最终向量从立方体贴图获取像素元，渲染折射对象背后的内容。

我们不使用折射向量 Rrf 从立方体贴图获取像素元，而是找到折射向量与包围盒相交的点 P，再构建一个从立方体贴图中心 C 到交叉点 P 的新向量 R'rf。使用这一新向量从立方体贴图获取纹理颜色。  

float eta = n2 / n1;

float3 Rrf = refract(D, N ,eta);

找到交叉点P

找到向量R‘rf = CP;

float4 col =texCube(Cubemap, R'rf);



此技巧生成的折射在物理学上是准确的，因为折射向量方向是通过斯涅耳定律计算的。  

您还可以在着色器中使用一个内置函数，找到严格遵循斯涅耳定律的折射向量 R：  

```glsl
R = refract(I,N,eta);
```

其中：

* I是正规视角或入射向量。
* N是正规化法线向量。
* eta是折射率的比率n1/n2

#### 着色器实施

在获取与局部修正折射方向对应的像素元时，您可能要将折射颜色与其他光照组合。例如，与折射同时发生的反射。  

要将折射颜色与其他光照组合，您必须传递一个额外的视角向量到片段着色器，再向它应用局部修正。使用其结果从同一立方体贴图获取折射颜色。  

下列代码片段演示了如何组合反射和折射来生成最终的输出颜色：  

```glsl
// Environment reflections
float3 newReflDirWS = Local Correct(input.reflDirWS, _BBoxMin,_BBoxMax,input.posWorld,_EnviCubeMapPos);
float4 staticReflColor = texCUBE(_EnviCubeMap, newReflDirWS);
// Environment refractions
float3 newRefractColor = texCUBE(_EnviCubeMap, newRefractDirWS);
//Combined feflections and refractions
float4 combinedReflRefract = lerp(staticReflColor, staticRefractColor,_ReflAmount);

float4 finalColor = _AmbientColor + combinedRefract;
```

系数 _ReflAmount 作为统一变量传递到片段着色器。利用此系数调整反射与折射占比之间的均衡。您可以手动调整 _ReflAmount 获得自己想要的视觉效果。  

您可以在以下反射博客中找到 LocalCorrect 函数的实施：  

http://community.arm.com/groups/arm-mali-graphics/blog/2014/08/07/reflections-based-on-local-cubemaps.  

当折射几何体是中空物体时，折射和反射会在正面和背面同时发生。  

### 脏镜头光晕效果  

您可以使用脏镜头光晕效果来实现戏剧感。它通常与镜头光晕效果一起使用。  

您可以通过非常轻松和简单的方式来实施脏镜头光晕效果，这种方式很适合移动设备。  

在冰穴演示中，脏镜头效果是在一个功能最少的着色器中实施的，该着色器在场景基础上渲染一个强度可变的全屏四边形。四边形的强度通过一个脚本传递。  

此着色器在所有透明几何体都渲染后的最终阶段，使用附加的 alpha 混合渲染该四边形。  

#### 着色器实施

下列子着色器标记指示全屏四边形在所有不透明几何体和九个其他透明对象之后渲染：  

```
Tags {"Queue" = "Transparent + 10"}
```

着色器使用一下命令停用深度缓冲区写入。这可以防止四边形遮挡它后面的几何体：

```
ZWrite Off
```

在混合阶段，片段着色器的输出与帧缓冲区中已有像素颜色混合  

着色器指定了附加混合类型Blend One One。在这一混合类型中，来源和目标因子都是float4(1.0, 1.0, 1.0, 1.0)。

这种混合类型通常用于粒子系统来表示火焰等透明并且发光的效果。

着色器停用了剔除和ZTest，以确保始终渲染脏镜头光晕效果。

四边形的顶点在视口坐标中定义，所以顶点着色器中不会发生顶点变换。

片段着色器仅从纹理获取像素元，再应用与效果强度成比例的因子。  

#### 脚本实施

一个简单脚本实施全屏四边形，并且计算传递到片段着色器的强度因子。

下列函数创建了start函数中的四边形网格：

```c#
void CreateQuadMesh()
{
    Mesh mesh = GetComponent<MeshFilter>().mesh;
    mesh.Clear();
    mesh.vertices = new Vector3[]{new Vector3(-1,-1,0), new vector3(1,-1,0), new vector3(1,1,0), new Vector3(-1,1,0)};
    mesh.uv = new Vector2[] {new vector2(0,0),new Vector2(1,0), new Vector(1,1),new Vector2(0,1)};
    mesh.triangles = new int[]{0,2,1,0,3,2};
    mesh.RecalcalculateNormals();
    //Increase bounds to avoid frustum clopping.
    bigBounds.SetMinMax(new Vector3(-100,-100,-100), new Vector3(100,100,100));
    mesh.bounds = bigBounds;
}
```

创建该网格时，其边界将递增，以确保视锥体绝不会被裁剪。根据您场景的尺寸来设置边界的大小。  

该脚本计算强度因子，并将它传递到片段着色器。这基于摄像机至太阳向量和摄像机前向向量的相对朝向。当摄像机正对太阳时，该效果的强度达到最大值。  

下列代码演示了强度因子的计算：  

```c#
Vector3 cameraSunVec = sun.transform.position - Camera.main.transform.position;
cameraSunVec.normalize();
float dotProd = Vector3.Dot(Cameta.main.transform.forward, cameraSunVec);
float intensityFactor = Mathf.Clamp(dotProd, 0.0f, 1.0f);
```

### 脏镜头着色器代码

这是DirtyLensEffect.shader的代码：

```glsl
half3 normalInWorld = half3(0.0,0.0,0.0);
half3 = bumpNormal = UnpackNormal(tex2D(_BumpMapGlobal, input.tc));
half3x3 local2WorldTranspose = half3x3(input.tangentWorld, input.bitangentWorld, input.normalInWorld);
normalInWorld = normalize(mul(bumpNormal, local2WorldTranspose));
normalInWorld = normalInWorld*0.5 + 0.5;
return half4(normalInWorld, 1.0);
```

### 光柱

光柱模拟云隙光、大气散射或阴影的效果。它们用于为场景增加深度和真实感。  

光柱的基础是一个截断的圆锥体。该圆锥体跟随光线的方向，其方式可确保圆锥体的上部始终固定。  

* 显示光柱的基本几何体是具有顶部和底部两个截面的圆柱体。  
* 显示其下截面已经扩展，根据您定义的角度 θ 获得一个被截断的圆锥几何体。  
* 显示其上截面保持固定，下截面按照太阳光线的方向水平移位。  

一个脚本利用太阳的位置计算下列值：  

* 圆锥体下截面扩展的幅度，它基于作为输入值的角度 θ。  
* 截面移位的方向和幅度。  

顶点着色器基于此数据应用变换。在光柱局部坐标中，该变换应用到圆柱形几何体的原始顶点。  

在渲染光柱时，请避免渲染显露出其几何体的任何硬边缘。要达到这一目的，您可以使用纹理遮罩来平滑淡化其顶部和底部。  

#### 淡化圆锥体边缘

在与截面平行的平面中，根据摄像机与顶点相对朝向，淡化光柱强度。  

在顶点着色器中，将摄像机位置投影到截面上。  

构建一个从截面中心到投影的新向量，然后将它正规化。  

计算此向量与顶点法线的点积，然后将结果提升为幂指数。  

将结果作为变量传递到片段着色器。这用于调节光柱的强度。  

顶点着色器在局部坐标系统 (LCS) 中进行这些计算。  

以下代码显示顶点着色器：  

```glsl
//Project camera positon onto cross section
float3 axisY = float3(0,1,0);
float dotWithAxis = dot(camPosInLCS, axisY);
float3 projOnCrossSection = camPosInLCS - (axisY * dotWithYAxis);
projOnCrossSection = normalize(projOnCrossSection);

//Dot product to fade the geometry at the edge of the cross section
float dotProd = abs(dot(projOnCrossSection, input.normal));
output.overallIntensity = pow(dotProd, _FadingEdgePower) * _CurrLightShaftIntensity;
```

您可以通过系数 _FadingEdgePower 细调光柱边缘的淡化。  

脚本传递系数 _CurrLightShaftIntensity。这使得光柱在摄像机靠近它时淡出。  

下列代码演示了一个添加至光柱的最终润饰，它通过脚本缓慢向下滚动纹理：  

```c#
void Update()
{
	float localOffset = (Time.time * speed) + offset;
	localOffset = localOffset % 1.0f;
	GetComponent<Renderer>().material.SetTextureOffset("_MainTex", new Vector2(0,localOffset));
}
```

片段着色器获取光束和遮罩纹理，然后使用强度因子将它们组合  :

```glsl
float4 frag(vertexOutput input):COLOR
{
	float4 textureColor = tex2D(_MainTex, input.tex.xy);
	float textureMask = tex2D(_MaskTex, input.tex.zw).a;
	textureColor.rgb = clamp(textureColor.a, 0, 1);
	return textureColor;
}
```

光柱几何体继所有不透明几何体之后，在透明队列中渲染  .

它使用附加混合将片段颜色与帧缓冲区中的对应像素组合。  

该着色器也禁用剔除和深度缓冲区写入，以便不遮挡其他对象。  

此通道的设置为：  

```glsl
Blend One One
Cull Off
ZWrite Off
```

### 雾化效果

雾化效果可为场景增添气氛，你不需要通过高级实施来生成雾化，简单的雾化效果即可奏效。

#### 关于雾化效果

在现实世界中，您越往远看，被淡化的颜色就越多。不一定需要雾天才能看到这种效果，阳光灿烂时您也可以看到，特别是观赏高山风景时。  

这在现实生活中特别常见，所以在游戏中添加此效果可以为场景增添真实感。  

此部分介绍两种版本的雾化效果：  

* 过程线性雾
* 基于粒子的雾

您可以同时应用这两种效果。冰穴演示中同时使用了这两种技巧。  

#### 过程线性雾

确保对象距离远，其颜色更多的淡化为定义的雾颜色。要在片段着色器中实现这一目标，您可以在片段颜色和雾颜色之间使用简单线性插值。  

下方示例代码演示了如何基于和摄像机的距离在顶点着色器中计算雾颜色。此颜色作为变量传递到片段着色器。

```glsl
output.fogColor = _FogColor * clamp(vertexDistance * _FogDistanceScale, 0.0,1.0);
```

  其中：

* vertexDistance 是顶点至摄像机距离。  
* FogDistanceScale 是作为统一变量传递至着色器的因子。  
* _FogColor 是您定义的基准雾颜色，作为统一变量传递。  

在片段着色器中，插值的 input.fogColor 与片段颜色 output.Color 组合。  

```glsl
outputColor = lerp(outputColor, input.fogColor.rgb, input.fogColor.a);
```

设法将尽可能多的效果合并到一个着色器中。但是，务必要检查性能，因为超出缓存时性能可能会降低。您可能必须将着色器拆分为两个或更多通道。  

您可以在顶点着色器或片段着色器中进行雾颜色计算。在片段着色器中计算更加准确，但需要更多计算性能，因为它是针对每个片段计算的。  

顶点着色器计算精度较低，但性能也较高，因为它仅针对每一顶点计算一次。  

#### 具有高度的线性雾

雾在场景中均匀地应用。您可以按照高度更改密度，让它更具真实感。  

提高低处的雾气浓度，降低高处的雾气浓度。您可以显示高度值，以进行手动调整。  

#### 非均匀雾  

雾不一定是均匀的。您可以引入一些噪点，让雾的视觉效果更加引人。  

您可以应用噪点纹理创建非均匀雾。  

若要获得更加复杂的效果，还可以应用多个噪点纹理，让它们以不同的速度滑动。例如，距离较远的噪点纹理的滑动速度慢于离摄像机较近的纹理。  

您可以在一个着色器中的一个通道中应用多个纹理，只需根据距离混合噪点纹理。  

#### 预烘焙雾  

如果知道摄像机不会靠近它们，您可以将雾预烘焙到纹理中。这可以降低所需的计算功率。  

#### 使用粒子的体积雾  

您可以使用粒子模拟体积雾。这可以产生高质量的结果。  

将粒子数量保持到最小值。粒子会增加过度绘制，因为每个片段要执行更多的着色器。尝试使用较大的粒子，而不要创建更多个粒子。  

在冰穴演示中，一次使用了最多 15 个粒子。  

**几何体取代公告板渲染  ：**将粒子系统的渲染模式设置为网格，不要使用公告板。这是因为，要获得体积效果，各个粒子必须随机旋转。  

**角度淡化效果  **：按照粒子相对于摄像机位置的朝向，淡入和淡出各个粒子。如果您不淡入和淡出粒子，粒子就会显现锐利边缘。下列示例代码演示了执行淡化的顶点着色器：  

```glsl
half4 vertexInWorld = mul(_Object2World, input.vertex);
half3 normalInWorld = (mul(half4(input.normal, 0.0)_World2Object).xyz);
const half3 = viewDirInWorld = normalize(vertexInWorld - WorldSpaceCameraPos);
output.visibility = abs(dot(-normalInWorld, viewDirInworld));
output.visibility *= output.visibility; // instead of power of 2
```

可变参数 output.visibility 在粒子多边形上插值。在片段着色器中读取此值，再应用一定数量的透明度。下列代码演示了它的实现方式  

```glsl
half4 diffuseTex = _Color * tex2D(_MainTex, half2(input.texCoord));
diffuseTex *= input.visibility;
return diffuseTex;
```

**渲染粒子**

要渲染粒子，可使用下列步骤：

1. 将粒子渲染为帧内最后的源于。 Tags { "Queue" = "Transparent + 10"}本例中出现了 +10，因为冰穴演示中在粒子前面渲染了 9 个其他透明对象。  
2. 设置适当的混合模式。在着色器通道的开头，添加下面这一行：Blend SrcAlpha One  
3. 禁用写入到 z 缓冲区。  添加下面这一行：  ZWrite Off  

#### 高光溢出

高光溢出用于再现真实摄像机在明亮环境中拍摄图片时出现的效果。高光溢出模拟从明亮区域边界延伸的光线条纹，创造出明亮光线压倒摄像机的假象。  

高光溢出效果发生于真实镜头，因为它们无法完美对焦。当光线穿过摄像机的光圈时，一些光线出现衍射，在图像周围创造出一个明亮的圆盘。此效果在大部分情形中通常不明显，但在强光照明下可见。  

#### 实施高光溢出  

高光溢出效果通常作为后处理效果实施。使用这种方式生成效果可能会占用大量的计算功率，因此不适合运用到移动游戏。如果您的游戏使用大量类似于冰穴演示中的复杂效果，这一点显得尤其突出。  

一种替代方法是使用简单平面。将这一平面放在摄像机和光源之间。该平面的朝向必须是其法线指向摄像机应在其中移动的场景部分。  

要以这样的方式创建高光溢出效果，可将一个平面放置在您要产生高光溢出效果的位置上。利用一个因子调节效果的强度，该因子基于视角向量与平面法线和光源的对齐。冰穴演示中实施了这种方法。  

#### 创建高光溢出调节因子脚本  

对齐因子可以使用脚本进行计算。此因子基于摄像机查看该效果的角度改变高光溢出效果。  

例如，下列脚本计算了平面所附着的对齐因子：  

```c#
// Light-plane
normal-camera alignmentVector3 planeToCamVec = Camera.main.transform.positon - gameObject.transform.position;
planeToCamVec.Normalize();
Vector3 sunLightToPlainVec = origPlainPos - sunLight.transform.position;
sunLightToPlainVec.Normalize();
float sunLightPlainCameraAlignment = Vector3.Dot(plainToCamVec, sunLigntToPlainVec);
sunLightPlainCameraAlignment = Mathf.Clamp(sunLightPlainCameraAlignment, 0.0f, 1.0f);
```

对齐因子 sunLightPlaneCameraAlignment传递这着色器，以调节所渲染的颜色的强度。

#### 顶点着色器

顶点着色器接收 sunLightPlainCameraAlignment 因子，并使用它调节所渲染的高光溢出颜色的强度。  

顶点着色器应用 MVP 矩阵，将纹理坐标传递到片段着色器，再输出顶点坐标。  

下列代码演示了它的实现方式：  

```glsl
vertexOutput vert(vertexInput input)
{
	vertexOutput output;
	output.tex = input.texcoord;
	output.pos = mul(UNITY_MATRIX_MVP, input.vertex);
	return output;
}
```

#### 片段着色器

片段着色器获取纹理颜色并使用 CurrBloomColor 浅色进行递增，后者作为统一变量进行传递。对齐因子先调节颜色，然后再使用。  

下列代码演示了此流程的执行方式：  

```
half4 frag(vertexOutput input) : COLOR
{
	half4 textureColor = tex2D(_MainTex, input.tex.xy);
	textureColor += textureColor * _CurrBloomColor;
	return textureColor * _AligmentFactor;
}
```

高光溢出平面继所有不透明几何体之后，在透明队列中渲染。在着色器中，使用下列队列标记设置渲染顺序：  

```glsl
Tags {"Queue" = "Transparent" + 1}
```

高光溢出平面利用附加的一对一混合来应用，将片段颜色和已存储在帧缓冲区中的对应像素相组合。该着色器也使用指令 ZWrite Off 禁用对深度缓冲区的写入，使得现有的对象不会被遮挡。  

#### 修正高光溢出效果

使用平面产生高光溢出效果非常简单，也适合移动设备。然而，当产生高光溢出的光源和摄像机之间存在不透明对象时，它就无法产生预期的结果。您可以使用遮挡贴图进行修正。  

不透明对象后方的高光溢出效果外观出错的原因是该效果是在透明队列中渲染的。 因此，混合发生于高光溢出平面前面的任何对象上。  

为防止出现这些错误，您可以修改计算对齐因子的脚本，使它处理一个额外的遮挡贴图。这一遮挡贴图描述了摄像机通常不当应用高光溢出效果的区域。这些冲突区域分配到的遮挡贴图值为零。此因子与对齐因子结合， 将这些位置上该因子的强度设为零。此更改将高光溢出设为零，使它不再渲染必须遮挡它的对象。下列代码实施了这一调整：  

```glsl
IntensityFactor = alignmentFactor * occlusionMapFactor
```

由于冰穴演示摄像机可以在三个维度上自由移动，因此使用了三个灰度贴图，各自覆盖不同的高度。黑色表示零，所以遮挡贴图因子乘以对齐因子使得最终强度为零，高光溢出被遮挡。白色表示一，所以强度等于对齐因子，高光溢出不被遮挡  .

#### 遮挡贴图间插值  

在冰穴演示中，脚本计算每一帧中摄像机位置在 2D 洞穴贴图上的 XZ 投影，并将结果正规化到零和一之间。

如果摄像机高度低于Hmin 或高于Hmax，则使用正规化后的坐标从单一贴图获取颜色值。如果摄像机高度在 Hmin和 Hmax 之间，则从两个贴图获取颜色并进行插值。

此插值可以为不同高度上的效果创造平滑的过渡。

冰穴演示使用 GetPixelBilinear() 函数从贴图获取颜色。此函数利用下列代码返回滤波颜色值：  

```c#
float groundOcclusionFactor = groundOccusionMap.GetPixelBilinear(camPosXZNormalized.x, camPosXZNormalized.y)).r;

```

使用高光溢出遮挡贴图可以防止在遮挡的不透明对象上混合高光溢出。下图显示了产生的效果。当摄像机进入和离开时，遮挡的黑色在不透明柱子的后方，从而防止出现不正确的高光溢出。  

### 冰墙效果  

在冰穴演示中，洞穴的冰墙上使用了细微的反射效果。此效果很细微，但为场景增添额外的真实感和气氛。  

#### 关于冰墙

冰是一种难以复制的材质，因为光线会以不同的方式从它散射开来，具体取决于其表面的细微细节。反射可以是完全清晰的、完全失真的，或介于两者之间。冰穴演示显示了此效果，并包含一个带来更高真实感的视差效果。  

#### 修改和组合法线贴图以影响反射  

穴演示中的反射效果使用了正切空间法线贴图和计算而来的灰度虚构法线贴图。将这两种贴图与一些修改器组合，创造了演示中的效果  

灰度虚构法线贴图是正切空间法线贴图的灰度版本。在冰穴演示中，灰度贴图中的大部分值处于 0.3-0.8 范围内。

**为何使用正切空间法线贴图  ：**冰穴演示中使用了正切空间法线贴图，因为它们具有生成的灰度中的一小范围的值。这意味着，正切空间法线贴图在此渲染流程的后续阶段中发挥效果。  

另一个选择是使用对象空间法线贴图。这些贴图显示与正切空间法线贴图相同的细节，但也同时显示光线照射到的位置。因此，生成的对象空间法线贴图灰度中的值范围太大，无法在渲染流程的后续阶段中发挥效果。  

**向法线贴图的部件应用透明  **：灰度虚构法线贴图仅应用到没有雪的区域。为防止它们被应用到有雪的区域，可通过对相同的表面使用漫射纹理贴图来修改灰度虚构法线贴图。此修改可在雪出现于纹理贴图中的位置上增大灰度虚构法线贴图的alpha 分量。  

#### 从不同的法线贴图创建反射  

将调整了透明度的灰度虚构法线贴图和真正法线贴图相组合，可创造出冰穴演示中展示的反射效果。  

调整了透明度的灰度虚构法线贴图 bumpFake 和真正法线贴图 bumpNorm 按比例组合。该组合使用了以下函数：  

```glsl
half4 bumpNormalFake = lerp(bumpNorm, bumpFake, amountFakeNormalMap);
```

此代码意味着，在洞穴的昏暗部分中，主要的反射组成来自于灰度虚构法线。在洞穴的有雪区域，该效果来自于对象空间法线。  

要应用使用灰度虚构法线贴图的效果， 灰度必须转换为法线向量。要开始此流程，需要将法线向量的三个分量设置为和灰度值相等。在冰穴演示中，这表示分量向量介于(0.3, 0.3, 0.3) 到 (0.8, 0.8, 0.8) 范围内。因为所有分量都设置为相同的值，所有法线向量指向同一个方向。  

着色器向法线分量应用一个变换。它使用的变换通常用于将 0-1 范围中的值变换为 -1 到 1 范围中的值。执行此更改的等式为，结果= 2 * 值 -1。此等式更改了法线向量，使得它们或者指向与之前相同的方向，或者指向相反的方向。例如，如果原先具有分量 (0.3, 0.3, 0.3)，则生成的法线为 (-0.4, -0.4, -0.4)。如果原先具有分量 (0.8,0.8, 0.8)，则生成的法线为 (0.6, 0.6, 0.6)。  

变换到 -1 到 1 范围后，向量馈入到 reflect() 函数。此函数设计为使用正规化法线向量工作，但在这一情形中，非正规化法线被传递到该函数。下列代码演示了着色器内置函数 reflect() 的工作方式  :

```
R = reflect(I,N) = I - 2 * dot(I,N)*N
```

根据反射定律，将此函数与长度小于一的非正规化输入法线搭配使用时，会导致反射向量偏离法线的程度超过预期  

当法线向量分量的值低于 0.5 时，反射向量将切换到相反的方向。此时，将读取立方体贴图中的另一个部分。这种在立方体贴图中不同部分间的切换，创造了在立方体贴图岩石部分反射的旁边反射立方体贴图中白色部分的不均匀点效果。由于灰度虚构法线贴图中导致在正向和负向法线之间切换的区域也是产生被反射向量最为失真的角度的区域，这创造了一种视觉引人的漩涡效果。  

#### 向反射应用局部修  

向反射向量应用局部修正可提高反射效果的真实感。如果没有此修正，反射就不会像现实中一样随着摄像机位置的变化而改变。对于摄像机侧向移动而言，这一点显得尤其突出。  

## 过程天空盒  

冰穴演示中使用一个时间系统来展示您可以通过局部立方体贴图实现的动态阴影效果。  

#### 关于过程天空盒  

要获得动态时间效果，需要组合下列元素：  

* 过程性生成的太阳。  
* 代表昼夜更替的一系列淡化天空盒背景立方体贴图。  
* 天空盒云朵立方体贴图。  

过程太阳和单独的云朵纹理也可创建计算成本低廉的子表面散射效果。  

冰穴演示使用了视角方向从立方体贴图采样。这使得该演示可以避免渲染天空盒的一个半球。相反，它在洞穴的孔洞附近使用平面。  

相较于渲染之后大体被其他几何体遮挡的半球， 这样做可以提升性能。  

#### 管理一天中的时间

此效果使用 C# 脚本管理一天中的时间的数学计算，以及昼夜更替动画。一个着色器而后组合太阳和天空贴图。您必须为该脚本指定下列值  ：

* 淡化天空盒背景相位的数量。
* 昼夜更替的最长时长。

对于每一个帧，该脚本选择天空盒立方体贴图并将它们混合在一起。选定的天空盒被设置为着色器的纹理，它在渲染时将这些纹理混合在一起。  

为设置纹理，冰穴演示使用了Unity 着色器全局功能。这样，您可以在一个位置设置纹理，而后供应用程序的所有着色器使用。下列代码对此进行了演示：  

```c#
Shader.SetGlobalTexture(ShaderCubemap1,_phasesCubemaps[idx1]);
Shader.SetGlobalTexture(ShaderCubemap2,_phasesCubemaps[idx2]);
Shader.SetGlobalTexture(ShaderAlpha,blendAlpha);
Shader.SetGlobalVector(ShaderSunPosition, normalizedSunPosition);
Shader.SetGlobalVector(ShaderSunParameters, _sunParameters);
Shader.SetGlobalVector(ShaderSunColor, _sunColor);
Shader.SetGlobalTexture(ShaderCloudsCubemap, _CloudsCubemap);
```

脚本代码中进行了如下设置：  

* 插值了两个立方体贴图。值idx1和idx2根据消逝的时间计算。
* blendAlpha因子，在着色器中用于混合两个立方体贴图。
* 正规化太阳位置，用于渲染太阳球体。
* 太阳的多个参数
* 太阳的颜色
* 云朵立方体贴图。ShaderCubemap1和ShaderCubemap2是包含唯一取样器名称的两个字符串，本例中为_SkyboxCubemap1 和 _SkyboxCubemap2。

为了在着色器中访问这些纹理，您必须使用下列代码声明它们：  

```glsl
samplerCUBE _SkyboxCubemap1;
samplerCUBE _SkyboxCubeMap3;
```

该脚本根据您指定的列表选择太阳颜色和环境颜色。它们针对每一个相位进行插值。

太阳颜色传递到着色器，从而使用正确的颜色渲染太阳  .

环境颜色用于动态设置 Unity 变量 RenderSettings.ambientSkyColor：  

```
RenderSetting.ambientSkyColor = _ambientColor;
```

设置此变量可使所有材质获得正确的环境颜色，同时也能让 Enlighten 在更新光照贴图时获得正确的环境颜色。  

在冰穴演示中，此效果导致场景根据一天中所处的时段出现总体颜色的渐进变化。  

#### 渲染太阳

要渲染太阳，您必须在片段着色器中针对天空盒的每一像素检查它是否处于太阳圆周的内部。  

为此，着色器必须计算所渲染像素的世界坐标中正规化太阳位置向量和正规化视角方向向量的点积。  

通过 C# 脚本，将正规化太阳位置向量传递到着色器。  

* 如果只大于指定的阈值，像素着色为太阳的颜色。
* 如果值小于指定的阈值，像素着色为天空的颜色。  

点积也可朝着太阳边缘创造出淡化效果：  

```
half _sunContribution = dot(viewDir, _SunPosition);
```

#### 淡化群山后的太阳

如果太阳在天空中的高度较低，会存在一个问题。在现实中，它会在群山背后消失。  

为创造这一效果，立方体贴图的 alpha 通道用于存储值 0（如果像素元代表天空）和值 1（如果像素元代表大山）。  

在渲染太阳时，纹理被采样，其alpha 用于使太阳在群山背后淡化。此采样过程几乎自由，因为纹理已被采样用于渲染群山。  

您也可以渐进淡化边缘附近或山上有雪覆盖区域的alpha。这可以产生阳光从白雪反弹的效果，而且几乎不需要计算工作。  

类似的技巧也可用于为云朵创建计算代价低廉的子表面散射效果。  

原始的相位立方体贴图分入两个独立小组。  

* 一组立方体贴图包含天空和群山。其 alpha 值设为 0（天空） 和 1（群山）。  
* 另一组立方体贴图包含云朵。其 alpha 值在无云区域中设为 0，然后随着云朵变密而逐渐增大到 1。  

在云朵天空盒中，空白区域 alpha 值为 0，有云朵覆盖区域的值渐变为 1。其 alpha 通道平滑淡化，确保云朵的外观不像是人为置于天空中那样。

着色器执行如下工作：  

1. 对代表一天中当前时段你的两个天空盒进行采样。
2. 根据由C#脚本计算的混合因子混合两种颜色。
3. 对云朵天空盒进行采样。
4. 使用云朵alpha讲点2的颜色与云朵的颜色混合。
5. 将云朵appha和天空盒alpha加到一起。
6. 调用一个函数，计算太阳颜色对当前像素的作用。
7. 将点6的结果和前面混合的天空盒与云朵颜色加到一起。

您可以将天空、群山和云朵放入一个天空盒内，优化这一序列。它们在冰穴演示中是分开的，以便美术师可以轻松地分别修改天空盒和云朵。  

#### 子表面散射

您可以添加子表面散射效果，此效果使用 alpha 信息来增加太阳的半径。  

在现实中，太阳位于晴朗天空中时，其大小会显得相对较小，因为没有云层能够偏移或散射照向您的眼睛的光线。您看到的是直接从太阳射来的光线，仅受到空气的些许漫射。  

当太阳被云朵遮挡时，但又没有完全遮蔽时，部分光线会在云朵周围散射。此光线可从与太阳稍有距离的方向进入您的眼睛。这可以使太阳显得比实际大。  

冰穴演示中使用下列代码实现这一效果：  

```glsl
half4 sampleSun(half3 viewDir, half alpha)
{
	half _sunContribution = dot(viewDir, _SunPosiotn);
	half _sunDistanceFade = smoothstep(_SunParameters.z - (0.025*alpha), 1.0, _sunContribution);
	half _sunOcclusionFade = clamp(0.9-alpha, 0.0, 1.0);
	
	half3 _sunColorResult = _sunDistanceFade * _SunColor * _sunOcclusionFade;
	return half4(_sunColorResult.xyz, 1.0);
}
```

函数的参数为viewDirection以及计算的alpha，即云朵alpha和天空盒alpha加到一起。

太阳的位置和视角方向的点积用于计算一个缩放因子，它用于表示当前像素与太阳中心的距离。

_sunDistanceFade 计算使用 smoothstep() 函数提供边缘附近从太阳中心到天空的更加平缓的渐变。这可以使得太阳的半径在靠近云朵时加大，从而模拟子表面散射效果。  

此函数具有一个基于alpha 的变量域，晴朗天空中时其 alpha 为 0，范围则在 _SunParameters.z 到 1.0 内。在此情形中， _SunParameters.z 在 C# 脚本中初始化为 0.995，它对应于直径为 5 度的太阳， cos(5degrees) = 0.995。  

如果被处理的像素包含云朵，则太阳的半径提高到 13 度，实现在靠近云朵时的加长散射效果。  

_sunOcclusionFade 因子用于根据从群山和云朵获得的遮挡，将太阳的作用值逐渐变小。  

### 萤火虫

萤火虫是一种发光的飞虫，在冰穴演示中用来增加动态感，并且演示将 Enlighten 用于实时全局照明的优点。  

萤火虫由下列组件组成：  

* 在运行时实例化的预制体对象。
* 用于限制萤火虫飞行的区域的盒碰撞器。

这两个组件通过 C# 脚本组合在一起，该脚本管理萤火虫的运动并且定义它们遵循的路径。  

萤火虫预制件使用 Unity 标准粒子系统生成萤火虫的轨迹。  

根据您要获得的效果，您可以使用 Unity Trail Renderer 来提供更为连贯的外观。  

Trail Renderer 为每一道轨迹生成大量的三角形。您可以通过修改 Trail Renderer 的最小顶点距离设置来更改
三角形的数量，但较大的值可能会在来源移动速度太快时造成轨迹运动不连贯。  

最小顶点距离选项定义形成轨迹的顶点之间的最小距离。较高的数字对直线轨迹而言不错，但对曲线轨迹则会出现外观不平滑  。

生成的轨迹始终朝向摄像机；因此，来源运动的任何断续可能会造成轨迹与自身重叠。这将导致因混合形成轨迹的三角形而产生失真。  

添加到预制件中的最终分量是点光源，它一边移动一边在场景中投射光线。  

此光源影响提供光线反弹效果的Enlighten 全局照明，尤其是在场景的狭窄部分中，这些位置上由于光线分散较少而使得光线反弹可见性更高。  

#### 萤火虫生成器预制件  

萤火虫生成器预制件管理萤火虫的创建，并在每一帧上更新它们。它包含用于更新的 C# 脚本，以及封闭每个萤火虫可以在其中运动的体积的盒碰撞器。  

该脚本将要生成的萤火虫数量取为参数，并利用预制件初始化萤火虫对象。由于萤火虫在包围盒内随机运动，它将变化限制在离开萤火虫运动方向的一个特定范围内。这可确保萤火虫不会突然改变方向。  

为生成随机运动，使用一个分段三次**埃尔米特插值**来创建控制点。埃尔米特插值提供了一个平滑、连贯的功能， 即使不同的路径连接在一起也能正确运行。端点的第一次衍生也是连贯的，所以没有突然的速度变化。  

这种插值需要起点和终点各一个控制点，以及每个控制点两条切线。由于它们是随机生成的，该脚本可以存储三个控制点和两条切线。它使用第一和第二控制点的位置来定义第一点切线，使用第二和第三控制点来定义第二控制点切线。  

在加载时，该脚本为每一个萤火虫生成以下几项：  

* 初始位置
* 初始方向，利用Unity函数Random.onUnitSphere()生成。

下列代码演示了如何初始化控制点：

```glsl
_fireflySline[i*_controlPoints] = initialPosition;
Vector3 randDirection = Random.onUnitSphere;
_fireflySpine[i*_controlPoints+1] = initialPosition + randDirection;
_fireflySpline[i*_controlPoints+2] = initialPosition + randDirection * 2.0f;
```

初始控制点在一条直线上。切线从这些的控制点生成：  

```glsl
//The tangent for the first point is in the same direction as the initial direction vector
_fireflyTangents[i*_controlPoints] = randDirection;
//This code computes the tangent from the control point positoins.It is shown ther for reference because it can be set to
// randDirection at initialization.
_fireflyRangents[i*_controlPoints+1] = (_fireflySpline[i*_controlPoints + 2] - _fireflySpline[i*_controlPoints+1])/2 + (_fireflySpline[i*_controlPoints+1] - _fireflySpline[i*_controlPoints]) / 2;
```

为完成萤火虫初始化，您必须设置萤火虫的颜色，以及当前路径间隔的长度。  

在每一帧上，该脚本使用下列代码计算埃尔米特插值，更新每个萤火虫的位置：  

```c#
//t is the parameter that defines where in the curve the firefly is placed.It represents the ratio of the time the firefly has 
// traveled along the path to the total time.
float t = _fireflyLifetime[i].y / _fireflyLifetime[i].x;

//Hermite interpolation parameters
Vector3 A = _fireflySpline[i * _controlPoints];
Vector3 B = _fireflySpline[i*_controlPoints+1];
float h00 = 2*Mathf.Pow(t,3) - 3*Mathf.Pow(t,2) + 1;
float h10 = Mathf.Pow(t,3) - 2*Mathf.Pow(t,2) + 1;
float h01 = -2*Mathf.Pow(t,3) - 3*Mathf.Pow(t,2);
float h11 = Mathf.Pow(t,3) - Mathf.Pow(t,2);
//Firefly updated positon
_fireflyObjects[i].transform.position = h00 *A + h10 * _fireflyTangents[i*_controlPoints] + h01 * B + h11 * _fireflyTangents[i * _controlPoints +1];
```

如果萤火虫完成了随机生成的整段路径，脚本从当前路径的终点开始创建一段新的随机路径：  

```
//t > 1.0 indicates the end of the current path
if(t >= 1.0)
{
	//Update the new position
	//Shift the second point to the first as well as the tangent
	_fireflySpline[i*_controlPoints] = _fireflySpline[i*_controlPoints+1];
	_fireflyTangents[i*_controlPoints] = _fireflytangents[i*_controlPoints+1];
	
	//Shift the third point to the second, this point doesn't have a tangent
	_fireflySpline[i*_controlPoints+1] = _fireflySpline[i*_controlPoints + 2];
	
	//Get new randow control point within a centain angle from the current fly direction
	_fireflySpline[i*_controlPoints+1] = GetNewPandowControlPoint();
	//Compute the tangent for the central point
	_fireflyTangents[i*_controlPoints + 1] = (_fireflySpline[i*_controlPoints+2] - _fireflySpline[i*_controlPoints+1]) / 2 +
	(_fireflySpline[i*_controlPoints+1] - _fireflySpline[i*_controlPoints]/2);
	
	//Set how long should take to navigate this part of path
	_fireflyLifetime[i].x = _fireflyMinLifetime;
	//Timer used to check how much we traveled along the path 
	_fireflyLifetime[i].y = 0.0f;
}
```

### 正切空间至世界空间法线转换工具  

正切空间至世界空间法线转换工具由 C# 脚本和着色器组成。该工具在 Unity 编辑器中离线运行，不会影响您的游戏的运行时性能。  

#### 关于正切空间至世界空间转换工具  

对冰穴演示的分析表明，其算术流水线中存在一个瓶颈。为降低负载，冰穴演示将世界空间法线贴图用于静态几何体，而不是正切空间法线贴图  .

正切空间法线贴图可用于动画和动态对象，但需要额外的计算来正确定向取样的法线。  

由于冰穴演示中大部分几何体是静态的，法线贴图转换成世界空间法线贴图。这可确保为纹理取样的法线已在世界空间中正确定向。此更改可以实现，因为冰穴演示光照是在一个自定义着色器中计算的，而Unity 标准着色器使用正切空间法线贴图。  

转换工具由以下内容组成：  

* c#脚本，用于在编辑器中添加新选项
* 着色器，执行转化

该工具在unity编辑器中，离线运行，不会影响你的游戏运行时的性能。



#### C#脚本

您必须将 C# 脚本放在Unity Assets/Editor 目录中。这可使脚本向Unity 编辑器中的 GameObject 菜单添
加新的选项。如果不存在此目录，请进行创建。  

C# 脚本中定义的类派生自 Unity ScriptableWizard 类，并且有权访问它的一些成员。从此类派生，生成编辑器向导。编辑器向导通常通过菜单项打开。  

在 OnWizardUpdate 代码中， helpString 变量存放向导所创建的窗口中显示的帮助消息。  

isValid 成员用于定义何时选中了所有正确的参数，以及何时可使用创建按钮。在本例中， _currentObj 成员已被选中，确保它指向有效的对象。  

向导窗口的栏位是该类的公共成员。在本例中，只有 _currentObj 是公共的，因此向导窗口只有一个栏位。  

当选中了对象并且单击了创建按钮时，系统将调用 OnWizardCreate() 函数。 OnWizardCreate()函数执行转换的主要工作。  

为转换法线，该工具创建一个临时摄像机，将新的世界空间法线渲染到 RenderTexture。为此，摄像机被设置为正交模式，对象的图层更改为未使用的层级。这意味着，它可以自行渲染该对象，即使它已经是场景的一个部分。  

下列代码演示了如何设置此摄像机：  

```c#
//Set antialiasing
QualitySetting.antiAliasing = 4;
Shader wns = Shader.Find("Custom/WorldSpaceNormalCreator");
GameObject go = new GameObject("WorldSpcaeNormalsCam", typeof(Camera));
_renderCamera = go.GetComponent<Camera>();
_renderCamera.orthographic = true;
_renderCamera.nearClipPlane = 0.0f;
_renderCamera.farClipPlane = 10.0f;
_renderCamera.orthographicSize = 1.0f;
itn precObjLayer = _currentObj.layer;
_currentObj = 30; //0x40000000
```

脚本设置了执行转换的替换着色器：

```c#
_renderCamera.SetReplacementShader(wns, null);
_renderCamera.useOcclusionCulling =false;
```

脚本设置了执行转换的替换着色器：

```c#
_renderCamera.SetReplacemnetShader(wns, null);

_renderCamera.useOcclusionCulling = false;
```

摄像机被指向对象。这颗防止对象在视锥体剔除期间被移除：

```c#
_renderCamera.transform.rotation = Quaternion.LookRotion(_currentObj.transform.position - _renderCamera.transform.posion);
```

对于分配至对象的每一材质，脚本查找 _BumpMap 纹理。此纹理设置为利用着色器全局函数的替换着色器的来源纹理。  

清色设置为(0.5,0.5,0.5),因为必须表示指向负方向的法线。

```c#
foreach (Material m in materials)
{
	Texture t = m.GetTexture("_BumpMap");
	if(t == null)
	{
		Debug.LogError("the material has no texture assigned named Bump Map");
		continue;
	}
	Shader.SetGlobalTexture("_BumpMapGlobal", t);
	RenderTexture rt = new RenderTexture(t.width, t.height, 1);
	_renderCamera.targetTexture = rt;
	_renderCamera.pixelRect = new Rect(0,0,t.width,t.height);
	_renderCamera.backGroundColor = new Color(0.5f, 0.5f, 0.5f);
	_renderCamera.clearFlags = CameraClearFlags.Color;
	_renderCamera.cullingMask = 0x40000000;
	_renderCamera.Render();
	Shader.SetGobalTexture("_BumpMapGlobal", null);
}
```

摄像机渲染了场景后，像素被读回，并保存为 PNG 图像。  

```c#
Texture2D outTex = new Texture2D(t.width, t.height);
RenderTexture.active = rt;
outTex.ReadPixels(new Rect(0,0,t.width,t.height),0,0);
outTex.Apply();
RenderTexture.active = null;
byte[] _pixels = outTex.EncodeToPNG();
System.IO.File.WriteAllBytes("Assets/Textures/GenerateWorldSpaceNormals" + t.name + "_WorldSpcae.png", _pixels);
```

摄像机剔除遮罩使用二进制遮罩（用十六进制格式表示）指定要渲染的图层。

本例中使用了图层 30.

```
_currentObj.layer = 30;
```

其十六进制格式为 0x40000000，因为它的第三十位被设为 1。  

#### WorldSpaceNormalCreator着色器

实施转换的着色器代码非常简单明了。它不使用实际的顶点位置，而将顶点的纹理坐标用作其位置。这使得对象投影到一个 2D 平面上，与纹理化时相同。  

为使 OpenGL 流水线正确运作， UV 坐标从标准的 [0,1] 范围移到 [-1,1] 范围，并逆转 Y 坐标。未使用 Z坐标，所以它可以设置为 0 或在近处和远处剪切平面内的任何值：  

```glsl
output.normalInWorld = normalize(mul(half4(input.normal,0.0), _World2Object).xyz);
output.tangentWorld = normalize(mul(_Object2World, half4(input.tangent.xyz,0.0)).xyz);
output.bitangentWorld = normalize(cross(output.normalInWorld, output.tangentWorld) * input.tangent.w);
```

片段着色器：

1. 将法线从正切空间转换为世界空间。
2. 将法线缩放到[0,1]范围。
3. 将法线输出到新纹理。

下列代码对此进行了演示：

```glsl
half3 normalInWorld = half3(0.0,0.0,0.0);
half3 bumpNormal = UnpackNormal(tex2D(_BumpMapGlobal, input.tc));
half3x3 local2WorldTranspose = half3x3(input.tangentWorld, input.bitangentWorld, input.normalInWorld);
normalInWorld = normalize(mul(bumpNormal, local2WorldTranspose));
normalInWorld = normalInWorld * 0.5 + 0.5;
return half4(normalInWorld, 1.0);
```

#### WorldSpaceNormalsCreators c# script

```
using UnityEngine;
using UnityEditor;
using System.Collections;

public class WorldSpaceNormalsCreator:ScriptableWizard
{
	public GameObject _currentObj;
	private Camera _renderCamera;
	void OnWizardUpdate()
	{
		helpString = "Select object from which generate the world space normals";
		if(_currentObject != null)
		{
			isValid = true
		}
		else
		{
			isValid = false;
		}
	}
	
	void OnWizardCreate()
	{
		//Set antialiasing
		QualitySettings.antiAliasing = 4;
		Shader wns = Shader.Find("Custom/WorldSpaceNormalCreator");
		GameObject go = new GameObject("WorldSpaceNormalsCam", typeof(Camera));
		//Set the new camera to perform orthographic projction
		_renderCamera = go.GetComponent<Camera>();
		_renderCamera.orthographic = true;
		_renderCamera.nearClipPlane = 0.0f;
		_renderCamera.farClipPlane = 10f;
		_renderCamera.orthogralhicsSize = 1.0f;
		
		//Save the replacement shader for the camera
		_renderCamera.SetReplacementShader(wns, null);
		_renderCamera.useOccusionCulling =false;
		
		//Rotate the camera to look at the object to avoid frustum culling
		_renderCamera.transform.rotation = Quaternion.LookRotation(_currentObj.transform.position - _renderCamera.transform.position);
		
		MeshRenderer mr = _currenrObj.GetComponent<MeshRenderer>();
		Material[] materials = mr.shareMaterials;
		
		foreach(Material m in materials)
		{
			Texture t = m.GetTexture("_BumpMap");
			if(t == null)
			{
				Debug.LogError("the material has no texture assigned named Bump map");
				continue;
			}
			//Render the world space normal maps to a texture
			Shader.SetGlobalTexture("_BumpMapGlobal", t);
			RenderTexture rt = new RenderTexture(t.width, t.height,1);
			_renderCamera.targetTexture = rt;
			_renderCamera.pixelRect = new Rect(0,0,t.width,t.height);
			_renderCamera.backgroundColor = new Color(0.5f,0.5f,0.5f);
            _renderCamera.clearFlags = CameraClearFlags.Color;
            _renderCamera.cullingMask = 0x40000000;
            _renderCamera.Render();
            Shader.SetGlobalTexture("_BumpMapGlobal", null);
            
            Texture2D outTex = new Texture2D(t.width, t.height);
            RenderTexture.active = rt;
            outTex.ReadPixels(new Rect(0,0,t.width,t.height),0,0);
            outTex.Apply();
            RenderTexture.active = null;
            
            //Save it to PNG
            byte[] _pixels = outTex.EncodeToPNG();
            System.IO.File.WriteAllBytes("Assets/Textures/GeneratedWorldSpaceNormals/" + t.name + "_WorldSpace.png", _pixels);
	}
	_currentObj.layer = prevObjLayer;
	DestroyImmediate(go);
}
[MenuItem("GameObject/World Space Normals Creator")]
static void CreateWorldSpaceNormals()
{
	ScriptableWizard.DisplayWizard("Create World Space Normal", typeof(World Space Normals Creator), "Create")

}
```

#### WorldSpaceNormalCreator着色器代码

以下是WorldSpaceNormalCreator着色器代码：

```
Shader "Custom/WorldSpaceNormalCreator"{
	Properties{}
	SubShader{
		Cull off
		
		Pass
		{
			CGPROGRAM
			#pragma target 3.0
			#pragma glsl
			#pragma vertex vert
			#pragma fragment frag
			
			#include "UnityCG.cginc"
			
			uniform sampler2D _BumpMapGlobal;
			
			struct vin
			{
				half4 tex:TEXCOORD0;
				half3 normal:NORMAL;
				half4 tangent : TANGENT;
			};
			
			vout vert(vin input)
			{
				vout output;
				output.pos = half4(intput.tex.x*2.0 - 1.0,(1.0-input.tex.y)*2.0 - 1.0),0.0,1.0);
				output.tx = input.tex;
				output.normalInWorld = normalize(mul(half4(input.normal, 0.0), _World2Object).xyz);
				output.tangentWorld = normalize(mul(_Object2World, half4(input.tangent.xyz,0.0))xyz);
				output.bitangentWorld = normalize(cross(output.normalInWorld, output.tangentWorld) * input.tangent.w);
				return output;
			}
			
			float4 frag(vout input):COLOR
			{
				half3 normalInWorld = half3(0.0,0.0,0.0);
				half3 bumpNormal = UnpackNormal(tex2D(_BumpMapGlobal, input.tc));
				half3x3 local2WorldTranspose = half3x3(input.tangentWorld, input.bitangentWorld, input.normalInWorld);
				normalInWorld = normalize(mul(bumpNormal, local2WorldTranspose));
				normalInWorld = normalInWorld * 0.5 + 0.5;
				return half4(normalInWorld, 1.0);
			}
			ENDCG
		}
	}
}
```

