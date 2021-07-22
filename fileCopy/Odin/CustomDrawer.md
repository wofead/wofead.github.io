# Custom Drawer

> 本章简述如何基于Odin制作可绘制的特性
>
> 本示例是在一个属性上面添加一个自定义特性，然后这个属性会基于这个特性按照我们定于的效果绘制。

##### 创建一个我们示例类

```cs
    // 演示如何为属性创建自定义drawer的示例。
    [TypeInfoBox("这里是使用自定义属性drawer绘制的HealthBar栏的可视化")]
    public class HealthBarExample : MonoBehaviour
    {
        public float Health;
    }
```

##### 随后我们要创建一个特性，放在 `Health`属性上面

```cs
    // Attribute used by HealthBarAttributeDrawer.
    [AttributeUsage(AttributeTargets.Field | AttributeTargets.Property)]
    public class HealthBarAttribute : Attribute
    {
        public float MaxHealth { get; private set; }

        public HealthBarAttribute(float maxHealth)
        {
            this.MaxHealth = maxHealth;
        }
    }
```

##### 添加特性

> 唯一的变化就是在属性`Health`添加了特性`[HealthBar(100)]`

```cs
    // 演示如何为属性创建自定义drawer的示例。
    [TypeInfoBox("这里是使用自定义属性drawer绘制的HealthBar栏的可视化")]
    public class HealthBarExample : MonoBehaviour
    {
        [HealthBar(100)]
        public float Health;
    }
```

> 这样我们在使用的层面上算是完成了，但是我们怎么自定义绘制呢？接下来就需要给这个特性`HealthBarAttribute `做详细的绘制规划了
> 创建HealthBarAttributeDrawer 并集成OdinAttributeDrawer，需要两个参数，第一个参数是需要绘制的特性，第二个参数是绘制这个特性中的那种类型字段，因为我们绘制的是`float Health`,所以第二个参数填写float
> 下面的代码主要实现如下几步
>
> - **获取此类型float对应的值`ValueEntry.SmartValue`**
> - **获取一块绘制区域`Rect rect = EditorGUILayout.GetControlRect();`**
> - **绘制底色`SirenixEditorGUI.DrawSolidRect(rect, new Color(0f, 0f, 0f, 0.3f), false);`**
> - **按照百分比出现的红色`SirenixEditorGUI.DrawSolidRect(rect.SetWidth(rect.width \* width), Color.red, false);`**
> - **健康条的Outline（边框）` SirenixEditorGUI.DrawBorders(rect, 1);`**

```cs
#if UNITY_EDITOR
    // Place the drawer script file in an Editor folder or wrap it in a #if UNITY_EDITOR condition.
    public class HealthBarAttributeDrawer : OdinAttributeDrawer<HealthBarAttribute, float>
    {
        protected override void DrawPropertyLayout(GUIContent label)
        {
            // 让此label传递下去，便于其他的特性进行绘制
            this.CallNextDrawer(label);

            // 找一个矩形来绘制健康条。您也可以使用GUILayout，但是使用rects使绘制健康栏变得更简单。
            Rect rect = EditorGUILayout.GetControlRect();

            // Draw the health bar.
            float width = Mathf.Clamp01(this.ValueEntry.SmartValue / this.Attribute.MaxHealth);
            SirenixEditorGUI.DrawSolidRect(rect, new Color(0f, 0f, 0f, 0.3f), false);
            SirenixEditorGUI.DrawSolidRect(rect.SetWidth(rect.width * width), Color.red, false);
            SirenixEditorGUI.DrawBorders(rect, 1);
        }
    }
#endif
```

![file](https://aihailan.com/wp-content/uploads/2020/11/image-1614665205210.png)



### 注意

> 自定义绘制的时候需要一些Unity原本GUI的知识，Odin本身的API就已经众多，如果要达到和Odin原始提供特性一样的出色效果，还是有一些难度的。
> 而且在使用Unity原本EditorAPI的时候，更推荐使用Rect方式，避免使用GUILayout相关。

##### 完整示例代码

```cs
#if UNITY_EDITOR
namespace Sirenix.OdinInspector.Demos
{
    using System;
    using UnityEngine;

#if UNITY_EDITOR

    using Sirenix.OdinInspector.Editor;
    using UnityEditor;
    using Sirenix.Utilities.Editor;
    using Sirenix.Utilities;

#endif

    // 演示如何为属性创建自定义drawer的示例。
    [TypeInfoBox("这里是使用自定义属性drawer绘制的HealthBar栏的可视化")]
    public class HealthBarExample : MonoBehaviour
    {
        [HealthBar(100)]
        public float Health;
    }

    // Attribute used by HealthBarAttributeDrawer.
    [AttributeUsage(AttributeTargets.Field | AttributeTargets.Property)]
    public class HealthBarAttribute : Attribute
    {
        public float MaxHealth { get; private set; }

        public HealthBarAttribute(float maxHealth)
        {
            this.MaxHealth = maxHealth;
        }
    }

#if UNITY_EDITOR
    // Place the drawer script file in an Editor folder or wrap it in a #if UNITY_EDITOR condition.
    public class HealthBarAttributeDrawer : OdinAttributeDrawer<HealthBarAttribute, float>
    {
        protected override void DrawPropertyLayout(GUIContent label)
        {
            // 让此label传递下去，便于其他的特性进行绘制
            this.CallNextDrawer(label);

            // 找一个矩形来绘制健康条。您也可以使用GUILayout，但是使用rects使绘制健康栏变得更简单。
            Rect rect = EditorGUILayout.GetControlRect();

            // Draw the health bar.
            float width = Mathf.Clamp01(this.ValueEntry.SmartValue / this.Attribute.MaxHealth);
            SirenixEditorGUI.DrawSolidRect(rect, new Color(0f, 0f, 0f, 0.3f), false);
            SirenixEditorGUI.DrawSolidRect(rect.SetWidth(rect.width * width), Color.red, false);
            SirenixEditorGUI.DrawBorders(rect, 1);
        }
    }
#endif
}
#endif
```