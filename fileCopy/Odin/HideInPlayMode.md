# Hide In Play Mode

> 在Play模式下隐藏对应属性

![img](https://aihailan.com/wp-content/uploads/2020/11/post-602-5fb7da0bbc27c.gif)

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class HideInPlayModeAttributeExample : MonoBehaviour
{
    [Title("Hidden in play mode")]
    [HideInPlayMode]
    public int A;

    [HideInPlayMode]
    public int B;
}
```