# Hide In Prefab Instances

> *属性所在的组件在预制体上，且预制体在为instance（在Hierarchy中）时，隐藏属性*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-606-5fb7da29e9cd8.png)

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class HideInPrefabInstancesAttributeExample : MonoBehaviour
{
    [HideInPrefabInstances]
    public GameObject HiddenInPrefabInstances;
}
```