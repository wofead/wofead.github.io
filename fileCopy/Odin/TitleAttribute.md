# Title Attribute

> *Title Attribute*特性：用于在属性上方生成粗体标题。

![img](https://aihailan.com/wp-content/uploads/2020/11/post-694-5fb7dcf414233.png)

##### 直接设置标题，或者添加标题和副标题

![img](https://aihailan.com/wp-content/uploads/2020/11/post-694-5fb7dcf4a8984.png)

```cs
    [Title("Static title")]
    public int C;
    public int D;

    [Title("Static title", "Static subtitle")]
    public int E;
    public int F;
```

##### 还可以设置标题是否为粗体和是否含有对应的下划线

![img](https://aihailan.com/wp-content/uploads/2020/11/post-694-5fb7dcf54d82d.png)

```cs
    [Title("Non bold title", "$MySubtitle", bold: false)]
    public int I;
    public int J;

    [Title("Non bold title", "With no line seperator", horizontalLine: false, bold: false)]
    public int K;
    public int L;
```

##### 也可以设置标题的不同布局

![img](https://aihailan.com/wp-content/uploads/2020/11/post-694-5fb7dcf5db883.png)

```cs
    [Title("$MyTitle", "$MySubtitle", TitleAlignments.Right)]
    public int M;
    public int N;

    [Title("$MyTitle", "$MySubtitle", TitleAlignments.Centered)]
    public int O;
    public int P;

    [Title("$MyTitle", "$MySubtitle", titleAlignment: TitleAlignments.Left)]
    public int Q;
    public int R;
    [Title("$MyTitle", "$MySubtitle", titleAlignment: TitleAlignments.Split)]
    public int S;
    public int T;
```

##### 同样，可是用特殊标识符$来获取一个属性字段或者函数的返回值作为消息内容

![img](https://aihailan.com/wp-content/uploads/2020/11/post-694-5fb7dcf5f3f77.gif)

##### 也可以使用特殊标识符@将方法体以字符串的形式当实参传入进去

![img](https://aihailan.com/wp-content/uploads/2020/11/post-694-5fb7dcf64bc17.gif)

##### 完整示例代码

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class TitleAttributeExample : MonoBehaviour
{
    [Title("Titles and Headers")]
    public string MyTitle = "My Dynamic Title";
    public string MySubtitle = "My Dynamic Subtitle";

    [Title("Static title")]
    public int C;
    public int D;

    [Title("Static title", "Static subtitle")]
    public int E;
    public int F;

    [Title("$MyTitle", "$MySubtitle")]
    public int G;
    public int H;

    [Title("Non bold title", "$MySubtitle", bold: false)]
    public int I;
    public int J;

    [Title("Non bold title", "With no line seperator", horizontalLine: false, bold: false)]
    public int K;
    public int L;

    [Title("$MyTitle", "$MySubtitle", TitleAlignments.Right)]
    public int M;
    public int N;

    [Title("$MyTitle", "$MySubtitle", TitleAlignments.Centered)]
    public int O;
    public int P;

    [Title("$MyTitle", "$MySubtitle", titleAlignment: TitleAlignments.Left)]
    public int Q;
    public int R;
    [Title("$MyTitle", "$MySubtitle", titleAlignment: TitleAlignments.Split)]
    public int S;
    public int T;

    [ShowInInspector]
    [Title("Title on a Property")]
    public int U { get; set; }

    [Title("Title on a Method")]
    [Button]
    public void DoNothing()
    { }

    [Title("@DateTime.Now.ToString(\"dd:MM:yyyy\")", "@DateTime.Now.ToString(\"HH:mm:ss\")")]
    public int Expresion;

    public string Combined { get { return this.MyTitle + " - " + this.MySubtitle; } }
}
```