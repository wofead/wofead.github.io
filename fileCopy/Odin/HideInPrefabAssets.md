# Hide In Prefab Assets

> *属性所在的组件在预制体上，且预制体在为Asset（在project中）时，隐藏属性*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-604-5fb7da1c7e7eb.png)

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class HideInPrefabAssetsAttributeExample : MonoBehaviour
{
    [HideInPrefabAssets] //在Asset中且是预制体时隐藏
    public GameObject HiddenInPrefabAssets;
}
```