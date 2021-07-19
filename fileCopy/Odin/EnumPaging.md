# EnumPaging

> Enum Paging Attribute特性：用于在检查器中使用下一个和上一个按钮护自己枚举选择器，以便循环访问枚举属性的可用值。

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class EnumPagingAttributeExample : MonoBehaviour
    {
        [EnumPaging]
        public SomeEnum SomeEnumField;

        public enum SomeEnum
        {
            A, B, C
        }

    [ShowInInspector]
    [EnumPaging, OnValueChanged("SetCurrentTool")]
    [InfoBox("Changing this property will change the current selected tool in the Unity editor.")]
    private UnityEditor.Tool sceneTool;

    private void SetCurrentTool()
    {
        UnityEditor.Tools.current = this.sceneTool;
    }
}
```