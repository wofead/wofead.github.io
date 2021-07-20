# Hide If

> *用于任何属性，并且可以在检查器中隐藏该属性。使用此选项可根据对象的当前状态隐藏不相关的属性。*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-592-5fb7d9b7c5680.gif)

##### 传一个属性的名称，此属性的值如果为true或者部位null，则隐藏此属性

```cs
    [HideIf("IsToggled")]
    public Vector3 HiddenWhenToggled;

    [HideIf("SomeObject")]
    public Vector3 ShowWhenNull;
```

##### 传入一个选项值（第二个参数），作为与第一个参数指定的属性拼配，如果一致，则隐藏属性

```cs
    [HideIf("SomeEnum", InfoMessageType.Info)]
    public Vector3 Info;
```

##### 使用@转义符传入表达式

```cs
    [HideIf("@this.IsToggled && this.SomeObject != null")]
    public int HideWithExpression;
```

##### 完整示例代码

```cs
using Sirenix.OdinInspector;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class HideIfAttributeExample : MonoBehaviour
{
    public UnityEngine.Object SomeObject;

    [EnumToggleButtons]
    public InfoMessageType SomeEnum;

    public bool IsToggled;

    [HideIf("SomeEnum", InfoMessageType.Info)]
    public Vector3 Info;

    [HideIf("IsToggled")]
    public Vector3 HiddenWhenToggled;

    [HideIf("SomeObject")]
    public Vector3 ShowWhenNull;

    [HideIf("@this.IsToggled && this.SomeObject != null")]
    public int HideWithExpression;
}
```