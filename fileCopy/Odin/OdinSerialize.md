# Odin Serialize

> *在Unity开发中，一些挂在物体上的脚本公开的成员变量，可通过inspector面板更改对应的值，但并不是所有公开成员的值，都可以通过inspector面板进行填写，而且有些数值即使填写，也保存不了（无法序列化），例如：Dictionary<Tkey,TValue>。但是Odin为我们提供了一套解决这种情况的序列化方案。*

*如下图所示，keyValuePairs_0成员和keyValuePairs_1成员都无法进行有效的序列化，虽然keyValuePairs_1通过特性***ShowInInspector***强制显示出对应的Dictionary，也可以填写，但是在运行时，填写的数据会丢失。*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-496-5fb7cd1369f20.gif)

```cs
public class ExampleUnitySerializedScript : MonoBehaviour
{
    public Dictionary keyValuePairs_0 = new Dictionary();

    [ShowInInspector]
    public Dictionary keyValuePairs_1 = new Dictionary();
}
```

## 使用Odin序列化

> *使用比较简单，默认情况下我们继承的是* `MonoBehaviour`*,现在我们只需要用*`SerializedMonoBehaviour`*代替*`MonoBehaviour`*即可，即开即用，非常简单。*

```cs
public class ExampleUnitySerializedScript : SerializedMonoBehaviour
{
    public Dictionary keyValuePairs_0 = new Dictionary();

    [ShowInInspector]
    public Dictionary keyValuePairs_1 = new Dictionary();
}
```

> 对应`SerializedMonoBehaviour`这类Odin序列化类，总共有以下7种，足以满足日常开发中的绝大多数需求

- **SerializedMonoBehaviour**
- **SerializedBehaviour**
- **SerializedComponent**
- **SerializedNetworkBehaviour**
- **SerializedScriptableObject**
- **SerializedStateMachineBehaviour**
- **SerializedUnityObject**

#### Odin序列化是不是强制替换了原有的Unity序列化？有什么需要注意的地方呢？

1. *Odin序列化默认情况是不会替换Unity原有序列化的，Odin序列化仅仅是基于Unity序列化的扩展，对成员进行序列化时，如果Unity支持，就使用Unity序列化，如果不支持，转由Odin序列化接手。*
2. *注意事项是有的，就像刚刚的回答，默认的情况下Odin不会替换Unity原有支持的序列化情况，但是我们可以强制使用Odin序列化，或者强制让Unity序列化和Odin序列化并存，当然这种并存也会出现2份数据，所以不建议这么做。如果对这段解答不是很理解，请看下面的示例*



#### 示例到代码

```cs
using Sirenix.OdinInspector;
using Sirenix.Serialization;
using System;
using System.Collections.Generic;
using UnityEngine;

public class ExampleOdinSerializedScript : SerializedMonoBehaviour
{
    // 使用Odin序列化，而非Unity序列化
    public Dictionary firstDictionary= new Dictionary();

    // MyClassByUnity 因为标记为 Serializable ,所以使用Unity 自带的序列化，而非Odin 序列化
    public MyClassByUnity myUnityReference = new MyClassByUnity();

    //强制使用 Odin 序列化，而不使用Unity的序列化
    [NonSerialized, OdinSerialize]
    public MyClassByOdin myOdinReference = new MyClassByOdin();

    private void Start()
    {
        Debug.Log(firstDictionary.Count);
        Debug.Log(myUnityReference.secondDictionary_Unity.Count);
        Debug.Log(myOdinReference.secondDictionary_Odin.Count);
    }

}

[Serializable]
public class MyClassByUnity
{
    // 虽然标记为 OdinSerialize 特性, 但是依然不会被序列化
    [OdinSerialize]
    public Dictionary secondDictionary_Unity = new Dictionary();
}

[Serializable]
public class MyClassByOdin
{
    [OdinSerialize]
    [NonSerialized]
    public Dictionary secondDictionary_Odin= new Dictionary();
}
```

成员变量`MyUnityReference`是MyClassByUnity类型，在`MyClassByUnity`类中对应的成员SecondDictionary_Unity标记特性`OdinSerialize`,按常理`secondDictionary_Unity`字段会使用Odin序列化，但是`MyClassByUnity`类标记了`Serializable`特性，所以`MyClassByUnity`会按照Unity序列化的方式，又因为MyClassByUnity为Unity序列化不支持的类，所以直接跳过。类似一个for循环中，在索引**0**处执行break，后面的数据不会遍历一样（此处感觉略微啰嗦）。

为了避免上面的情况发生，我们使用`[NonSerialized, OdinSerialize]`两个特性，这是告诉编辑器，拒绝使用Unity原有的序列化，而是强制使用Odin自带的序列化系统，这样，上面的问题就可迎刃而解。