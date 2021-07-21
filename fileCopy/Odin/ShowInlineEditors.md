# Show Inline Editors

> 用于在Inline中显示对应的属性。

![img](https://aihailan.com/wp-content/uploads/2020/11/post-676-5fb7dc681d52c.png)

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class ShowInInlineEditorsAttributeExample : MonoBehaviour
{
    [InfoBox("单击属性值打开一个新的检查窗口，也可以看到这些属性的不同.")]
    [InlineEditor(Expanded = true)]
    public MyInlineScriptableObject InlineObject;
}

[CreateAssetMenu(fileName = "MyInline_ScriptableObject", menuName = "CreatScriptableObject/MyInlineScriptableObject")]
public class MyInlineScriptableObject : ScriptableObject
{
    [ShowInInlineEditors]
    public string ShownInInlineEditor;

    [HideInInlineEditors]//在绘制的里面不显示
    public string HiddenInInlineEditor;

    [DisableInInlineEditors]//显示但是是灰态
    public string DisabledInInlineEditor;
}
```