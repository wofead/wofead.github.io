# Max Value

> *用于基本字段。它将字段的值限制为最大值。使用此定义字段的最大值。*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-634-5fb7db0868432.gif)

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class MaxValueAttributeExample : MonoBehaviour
{
    [MaxValue(0)]
    public int IntMaxValue0;

    [MaxValue(0)]
    public float FloatMaxValue0;

    [MaxValue(0)]
    public Vector3 Vector3MaxValue0;
}
```