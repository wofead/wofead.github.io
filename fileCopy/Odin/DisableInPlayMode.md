# Disable In Play Mode

> *在play模式下灰态指定属性，editor模式下显示*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-562-5fb7d8447a19e.gif)

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class DisableInPlayModeAttributeExample : MonoBehaviour
{
    [Title("运行模式下禁用属性")]
    [DisableInPlayMode]
    public int A;

    [DisableInPlayMode]
    public Material B;
}
```

