# Unity自带特性

[toc]

## AddComponentMenu

> AddComponentMenu属性允许您将脚本放置在“组件”菜单中的任何位置，而不仅仅是“组件 - >脚本”菜单。
>  您可以使用它来更好地组织“组件”菜单，这样可以在添加脚本时改进工作流程。重要提示：您需要重新启动。

- **componentOrder** 组件菜单中的组件顺序（低于顶部）。
- **AddComponentMenu** 在“组件”菜单中添加一个项目。



```csharp
using UnityEngine;

[AddComponentMenu("Transform/Follow Transform")]
public class FollowTransform : MonoBehaviour
{
}
```

#### 效果如下

![img](https:////upload-images.jianshu.io/upload_images/7643202-c73201285b29e944.gif?imageMogr2/auto-orient/strip|imageView2/2/w/946/format/webp)

## AssenblyIsEditorAssembly

> 装配级属性。具有此属性的程序集中的任何类将被视为编辑器类。
> **构造函数**
> [AssemblyIsEditorAssembly](https://link.jianshu.com/?t=https%3A%2F%2Fdocs.unity3d.com%2FScriptReference%2FAssemblyIsEditorAssembly-ctor.html)

## ContextMenu

> 可以在Inspector的ContextMenu中增加选项。

```c#
public class TestMenu : MonoBehaviour {
    [ContextMenu ("Do Something")]
    void DoSomething () {
        Debug.Log ("Perform operation");
    }
}
```

![f:id:lvmingbei:20150406162538p:plain](http://cdn-ak.f.st-hatena.com/images/fotolife/l/lvmingbei/20150406/20150406162538.png)

## ContextMenuItemAttribute

> 可以在Inspector上面对变量追加一个右键菜单，并执行指定的函数。

```c#
public class Sample : MonoBehaviour {
    [ContextMenuItem("Reset", "ResetName")]
    public string name = "Default";
    void ResetName() {
        name = "Default";
    }
}
```

#### 效果如下

![img](https://upload-images.jianshu.io/upload_images/7643202-e92afd36d1668f69.gif?imageMogr2/auto-orient/strip|imageView2/2/w/943/format/webp)

## CreateAssetMenu

> [fileName](https://link.jianshu.com/?t=https%3A%2F%2Fdocs.unity3d.com%2FScriptReference%2FCreateAssetMenuAttribute-fileName.html)
> 新创建的此类实例使用的默认文件名。（创建文件必须以 .asset 结尾）
> [menuName](https://link.jianshu.com/?t=https%3A%2F%2Fdocs.unity3d.com%2FScriptReference%2FCreateAssetMenuAttribute-menuName.html)
> 此类型显示的名称显示在“资产/创建”菜单中。
> [order](https://link.jianshu.com/?t=https%3A%2F%2Fdocs.unity3d.com%2FScriptReference%2FCreateAssetMenuAttribute-order.html)
> 菜单项在资产/创建菜单中的位置。



> 创建菜单中的位置。



```csharp
using UnityEngine;

[CreateAssetMenu(fileName = "自定义资源.asset", menuName = "菜单/子项0")]
public class CreateAsset : ScriptableObject
{
    public string Name = "自定义资源";

    public Vector3[] Pos = new Vector3[10];
}
```

#### 效果如下

![img](https:////upload-images.jianshu.io/upload_images/7643202-f5e7278ebe4335f9.gif?imageMogr2/auto-orient/strip|imageView2/2/w/943/format/webp)

## DisallowMultipleComponent

> 防止将相同类型（或子类型）的MonoBehaviour多次添加到GameObject。

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
[DisallowMultipleComponent]
public class Disallow : MonoBehaviour
{

}
```

### 效果如下

![img](https:////upload-images.jianshu.io/upload_images/7643202-85409f78d9e21711.gif?imageMogr2/auto-orient/strip|imageView2/2/w/943/format/webp)



## ExecuteInEditMode

> 使脚本的所有实例在编辑模式下执行。

默认情况下，MonoBehaviours只能在播放模式下执行。
 通过添加此属性，MonoBehaviour的任何实例将在编辑器不处于播放模式时执行其回调函数。

这些功能不会像播放模式一样被调用。

[更新](https://link.jianshu.com?t=https%3A%2F%2Fdocs.unity3d.com%2FScriptReference%2FMonoBehaviour.Update.html)仅在场景中的某些内容更改时才调用。当游戏视图接收到一个[事件](https://link.jianshu.com?t=https%3A%2F%2Fdocs.unity3d.com%2FScriptReference%2FEvent.html)时，[OnGUI](https://link.jianshu.com?t=https%3A%2F%2Fdocs.unity3d.com%2FScriptReference%2FMonoBehaviour.OnGUI.html)被调用。[OnRenderObject](https://link.jianshu.com?t=https%3A%2F%2Fdocs.unity3d.com%2FScriptReference%2FMonoBehaviour.OnRenderObject.html)和其他渲染回调函数在场景视图或游戏视图的每次重绘时都被调用。
 另请参见：[runInEditMode](https://link.jianshu.com?t=https%3A%2F%2Fdocs.unity3d.com%2FScriptReference%2FMonoBehaviour-runInEditMode.html)。



```csharp
using UnityEngine;

[ExecuteInEditMode]
public class PrintAwake : MonoBehaviour
{
    void Awake()
    {
        Debug.Log("Editor causes this Awake");
    }

    void Update()
    {
        Debug.Log("Editor causes this Update");
    }
}
```

#### 效果如下

![img](https:////upload-images.jianshu.io/upload_images/7643202-bee5ec60ca4f5382.gif?imageMogr2/auto-orient/strip|imageView2/2/w/943/format/webp)



## GUITargetAttribute

> 控制对应的OnGUI在那个Display上显示



```csharp
using UnityEngine;
public class ExampleClass1 : MonoBehaviour
{
    // Label will appear on display 0 and 1 only
    [GUITarget(0, 1)]
    void OnGUI()
    {
        GUI.Label(new Rect(10, 10, 300, 100), "Visible on TV and Wii U GamePad only");
    }
}
```

#### 效果如下

![img](https:////upload-images.jianshu.io/upload_images/7643202-5bd5f48d748b97a8.gif?imageMogr2/auto-orient/strip|imageView2/2/w/943/format/webp)



## HelpURLAttribute

> 帮助图标对应跳转的URL（注意此类需要继承MonoBehaviour，不继承在2017测试跳转地址无效）



```csharp
using UnityEngine;
using UnityEditor;

[HelpURL("https://www.baidu.com")]
public class MyComponent:MonoBehaviour
{
}
```

## HideInInspector

> 隐藏在Inspector面板中显示的Public属性



```csharp
using UnityEngine;
using System.Collections;

public class ExampleClass : MonoBehaviour
{
    [HideInInspector]
    public int Hide = 0;

    [Header("生命 Settings")]
    public int health = 0;
    public int maxHealth = 100;
    [Header("防御 Settings")]
    public int shield = 0;
    public int maxShield = 0;
}
```

#### 效果如下

![img](https:////upload-images.jianshu.io/upload_images/7643202-3edea463d2c7b5b3.gif?imageMogr2/auto-orient/strip|imageView2/2/w/943/format/webp)

## ImageEffectAllowedInSceneView

> 具有此属性的任何图像效果可以渲染到场景视图相机中。
>  如果您希望将图像效果应用于场景视图相机，则会添加此属性。效果将应用于相同的位置，并且具有与相机效果相同的值。(5.4开始新加的特性，目前不知道怎么用)



## ImageEffectOpaque

在OnRenderImage上使用，可以让渲染顺序在非透明物体之后，透明物体之前。
例子

```
[ImageEffectOpaque]
void OnRenderImage (RenderTexture source, RenderTexture destination){
}
```

## MultilineAttribute

> 用于使字符串值的属性显示在多行文本区域中。



```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Example1 : MonoBehaviour
{
    [MultilineAttribute(10)]
    public string str = "";

}
```

#### 效果如下

![img](https:////upload-images.jianshu.io/upload_images/7643202-99b07d5601febdc7.png?imageMogr2/auto-orient/strip|imageView2/2/w/923/format/webp)



![img](https://aihailan.com/wp-content/uploads/2020/11/post-692-5fb7dce7072e0.png)

## PreferBinarySerialization

> 这对于包含大量数据的自定义资产类型很有用。始终保持存储为二进制可以提高读/写性能，并在磁盘上生成更紧凑的表示。(说白了就是一二进制形式保存提高性能)



```csharp
using UnityEngine;


[CreateAssetMenu]
[PreferBinarySerialization]
public class CustomData : ScriptableObject
{
    public float[] lotsOfFloatData = new[] { 1f, 2f, 3f };
    public byte[] lotsOfByteData = new byte[] { 4, 5, 6 };
}
```

## NotConvertedAttribute

> 在变量上使用，可以指定该变量在build的时候，不要转化为目标平台的类型。

## NotFlashValidedAttribute

> 在变量上使用，在Flash平台build的时候，对该变量不惊醒类型检查

## NotRenamedAttribute

> 禁止对变量和方法进行重命名

## RequireComponent

> 在Class上使用，添加对另一个Component的依赖。
> 当该Class被添加到一个GameObject上的时候，如果这个GameObject不含有依赖的Component，会自动添加该Component。
> 且该Componet不可被移除。

```c#
[RequireComponent(typeof(Rigidbody))]
public class TestRequireComponet : MonoBehaviour {
}
```

## SelectionBaseAttribute

> 当一个GameObject含有使用了该属性的Component的时候，在SceneView中选择该GameObject，Hierarchy上面会自动选中该GameObject的Parent。

## SerializeField

> 在变量上使用该属性，可以强制该变量进行序列化。即可以在Editor上对变量的值进行编辑，即使变量是private的也可以。
> 在UI开发中经常可见到对private的组件进行强制序列化的用法。

```c#
public class TestSerializeField : MonoBehaviour {
[SerializeField]
private string name;
[SerializeField]
private Button _button;
}	
```

## SharedBetweenAnimatorsAttribute

> 用于StateMachineBehaviour上，不同的Animator将共享这一个StateMachineBehaviour的实例，可以减少内存占用。

## SpaceAttribute

> 使用该属性可以在Inspector上增加一些空位。 Odin的 PropertySpace

```c#
public class TestSpaceAttributeByLvmingbei : MonoBehaviour {
    public int nospace1 = 0;
    public int nospace2 = 0;
    [Space(10)]
    public int space = 0;
    public int nospace3 = 0;
}
```

## TooltipAttribute

> 这个属性可以为变量上生成一条tip，当鼠标指针移动到Inspector上时候显示。

```c#
public class TestTooltipAttributeByLvmingbei : MonoBehaviour {
    [Tooltip("This year is 2015!")]
    public int year = 0;
}
```

## TextAreaAttribute

> 该属性可以把string在Inspector上的编辑区变成一个TextArea。Multiline和它类似。

```c#
public class TestTextAreaAttributeByLvmingbei : MonoBehaviour {
    [TextArea]
    public string mText;
}
```

## FormerlySerializedAsAttribute

> 该属性可以令变量以另外的名称进行序列化，并且在变量自身修改名称的时候，不会丢失之前的序列化的值。

```c#
using UnityEngine;
using UnityEngine.Serialization;
public class MyClass : MonoBehaviour {
    [FormerlySerializedAs("myValue")]
    private string m_MyValue;
    public string myValue
    {
        get { return m_MyValue; }
        set { m_MyValue = value; }
    }
}
```

## CallbackOrderAttribute

> 定义Callback的顺序

## CanEditMultipleObjects

> Editor同时编辑多个Component的功能



## CustomEditor

> 声明一个Class为自定义Editor的Class

## CustomPreviewAttribute

> 将一个class标记为指定类型的自定义预览



```c#
[CustomPreview(typeof(GameObject))] 
public class MyPreview : ObjectPreview {    
    public override bool HasPreviewGUI()    
    {        
        return true;    
    }
    public override void OnPreviewGUI(Rect r, GUIStyle background)
    {
        GUI.Label(r, target.name + " is being previewed");
    }
}
```

## CustomPropertyDrawer

> 标记自定义PropertyDrawer时候使用。
> 当自己创建一个PropertyDrawer或者DecoratorDrawer的时候，使用该属性来标记。

## DrawGizmo

> 可以在Scene视图中显示自定义的Gizmo
> 下面的例子，是在Scene视图中，当挂有MyScript的GameObject被选中，且距离相机距离超过10的时候，便显示自定义的Gizmo。
> Gizmo的图片需要放入Assets/Gizmo目录中。



```c#
using UnityEngine;
using UnityEditor;
public class MyScript : MonoBehaviour {


}
public class MyScriptGizmoDrawer {
    [DrawGizmo (GizmoType.Selected | GizmoType.Active)]
    static void DrawGizmoForMyScript (MyScript scr, GizmoType gizmoType) {
        Vector3 position = scr.transform.position;

        if(Vector3.Distance(position, Camera.current.transform.position) &gt; 10f)
            Gizmos.DrawIcon (position, "300px-Gizmo.png");
    }
}
```

## InitializeOnLoadAttribute

> 在Class上使用，可以在Unity启动的时候，运行Editor脚本。
> 需要该Class拥有静态的构造函数。
> 做一个创建一个空的gameobject的例子。

```c#
using UnityEditor;
using UnityEngine;
[InitializeOnLoad]
class MyClass
{
    static MyClass ()
    {
        EditorApplication.update += Update;
        Debug.Log("Up and running");
    }
    static void Update ()
    {
        Debug.Log("Updating");
    }
}
```

## InitializeOnLoadMethodAttribute

> 在Method上使用，是InitializeOnLoad的Method版本。
> Method必须是static的。

## MenuItem

> 在方法上使用，可以在Editor中创建一个菜单项，点击后执行该方法，可以利用该属性做很多扩展功能。 需要方法为static。

```c#
using UnityEngine;
using UnityEditor;
using System.Collections;
public class TestMenuItem : MonoBehaviour {
    [MenuItem("MyMenu/Create GameObject")]
    public static void CreateGameObject(){
        new GameObject("Jow");
    }
}
```

## PreferenceItem

> 使用该属性可以定制Unity的Preference界面。

```c#
using UnityEngine;
using UnityEditor;
using System.Collections;
public class OurPreferences {

    // Have we loaded the prefs yet

    private static bool prefsLoaded = false;
    //My Preference
    public static bool boolPreference = false;
    // Add preferences section named "My Preferences" to the Preferences Window
    [PreferenceItem("My Preference")]
    public static void PreferencesGUI(){
        // Load the preferences
        if(!prefsLoaded){
            boolPreference = EditorPrefs.GetBool("BoolPreferenceKey", false);
            prefsLoaded = true;
        }
        
        // Preferences GUI
    boolPreference = EditorGUILayout.Toggle ("Bool Preference", boolPreference);
    
    // Save the preferences
    if (GUI.changed)
        EditorPrefs.SetBool ("BoolPreferenceKey", boolPreference);
    }
}
```

## OnOpenAssetAttribute

> 在打开一个Asset后被调用。

```c#
using UnityEngine;
using UnityEditor;
using UnityEditor.Callbacks;
public class MyAssetHandler {
    [OnOpenAssetAttribute(1)]
    public static bool step1(int instanceID, int line) {
        string name = EditorUtility.InstanceIDToObject(instanceID).name;
        Debug.Log("Open Asset step: 1 ("+name+")");
        return false; // we did not handle the open
    }

    // step2 has an attribute with index 2, so will be called after step1
    [OnOpenAssetAttribute(2)]
    public static bool step2(int instanceID, int line) {
        Debug.Log("Open Asset step: 2 ("+instanceID+")");
        return false; // we did not handle the open
    }
}
```

## PostProcessBuildAttribute

> 该属性是在build完成后，被调用的callback。
> 同时具有多个的时候，可以指定先后顺序。

```c#
using UnityEngine;
using UnityEditor;
using UnityEditor.Callbacks;
public class MyBuildPostprocessor {

[PostProcessBuildAttribute(1)]

public static void OnPostprocessBuild(BuildTarget target, string pathToBuiltProject) {

Debug.Log( pathToBuiltProject );

}

}
```

## PostProcessSceneAttribute

> 使用该属性的函数，在scene被build之前，会被调用。
> 具体使用方法和PostProcessBuildAttribute类似。

## RuntimeInitializeOnLoadMethodAttribute

> 允许运行时类方法在运行时加载游戏时被初始化，而不需要用户的操作。
>  标记[RuntimeInitializeOnLoadMethod]的方法在游戏加载后被调用。这是在Awake方法被调用之后。
>  注意：[RuntimeInitializeOnLoadMethod]不保证标记的方法的执行顺序。调用的方法需静态

```c#
// Create a non-MonoBehaviour class which displays
// messages when a game is loaded.
using UnityEngine;

class MyClass
{
    [RuntimeInitializeOnLoadMethod]
    static void OnRuntimeMethodLoad()
    {
        Debug.Log("场景加载和游戏运行后");
    }

    [RuntimeInitializeOnLoadMethod]
    static void OnSecondRuntimeMethodLoad()
    {
        Debug.Log("SecondMethod场景加载和游戏运行后");
    }
}
```

## SelectionBaseAttribute

> 将此属性添加到脚本类以将其GameObject标记为用于场景视图挑选的选择基础对象。
>  在Unity Scene View中，单击选择对象时，Unity将尝试找出最适合您选择的对象。如果单击作为预制体一部分的对象，则选择预制根的根，因为预制根被视为选择库。您也可以使其他对象也被视为选择库。您需要使用SelectionBase属性创建一个脚本类，然后您需要将该脚本添加到GameObject中。

