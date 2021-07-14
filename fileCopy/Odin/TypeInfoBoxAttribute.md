# TypeInfoBoxAttribute

> *TypeInfoBox特性：将信息框添加到Inspector中类型的最顶部。*
> *使用此选项可将信息框添加到Inspector中类的顶部，而无需同时使用PropertyOrder和OnInspectorGUI属性。*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-706-5fb7dd4a22504.gif)

##### 完整示例代码

###### TypeInfoBoxExample

```cs
using Sirenix.OdinInspector;
using System;
using UnityEngine;

public class TypeInfoBoxExample : MonoBehaviour
{
    public MyType MyObject = new MyType();

    [InfoBox("双击此此段的value值，可在inspecter中查看对应ScriptableObject信息")]
    public MyScripty Scripty = null;
    public void Awake()
    {
        Scripty = ExampleHelper.GetScriptableObject();
    }

    [Serializable]
    [TypeInfoBox("TypeInfoBox特性可以放在类型定义上，并将导致在属性的顶端处绘制一个InfoBox。")]
    public class MyType
    {
        public int Value;
    }
}
```

##### MyScripty

```cs
using Sirenix.OdinInspector;
using UnityEngine;

[CreateAssetMenu(fileName = "MyScripty_ScriptableObject", menuName = "CreatScriptableObject/MyScripty", order = 100)]
[TypeInfoBox("TypeInfoBox 特性 能以文本的形式显示在顶端 。例如, MonoBehaviours or ScriptableObjects.")]
public class MyScripty : ScriptableObject
{
    public string MyText = ExampleHelper.GetString();
    [TextArea(10, 15)]
    public string Box;
}
```