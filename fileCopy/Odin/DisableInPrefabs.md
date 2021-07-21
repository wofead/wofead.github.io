# Disable In Prefabs

> *用于当所在的属性的组件在预制体上时，禁用组件*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-568-5fb7d8ee4cfe3.png)

![img](https://aihailan.com/wp-content/uploads/2020/11/post-568-5fb7d8eec2fdf.png)

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class DisableInPrefabsAttributeExample : MonoBehaviour
{
    [DisableInPrefabs]//只要是预制体，就隐藏此属性，不管是否在asset还是hierarchy
    public GameObject DisabledInPrefabs;
}
```