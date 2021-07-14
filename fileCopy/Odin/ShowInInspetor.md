# ShowInInspector Attribute

> *Show In Inspector Attribute特性：用于任何成员，并在inspector中显示该值。请记住，ShowInInspector特性不会序列化任何内容; 这意味着您所做的任何更改都不会仅仅使用ShowInInspector属性进行保存。*
> *如果需要序列化，需要配合SerializeField特性使用*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-678-5fb7dc73a7e8d.gif)

##### 完整示例代码

```cs
using Sirenix.OdinInspector;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ShowInInspectorExample : MonoBehaviour
{
    [ShowInInspector]
    private int myPrivateInt;

    [ShowInInspector]
    public int MyPropertyInt { get; set; }

    [ShowInInspector]
    public int ReadOnlyProperty
    {
        get { return this.myPrivateInt; }
    }

    [ShowInInspector]
    public static bool StaticProperty { get; set; }

    private void Start()
    {
        Debug.Log($"{MyPropertyInt}");
    }
}
```