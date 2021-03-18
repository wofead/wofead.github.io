# Unity3D SerializeField

在Unity3d中Unity3D 中提供了非常方便的功能可以帮助用户将 成员变量 在Inspector中显示，并且定义Serialize关系。也就是说凡是显示在Inspector 中的属性都同时具有Serialize功能(**序列化的意思是说再次读取Unity时序列化的变量是有值的，不需要你再次去赋值，因为它已经被保存下来**)。

所以在Unity中有两种方式序列化：

1. **public**：在没有加入任何Attribute的前提下，public变量是默认被视为可以被Serialize的。所以public声明的变量在Inspector面板中是可见的。而Private变量在Inspector视图面板是不可见的。

2. **[SerializeField] Attribute**：强制unity去序列化一个私有域。这是一个内部的unity序列化功能，有时候我们需要Serialize一个private或者protected的属性，这个时候可以使用[SerializeField]这个Attribute:之后就可以在面板看到该变量。

   

```c#
using UnityEngine;

public class Graph : MonoBehaviour
{
    public prefab = default;
    [SerializeField]
    Transform pointPrefab = default;
    // Start is called before the first frame update
    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
        
    }
}
```

