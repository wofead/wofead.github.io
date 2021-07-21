# Disable In Prefab Instances 

> *用于当属性所在的组件在预制体上且预制体在Hierarchy（实例）中时，禁用属性*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-566-5fb7d8a273608.png)

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class DisableInPrefabInstancesAttributeExample : MonoBehaviour
{
    [DisableInPrefabInstances]//在hierarchy中为预制体时则禁用此属性
    public GameObject DisabledInPrefabInstances;
}
```