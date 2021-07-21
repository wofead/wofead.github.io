# Disable In Non Prefabs

> *用于当属性所在的组件在非预制体上面时，禁用对应的属性*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-560-5fb7d82e66b30.png)

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class DisableInNonPrefabsAttributeExample : MonoBehaviour
{
    [InfoBox("这些属性只有在检查GameObject的组件时才会起作用。")]

    [DisableInNonPrefabs] // 当不是预制体是灰态此属性
    public GameObject DisabledInNonPrefabs;
}
```