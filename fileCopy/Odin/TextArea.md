# TextArea

> *Unity自带属性，用于在inspector面板中给字符绘制一个填写区域*



![img](https://aihailan.com/wp-content/uploads/2020/11/post-692-5fb7dce7072e0.png)

```cs
using UnityEngine;

public class TextAreaAttributeExample : MonoBehaviour
{
    [TextArea]
    public string content = "";
}
```