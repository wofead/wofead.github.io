# Show If Group

> *许根据条件显示或隐藏一组属性。该属性是组属性，因此可以与其他组属性组合，甚至可以用于显示或隐藏整个组。*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-674-5fb7dc58c5ad1.gif)

##### 有组准定有层级，先说单层级，指定的名称既是组的名称，也是对应属性的名称，如果指定的属性的值为true或者不为null，则显示对应的组

```cs
    public bool Toggle = true;

    [ShowIfGroup("Toggle")]
    [BoxGroup("Toggle/Shown Box")]
    public int A, B;
```

##### 多层级的情况下，组最后的名称为指定属性的名称

```cs
    [BoxGroup("Box")]
    [ShowIfGroup("Box/Toggle")]
    public Vector3 X, Y;
```

##### 也可以特别指定属性的名称

```cs
    // ShowIfGroup将默认使用组的名称，
    //但是您也可以使用MemberName属性来覆盖它。
    [ShowIfGroup("RectGroup", MemberName = "Toggle")]
    public Rect Rect;
```

##### 可以设置与指定属性的匹配值，如果匹配，则显示

```cs
    //与常规if属性一样，ShowIfGroup也支持指定值。
    //您还可以将多个ShowIfGroup属性链接在一起，以实现更复杂的行为。
    [ShowIfGroup("Box/Toggle/EnumField", Value = InfoMessageType.Info)]
    [BoxGroup("Box/Toggle/EnumField/Border", ShowLabel = false)]
    public string Name;
```

##### 完整示例代码

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class ShowIfGroupAttributeExample : MonoBehaviour
{
    public bool Toggle = true;

    [ShowIfGroup("Toggle")]
    [BoxGroup("Toggle/Shown Box")]
    public int A, B;

    [BoxGroup("Box")]
    public InfoMessageType EnumField = InfoMessageType.Info;

    [BoxGroup("Box")]
    [ShowIfGroup("Box/Toggle")]
    public Vector3 X, Y;

    //与常规if属性一样，ShowIfGroup也支持指定值。
    //您还可以将多个ShowIfGroup属性链接在一起，以实现更复杂的行为。
    [ShowIfGroup("Box/Toggle/EnumField", Value = InfoMessageType.Info)]
    [BoxGroup("Box/Toggle/EnumField/Border", ShowLabel = false)]
    public string Name;

    [BoxGroup("Box/Toggle/EnumField/Border")]
    public Vector3 Vector;

    // ShowIfGroup将默认使用组的名称，
    //但是您也可以使用MemberName属性来覆盖它。
    [ShowIfGroup("RectGroup", MemberName = "Toggle")]
    public Rect Rect;
}
```