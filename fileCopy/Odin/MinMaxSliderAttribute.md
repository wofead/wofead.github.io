# Min Max Slider

> Min Max Slider:*用于绘制一个特殊的滑块，用户可以用来指定最小值和最大值之间的范围。使用Vector2，其中x为最小值，y为最大值。*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-636-5fb7db160a4aa.gif)

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class MinMaxSliderAttributeExample : MonoBehaviour
{
    [MinMaxSlider(-10, 10)]
    public Vector2 MinMaxValueSlider = new Vector2(-7, -2);

    [MinMaxSlider(-10, 10, true)]
    public Vector2 WithFields = new Vector2(-3, 4);

    [InfoBox("您还可以通过引用成员来动态分配最小最大值。.")]
    [MinMaxSlider("DynamicRange", true)]
    public Vector2 DynamicMinMax = new Vector2(25, 50);

    [MinMaxSlider("Min", 10f, true)]
    public Vector2 DynamicMin = new Vector2(2, 7);

    [InfoBox("您还可以使用带有@符号的属性表达式.")]
    [MinMaxSlider("@DynamicRange.x", "@DynamicRange.y * 10f", true)]
    public Vector2 Expressive = new Vector2(0, 450);

    public Vector2 DynamicRange = new Vector2(0, 50);

    public float Min { get { return this.DynamicRange.x; } }

    public float Max { get { return this.DynamicRange.y; } }

}
```