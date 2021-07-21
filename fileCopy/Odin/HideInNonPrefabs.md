# Hide In Non Prefabs

> *用于当属性所在的组件在非预制体上面时，则隐藏属性*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-600-5fb7d9fec9190.png)

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class HideInNonPrefabsAttributeExample : MonoBehaviour
{
    [HideInNonPrefabs] //非预制体时隐藏属性
    public GameObject HiddenInNonPrefabs;
}
```