# Max Value

> **用于基本字段。它将字段的值限制为最小值。使用此定义字段的最小值。**

![img](https://aihailan.com/wp-content/uploads/2020/11/post-638-5fb7db2751f81.gif)

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class MinValueAttributeExample : MonoBehaviour
{
    // Ints
    [Title("Int")]
    [MinValue(0)]
    public int IntMinValue0;

    // Floats
    [Title("Float")]
    [MinValue(0)]
    public float FloatMinValue0;

    // Vectors
    [Title("Vectors")]
    [MinValue(0)]
    public Vector3 Vector3MinValue0;
}
```