# Serialization Debugger

> *Odin包含一个***Serialization Debugger工具***，用于调试查看对应的序列化信息.*
> *如果在Inspector中的字段等序列化信息出现丢失或者不可见等问题，可以使用Serialization Debugger进行快速查看并定位问题。*
> *调试器将显示正在序列化任何给定类型的成员，以及它们是否被Unity，Odin或两者序列化。它还提供成员序列化方式的介绍信息。*
>
> #### 打开Serialization Debugger工具有两种方式
>
> - **Tools/Odin Inspector/Serialization Debugger**
> - **右键点击已经成为组件的脚本，选择Debug Serialization**

![img](https://aihailan.com/wp-content/uploads/2020/11/post-670-5fb7dc3ea5d6a.gif)

![img](https://aihailan.com/wp-content/uploads/2020/11/post-670-5fb7dc3ef19dd.gif)

> 在Serialization Debugger面板中会查看对应的字段的序列化形式，是以Odin序列化还是Unity自带的序列化，点击对应的字段在下方也会也有相对应的提示。

![img](https://aihailan.com/wp-content/uploads/2020/11/post-670-5fb7dc3f5b45b.gif)

> 而且脚本列表和搜索框，可以快速定位需要查看的脚本序列化

![img](https://aihailan.com/wp-content/uploads/2020/11/post-670-5fb7dc3f76b2f.gif)

#### 对应示例脚本

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Sirenix.OdinInspector;
using Sirenix.Serialization;
using System;

public class SerializationDebugger_ExampleOne : MonoBehaviour
{

    public string UnityString = "Unity_菜鸟海澜";
    public List UnityStringList = new List();

    [NonSerialized][OdinSerialize]
    public string OdinStringInvalid= "错误序列化";

    public TempUnitySerializationData tempUnitySerializationData = new TempUnitySerializationData();
    public TempOdinSerializationData tempOdinSerializationData = new TempOdinSerializationData();

    public List UnityList = new List();

    public Dictionary keyValuePairs = new Dictionary();

    void Start()
    {

    }

}
[Serializable]
public class TempUnitySerializationData
{
    public string UnityString = "菜鸟海澜";

}

public class TempOdinSerializationData
{
    public string UnityString = "菜鸟海澜";
}
```

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Sirenix.OdinInspector;
using Sirenix.Serialization;
using System;

public class SerializationDebugger_ExampleTwo : SerializedMonoBehaviour
{
    public string UnityString = "Unity_菜鸟海澜";

    [OdinSerialize]
    public string OdinAndUnityString = "OdinAndUnity_菜鸟海澜";
    [OdinSerialize][NonSerialized]
    public string OdinString = "Odin_菜鸟海澜";

    public List OdinList = new List();

    [SerializeField]
    public TempUnitySerializationData tempUnitySerializationData = new TempUnitySerializationData();

    public TempOdinSerializationData tempOdinSerializationData = new TempOdinSerializationData();

    public Dictionary keyValuePairs = new Dictionary();
    void Start()
    {

    }
}

```

