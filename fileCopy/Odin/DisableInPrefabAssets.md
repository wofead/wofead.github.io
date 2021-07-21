# Disable In Prefab Assets

> *用于当属性所在的组件是预制体，且预制体在Asset中时禁用属性*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-564-5fb7d88b9a40c.png)

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class DisableInPrefabAssetsAttributeExample : MonoBehaviour
{
    [DisableInPrefabAssets]//在asset中且为预制体时，这个属性被警用
    public GameObject DisabledInPrefabAssets;
}
```