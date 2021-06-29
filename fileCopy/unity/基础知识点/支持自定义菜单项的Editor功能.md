## 支持自定义菜单项和 Editor 功能

在 Editor 脚本中，使用 [ObjectFactory](https://docs.unity.cn/cn/2018.4/ScriptReference/ObjectFactory.html) 类可创建新的游戏对象、组件和资源。创建这些项时，`ObjectFactory` 类自动使用默认预设。脚本不必搜索并应用默认[预设](https://docs.unity.cn/cn/2018.4/Manual/Presets.html)，因为 `ObjectFactory` 会负责处理此问题。

## 支持新类型

要默认支持和启用预设，您的类必须继承自以下之一：

- [UnityEngine.Monobehaviour](https://docs.unity.cn/cn/2018.4/ScriptReference/MonoBehaviour.html)
- [UnityEngine.ScriptableObject](https://docs.unity.cn/cn/2018.4/ScriptReference/ScriptableObject.html)
- [UnityEngine.ScriptedImporter](https://docs.unity.cn/cn/2018.4/ScriptReference/Experimental.AssetImporters.ScriptedImporter.html)

Preset Inspector 会创建类的临时实例，便于用户修改其值，因此请确保您的类不会影响或依赖其他对象，如静态值、项目资源或场景实例。

**提示**：[CustomEditor](https://docs.unity.cn/cn/2018.4/ScriptReference/CustomEditor.html) 属性是可选的。

## 用例示例：自定义 Editor 窗口中的预设设置

使用可能使用预设的设置来设置自定义 [EditorWindow](https://docs.unity.cn/cn/2018.4/ScriptReference/EditorWindow.html) 类时：

- 使用 [ScriptableObject](https://docs.unity.cn/cn/2018.4/ScriptReference/ScriptableObject.html) 来存储设置副本。还可以有 [CustomEditor](https://docs.unity.cn/cn/2018.4/ScriptReference/CustomEditor.html) 属性。预设系统会处理此对象。
- 始终使用此临时 `ScriptableObject` Inspector 在 UI 中显示预设设置。这样，您的用户在 `EditorWindow` 中和编辑保存的预设时会看到相同的 UI。
- 在 **Select Preset** 窗口中选择预设时，显示 Preset 按钮并使用您自己的 [PresetSelectorReceiver](https://docs.unity.cn/cn/2018.4/ScriptReference/Presets.PresetSelectorReceiver.html) 实现使 `EditorWindow` 设置保持最新。

以下脚本示例演示了如何将预设设置添加到简单的 `EditorWindow`。

此脚本示例演示了一个 ScriptableObject 在自定义窗口中保存并显示设置（保存到名为 *Editor/MyWindowSettings.cs* 的文件）：

```c#
using UnityEngine;

// 预设系统使用的临时 ScriptableObject

public class MyWindowSettings : ScriptableObject
{
    [SerializeField]
    string m_SomeSettings;
    
    public void Init(MyEditorWindow window)
    {
        m_SomeSettings = window.someSettings;
    }
    
    public void ApplySettings(MyEditorWindow window)
    {
        window.someSettings = m_SomeSettings;
        window.Repaint();
    }
}
```

用于更新自定义窗口中使用的 `ScriptableObject` 的 `PresetSelectorReceiver` 脚本示例（保存到名为 *Editor/MySettingsReceiver.cs* 的文件）：

```c#
using UnityEditor.Presets;

// PresetSelector 接收器用所选值更新 EditorWindow。

public class MySettingsReceiver : PresetSelectorReceiver
{
    Preset initialValues;
    MyWindowSettings currentSettings;
    MyEditorWindow currentWindow;
    
    public void Init(MyWindowSettings settings, MyEditorWindow window)
    {
        currentWindow = window;
        currentSettings = settings;
        initialValues = new Preset(currentSettings);
    }
    
    public override void OnSelectionChanged(Preset selection)
    {
        if (selection != null)
        {
            //将选择应用于临时设置
            selection.ApplyTo(currentSettings);
        }
        else
        {
            // 未进行任何选择。将初始值反向应用于临时选择。
            initialValues.ApplyTo(currentSettings);
        }
        
        //将新的临时设置应用于我们的管理器实例
        currentSettings.ApplySettings(currentWindow);
    }
    
    public override void OnSelectionClosed(Preset selection)
    {
        // 最后一次调用选择更改以确保您具有最后的选择值。
        OnSelectionChanged(selection);
        //在此处销毁接收器，所以您不需要保留其引用。
        DestroyImmediate(this);
    }
}
```

使用临时 ScriptableObject Inspector 及其 Preset 按钮显示自定义设置的 [EditorWindow](https://docs.unity.cn/cn/2018.4/ScriptReference/EditorWindow.html) 脚本示例（保存到名为 *Editor/MyEditorWindow.cs* 的文件）：

```c#
using UnityEngine;
using UnityEditor;
using UnityEditor.Presets;

public class MyEditorWindow : EditorWindow

{
    // 获取预设图标和样式来进行显示
    private static class Styles
    {
        public static GUIContent presetIcon = EditorGUIUtility.IconContent("Preset.Context");
        public static GUIStyle iconButton = new GUIStyle("IconButton");

    }

    Editor m_SettingsEditor;
    MyWindowSettings m_SerializedSettings;
    
    public string someSettings
    {
        get { return EditorPrefs.GetString("MyEditorWindow_SomeSettings"); }
        set { EditorPrefs.SetString("MyEditorWindow_SomeSettings", value); }
    }
   
    // 打开窗口的方法
    [MenuItem("Window/MyEditorWindow")]
    static void OpenWindow()
    {
        GetWindow<MyEditorWindow>();
    }

    void OnEnable()
    {
        // 立即创建您的设置及其关联的 Inspector，
        // 用于为窗口和预设中的设置仅创建一个自定义 Inspector。
        m_SerializedSettings = ScriptableObject.CreateInstance<MyWindowSettings>();
        m_SerializedSettings.Init(this);
        m_SettingsEditor = Editor.CreateEditor(m_SerializedSettings);
    }

    void OnDisable()
    {
        Object.DestroyImmediate(m_SerializedSettings);
        Object.DestroyImmediate(m_SettingsEditor);
    }

    void OnGUI()
    {
        EditorGUILayout.BeginHorizontal();
        EditorGUILayout.LabelField("My custom settings", EditorStyles.boldLabel);
        GUILayout.FlexibleSpace();
        // 在"MyManager Settings"行的末尾创建 Preset 按钮。
        var buttonPosition = EditorGUILayout.GetControlRect(false, EditorGUIUtility.singleLineHeight, Styles.iconButton);

        if (EditorGUI.DropdownButton(buttonPosition, Styles.presetIcon, FocusType.Passive, Styles.iconButton))
        {
            //创建接收器实例。当窗口出现时，此接收器会自行毁坏，所以您不需要保留其引用。
            var presetReceiver = ScriptableObject.CreateInstance<MySettingsReceiver>();
            presetReceiver.Init(m_SerializedSettings, this);
            //显示 PresetSelector 模态窗口。presetReceiver 会更新您的数据。
            PresetSelector.ShowSelector(m_SerializedSettings, null, true, presetReceiver);
        }
        EditorGUILayout.EndHorizontal();
        
        //绘制设置默认 Inspector 并捕获对其所做的任何更改。
        EditorGUI.BeginChangeCheck();
        m_SettingsEditor.OnInspectorGUI();

        if (EditorGUI.EndChangeCheck())
        {
            //将设置编辑器中所做的更改应用于我们的实例。
            m_SerializedSettings.ApplySettings(this);
        }
    }
}
```

