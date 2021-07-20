# Show If

> *用于任何属性，并且可以在检查器中隐藏该属性。使用此选项可根据对象的当前状态隐藏不相关的属性。*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-672-5fb7dc4bb5db4.gif)

##### 这个特性的效果主要是当指定条件满足时，显示对应的属性，默认传入的参数为对应属性的名称，如果为True或者不为null时，显示属性

```cs
    [ShowIf("IsToggled")]
    public Vector2 VisibleWhenToggled;
```

##### 还以指定一个选项值，当指定的属性与这个值拼配时，显示属性

```cs
    [ShowIf("SomeEnum", InfoMessageType.Info)]
    public Vector3 Info;

    [ShowIf("SomeEnum", InfoMessageType.Warning)]
    public Vector2 Warning;

    [ShowIf("SomeEnum", InfoMessageType.Error)]
    public Vector3 Error;
```

##### 可以使用@特殊符号写入表达式，其表达式的值作为实参

```cs
    [ShowIf("@this.IsToggled && this.SomeObject != null || this.SomeEnum == InfoMessageType.Error")]
    public Vector3 HideWhenNull;
```

##### 完整示例代码

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class ShowIfAttributeExample : MonoBehaviour
{
    public UnityEngine.Object SomeObject;

    [EnumToggleButtons]
    public InfoMessageType SomeEnum;

    public bool IsToggled;

    [ShowIf("SomeEnum", InfoMessageType.Info)]
    public Vector3 Info;

    [ShowIf("SomeEnum", InfoMessageType.Warning)]
    public Vector2 Warning;

    [ShowIf("SomeEnum", InfoMessageType.Error)]
    public Vector3 Error;

    [ShowIf("IsToggled")]
    public Vector2 VisibleWhenToggled;

    [ShowIf("@this.IsToggled && this.SomeObject != null || this.SomeEnum == InfoMessageType.Error")]
    public Vector3 HideWhenNull;
}
```