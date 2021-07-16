# InlineEditorAttribute

> *InlineAttribute用于任何属性或字段，其类型继承自UnityEngine.Object。这包括组件和资产等。*

![img](../image/InLineEditorAttribute/post-624-5fb7dab2dbe8b.gif)

![img](../image/InLineEditorAttribute/post-624-5fb7dab3edd97.gif)

![img](../image/InLineEditorAttribute/post-624-5fb7dab492e53.png)

![img](../image/InLineEditorAttribute/post-624-5fb7dab5026dc.gif)

![img](../image/InLineEditorAttribute/post-624-5fb7dab5e5807.gif)

##### 【InlineEditorObjectFieldModes.Boxed】属性以Box形式展示

![img](../image/InLineEditorAttribute/post-624-5fb7dab78edcf.png)

```cs
    [Title("Boxed / Default")]
    [InlineEditor(InlineEditorObjectFieldModes.Boxed)]
    public ExampleTransform Boxed;
```

##### 【InlineEditorObjectFieldModes.Foldout】属性以折页形式展示

![img](../image/InLineEditorAttribute/post-624-5fb7dab832272.png)

```cs
    [Title("Foldout")]
    [InlineEditor(InlineEditorObjectFieldModes.Foldout)]
    public ExampleTransform Foldout;
```

##### 【InlineEditorObjectFieldModes.CompletelyHidden】隐藏属性名称

![img](../image/InLineEditorAttribute/post-624-5fb7dab8a503b.png)

```cs
    [Title("Hide ObjectField")]
    [InlineEditor(InlineEditorObjectFieldModes.CompletelyHidden)]
    public ExampleTransform CompletelyHidden;
```

##### 【InlineEditorObjectFieldModes.Hidden】只有为null的时候才显示字段

![img](../image/InLineEditorAttribute/post-624-5fb7dab92ebaa.gif)

```cs
    [Title("Show ObjectField if null")]
    [ShowInInspector]
    [InlineEditor(InlineEditorObjectFieldModes.Hidden)]
    public ExampleTransform OnlyHiddenWhenNotNull;
```



> **4预览模式**

##### 【InlineEditorModes.FullEditor】

![img](../image/InLineEditorAttribute/post-624-5fb7daba08d14.gif)

```cs
    [InlineEditor(InlineEditorModes.FullEditor)]
    public Material FullInlineEditor;
```

##### 【InlineEditorModes.GUIAndHeader】

![img](../image/InLineEditorAttribute/post-624-5fb7daba97114.png)

```cs
    [InlineEditor(InlineEditorModes.GUIAndHeader)]
    public Material InlineMaterial ;
```

##### 【InlineEditorModes.SmallPreview】

![img](../image/InLineEditorAttribute/post-624-5fb7dabb44bd2.gif)

```cs
    [InlineEditor(InlineEditorModes.SmallPreview)]
    public Material[] InlineMaterialList = new Material[]
    {
    };
```

##### 【InlineEditorModes.LargePreview】

![img](../image/InLineEditorAttribute/post-624-5fb7dabc49002.gif)

```cs
    [InlineEditor(InlineEditorModes.LargePreview)]
    public Mesh InlineMeshPreview ;
```

##### 完整示例代码

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class InlineEditorAttributeExample : MonoBehaviour
{
    [Title("Boxed / Default")]
    [InlineEditor(InlineEditorObjectFieldModes.Boxed)]
    public ExampleTransform Boxed;

    [Title("Foldout")]
    [InlineEditor(InlineEditorObjectFieldModes.Foldout)]
    public ExampleTransform Foldout;

    [Title("Hide ObjectField")]
    [InlineEditor(InlineEditorObjectFieldModes.CompletelyHidden)]
    public ExampleTransform CompletelyHidden;

    [Title("Show ObjectField if null")]
    [ShowInInspector]
    [InlineEditor(InlineEditorObjectFieldModes.Hidden)]
    public ExampleTransform OnlyHiddenWhenNotNull;

    [InlineEditor]
    public ExampleTransform InlineComponent ;

    [InlineEditor(InlineEditorModes.FullEditor)]
    public Material FullInlineEditor;

    [InlineEditor(InlineEditorModes.GUIAndHeader)]
    public Material InlineMaterial ;

    [InlineEditor(InlineEditorModes.SmallPreview)]
    public Material[] InlineMaterialList = new Material[]
    {
    };

    [InlineEditor(InlineEditorModes.LargePreview)]
    public Mesh InlineMeshPreview ;

}
```