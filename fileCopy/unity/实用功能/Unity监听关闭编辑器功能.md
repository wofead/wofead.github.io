# 监听关闭编辑器功能

```c#
using UnityEditor;
public class Test 
{
 
    [InitializeOnLoadMethod]
    static void InitializeOnLoadMethod()
    {
        EditorApplication.wantsToQuit -= Quit;
        EditorApplication.wantsToQuit += Quit;
    }
 
    static bool Quit()
    {
        EditorUtility.DisplayDialog("我就是不让你关闭unity", "我就是不让你关闭unity", "呵呵");
        return false; //return true表示可以关闭unity编辑器
    }
}
```

