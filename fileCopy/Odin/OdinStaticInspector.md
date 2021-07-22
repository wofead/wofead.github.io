# Odin Static Inspector

> **Odin Static Inspector，一个快速搜索并允许调用相应的静态成员的便捷工具，提高测试效率。**
>
> 使用起来非常方便，只需要打开**Tools/Odin Inspector/Static Inspector**即可打开对应的操作面板

![img](https://aihailan.com/wp-content/uploads/2020/11/post-642-5fb7db5244a6d.gif)

> ##### 快速搜索需要调试的静态类

![img](https://aihailan.com/wp-content/uploads/2020/11/post-642-5fb7db5296ecd.gif)

> ##### 可以搜索及过滤对应的成员

![img](https://aihailan.com/wp-content/uploads/2020/11/post-642-5fb7db52edcc5.gif)

> ##### 可以配合Odin特性进行相关函数等功能的调用

![img](https://aihailan.com/wp-content/uploads/2020/11/post-642-5fb7db533a964.gif)

##### 简单示例代码

```cs
using Sirenix.OdinInspector;
using System.Collections.Generic;
using UnityEngine;

public class StaticInspectorTutorials : MonoBehaviour
{
    public enum TempEnum
    {
        One,Two,Three
    }
    public static TempEnum tempEnum;
    public static string tempStr;
    public static int tempInt;
    public static List staticInspectorTutorials_Ones = new List();

    [Button(ButtonSizes.Large)]
    public static void TestStaticFunction()
    {
        Debug.Log("TestFunction");
    }
    [Button(ButtonSizes.Large, ButtonStyle.FoldoutButton)]
    public static void TestStaticFunction(string str)
    {
        Debug.Log($"TestFunction:{str}");
    }

    [Button(ButtonSizes.Large, ButtonStyle.FoldoutButton)]
    public static void TestStaticFunction(List tempList)
    {
        for (int i = 0; i < tempList.Count; i++)
        {
            Debug.Log($"List Index :{i}---value:{tempList[i]}");
        }
    }

    public void NoStaticFunction()
    {
        Debug.Log("NoStaticFunction");
    }
}

public  class StaticInspectorTutorials_One
{
    public static string tempStr;
}
```