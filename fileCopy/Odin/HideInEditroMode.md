# Hide In Editor Mode

> *用于在editor模式中隐藏指定属性，在play模式中显示*



![img](https://aihailan.com/wp-content/uploads/2020/11/post-596-5fb7d9dadf226.gif)

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class HideInEditorModeAttributeExample : MonoBehaviour
{
    [Title("Hidden in editor mode")]//在editor下隐藏属性，运行时显示属性
    [HideInEditorMode]
    public int C;

    [HideInEditorMode]
    public int D;
}
```