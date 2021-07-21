# Disable Inline Editor

> *用于在Inline中禁用（灰态）对应的属性*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-558-5fb7d8145d051.png)

```cs
public class DisableInInlineEditorsAttributeExample : MonoBehaviour
{
    [InfoBox("Click the pen icon to open a new inspector window for the InlineObject too see the difference this attribute make.")]
    [InlineEditor(Expanded = true)]
    public MyInlineScriptableObject InlineObject ;
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