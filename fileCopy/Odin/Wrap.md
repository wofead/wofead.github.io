# Wrap

> 用于大多数原始属性，当值超出定义范围时，可以包装该值。当你需要一个绕圆的值（类似于角度）时，请使用此项。
>
> 类似： Mathf.PingPong

![img](https://aihailan.com/wp-content/uploads/2020/11/post-714-5fb7dd8c9205c.gif)

```csharp
using Sirenix.OdinInspector;
using UnityEngine;

public class WrapAttributeExample : MonoBehaviour
{
    [Wrap(0f, 100f)]
    public int IntWrapFrom0To100;

    [Wrap(0f, 100f)]
    public float FloatWrapFrom0To100;

    [Wrap(0f, 100f)]
    public Vector3 Vector3WrapFrom0To100;

    [Wrap(0f, 360)]
    public float AngleWrap;

    [Wrap(0f, Mathf.PI * 2)]
    public float RadianWrap;
}
```