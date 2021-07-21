# Range

> *Unity自带属性，用于给一个数值创建一个滑动控件*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-660-5fb7dbe8be247.gif)

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class RangeAttributeExample : MonoBehaviour
{
    [Range(0, 10)]
    public int Field = 2;

    [InfoBox("Odin的PropertyRange属性类似于Unity的Range属性，但也适用于属性。")]
    [ShowInInspector, PropertyRange(0, 10)]
    public int Property { get; set; }

    [InfoBox("您还可以为最小值和最大值中的一个或两个引用成员.")]
    [PropertyRange(0, "Max"), PropertyOrder(3)]
    public int Dynamic = 6;

    [PropertyOrder(4)]
    public int Max = 100;
}
```