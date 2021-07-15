# Hide Reference Object Picker

> Hide Reference Object Picker 特性：隐藏非Unity序列化引用类型属性上方显示的多态对象选择器。

![img](https://aihailan.com/wp-content/uploads/2020/11/post-614-5fb7da686b866.png)

```cs
using Sirenix.OdinInspector;
using System.Collections.Generic;
using UnityEngine;

public class HideReferenceObjectPickerAttributeExample : MonoBehaviour
{
    [Title("Hidden Object Pickers")]
    [ShowInInspector]
    [HideReferenceObjectPicker]
    public MyCustomReferenceType OdinSerializedProperty1 = new MyCustomReferenceType();
    [ShowInInspector]
    [HideReferenceObjectPicker]
    public MyCustomReferenceType OdinSerializedProperty2 = new MyCustomReferenceType();
    [ShowInInspector]
    [PropertySpace(40)]
    [Title("Shown Object Pickers")]
    public MyCustomReferenceType OdinSerializedProperty3 = new MyCustomReferenceType();
    [ShowInInspector]
    public MyCustomReferenceType OdinSerializedProperty4 = new MyCustomReferenceType();

    // Protip: 您还可以将HideInInspector属性放在类定义本身上，以便为所有成员全局隐藏它。
    //[HideReferenceObjectPicker]
    public class MyCustomReferenceType
    {
        public int A;
        public int B;
        public int C;
    }
}
```