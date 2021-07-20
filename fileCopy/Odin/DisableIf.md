# DisableIf

> *用于任何属性，并且可以在检查器中启用或禁用该属性。相关属性时，使用此选项可禁用属性。*

##### 这个特性的效果主要是当指定条件满足时，灰态对应的属性，默认传入的参数为对应属性的名称，如果为True或者不为null时，灰态对应属性

![img](https://aihailan.com/wp-content/uploads/2020/11/post-554-5fb7d7dff0460.gif)

```cs
    //默认判断bool或者是否为null 为null则是false
    [DisableIf("IsToggled")]
    public int DisableIfToggled;

    [DisableIf("SomeObject")]
    public Vector3 EnabledWhenNull;
```

##### 还以指定一个选项值，当指定的属性与这个值拼配时，显示属性

```cs
    //指定的属性的值是否与给定的值一致，如果结果为true，则灰态对应的属性
    [DisableIf("SomeEnum", InfoMessageType.Info)]
    public Vector2 Info;

    [DisableIf("SomeEnum", InfoMessageType.Error)]
    public Vector2 Error;

    [DisableIf("SomeEnum", InfoMessageType.Warning)]
    public Vector2 Warning;
```

##### 可以使用@特殊符号写入表达式，其表达式的值作为实参

```cs
    [DisableIf("@this.IsToggled && this.SomeObject != null || this.SomeEnum == InfoMessageType.Error")]
    public int DisableWithExpression;
```

##### 完整示例代码

```cs
using Sirenix.OdinInspector;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class DisableIfAttributeExample : MonoBehaviour
{
    public UnityEngine.Object SomeObject;

    [EnumToggleButtons]
    public InfoMessageType SomeEnum;

    public bool IsToggled;

    //指定的属性的值是否与给定的值一致，如果结果为true，则灰态对应的属性
    [DisableIf("SomeEnum", InfoMessageType.Info)]
    public Vector2 Info;

    [DisableIf("SomeEnum", InfoMessageType.Error)]
    public Vector2 Error;

    [DisableIf("SomeEnum", InfoMessageType.Warning)]
    public Vector2 Warning;

    //默认判断bool或者是否为null 为null则是false
    [DisableIf("IsToggled")]
    public int DisableIfToggled;

    [DisableIf("SomeObject")]
    public Vector3 EnabledWhenNull;

    [DisableIf("@this.IsToggled && this.SomeObject != null || this.SomeEnum == InfoMessageType.Error")]
    public int DisableWithExpression;
}
```