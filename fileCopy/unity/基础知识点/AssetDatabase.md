# AssetDatabase

AssetDatabase 是一个API，可用于访问项目中包含的资源。除此之外，它还提供了查找和加载资源的方法，以及创建、删除和修改资源的方法。Unity Editor 在内部使用 AssetDatabase 来跟踪资源文件并维护资源和引用它们的对象之间的链接关系。由于 Unity 需要跟踪项目文件夹的所有更改，因此如果要访问或修改资源数据，则应始终使用 AssetDatabase API 而不是文件系统。

AssetDatabase 接口仅在编辑器中可用，并且在构建的播放器中没有任何函数。与所有其他编辑器类一样，它仅适用于放置在 Editor 文件夹中的脚本（直接在项目的 Assets 主文件夹中创建名为“Editor”的文件夹，如果还没有的话）。

## 导入资源

Unity 通常会在资源被拖入项目时自动导入资源，但也可以在脚本控制下导入资源。为此，可以使用 [AssetDatabase.ImportAsset](https://docs.unity.cn/cn/2018.4/ScriptReference/AssetDatabase.ImportAsset.html) 方法，如以下示例所示。

```c#
using UnityEngine;
using UnityEditor;

public class ImportAsset {
    [MenuItem ("AssetDatabase/ImportExample")]
    static void ImportExample ()
    {
        AssetDatabase.ImportAsset("Assets/Textures/texture.jpg", ImportAssetOptions.Default);
    }
}
```

还可以将 [AssetDatabase.ImportAssetOptions](https://docs.unity.cn/cn/2018.4/ScriptReference/ImportAssetOptions.html) 类型的额外参数传递给 AssetDatabase.ImportAsset 调用。脚本参考页面介绍了不同的选项及其对函数行为的影响。

## 加载资源

编辑器仅在需要时加载资源，例如，是否将资源添加到场景或从 Inspector 面板中进行编辑。但是，可以使用 [AssetDatabase.LoadAssetAtPath](https://docs.unity.cn/cn/2018.4/ScriptReference/AssetDatabase.LoadAssetAtPath.html)、[AssetDatabase.LoadMainAssetAtPath](https://docs.unity.cn/cn/2018.4/ScriptReference/AssetDatabase.LoadMainAssetAtPath.html)、[AssetDatabase.LoadAllAssetRepresentationsAtPath](https://docs.unity.cn/cn/2018.4/ScriptReference/AssetDatabase.LoadAllAssetRepresentationsAtPath.html) 和 [AssetDatabase.LoadAllAssetsAtPath](https://docs.unity.cn/cn/2018.4/ScriptReference/AssetDatabase.LoadAllAssetsAtPath.html) 从脚本加载和访问资源。有关更多详细信息，请参阅脚本文档。

```c#
using UnityEngine;
using UnityEditor;

public class ImportAsset {
    [MenuItem ("AssetDatabase/LoadAssetExample")]
    static void ImportExample ()
    {
        Texture2D t = AssetDatabase.LoadAssetAtPath("Assets/Textures/texture.jpg", typeof(Texture2D)) as Texture2D;
    }
}
```

## 使用 AssetDatabase 进行文件操作

由于 Unity 保留有关资源文件的元数据，因此不应使用文件系统创建、移动或删除这些文件。相反，可以使用 [AssetDatabase.Contains](https://docs.unity.cn/cn/2018.4/ScriptReference/AssetDatabase.Contains.html)、[AssetDatabase.CreateAsset](https://docs.unity.cn/cn/2018.4/ScriptReference/AssetDatabase.CreateAsset.html)、[AssetDatabase.CreateFolder](https://docs.unity.cn/cn/2018.4/ScriptReference/AssetDatabase.CreateFolder.html)、[AssetDatabase.RenameAsset](https://docs.unity.cn/cn/2018.4/ScriptReference/AssetDatabase.RenameAsset.html)、[AssetDatabase.CopyAsset](https://docs.unity.cn/cn/2018.4/ScriptReference/AssetDatabase.CopyAsset.html)、[AssetDatabase.MoveAsset](https://docs.unity.cn/cn/2018.4/ScriptReference/AssetDatabase.MoveAsset.html)、[AssetDatabase.MoveAssetToTrash](https://docs.unity.cn/cn/2018.4/ScriptReference/AssetDatabase.MoveAssetToTrash.html) 和 [AssetDatabase.DeleteAsset](https://docs.unity.cn/cn/2018.4/ScriptReference/AssetDatabase.DeleteAsset.html)。

```c#
public class AssetDatabaseIOExample {
    [MenuItem ("AssetDatabase/FileOperationsExample")]
    static void Example ()
    {
        string ret;
        
        // 创建
        Material material = new Material (Shader.Find("Specular"));
        AssetDatabase.CreateAsset(material, "Assets/MyMaterial.mat");
        if(AssetDatabase.Contains(material))
            Debug.Log("Material asset created");
        
        // 重命名
        ret = AssetDatabase.RenameAsset("Assets/MyMaterial.mat", "MyMaterialNew");
        if(ret == "")
            Debug.Log("Material asset renamed to MyMaterialNew");
        else
            Debug.Log(ret);
        
        // 创建文件夹
        ret = AssetDatabase.CreateFolder("Assets", "NewFolder");
        if(AssetDatabase.GUIDToAssetPath(ret) != "")
            Debug.Log("Folder asset created");
        else
            Debug.Log("Couldn't find the GUID for the path");
        
        // 移动
        ret = AssetDatabase.MoveAsset(AssetDatabase.GetAssetPath(material), "Assets/NewFolder/MyMaterialNew.mat");
        if(ret == "")
            Debug.Log("Material asset moved to NewFolder/MyMaterialNew.mat");
        else
            Debug.Log(ret);
        
        // 复制
        if(AssetDatabase.CopyAsset(AssetDatabase.GetAssetPath(material), "Assets/MyMaterialNew.mat"))
            Debug.Log("Material asset copied as Assets/MyMaterialNew.mat");
        else
            Debug.Log("Couldn't copy the material");
        // 手动刷新数据库以通知更改
        AssetDatabase.Refresh();
        Material MaterialCopy = AssetDatabase.LoadAssetAtPath("Assets/MyMaterialNew.mat", typeof(Material)) as Material;
        
        // 移到垃圾箱
        if(AssetDatabase.MoveAssetToTrash(AssetDatabase.GetAssetPath(MaterialCopy)))
            Debug.Log("MaterialCopy asset moved to trash");
        
        // 删除
        if(AssetDatabase.DeleteAsset(AssetDatabase.GetAssetPath(material)))
            Debug.Log("Material asset deleted");
        if(AssetDatabase.DeleteAsset("Assets/NewFolder"))
            Debug.Log("NewFolder deleted");
        
        // 进行所有更改后刷新 AssetDatabase
        AssetDatabase.Refresh();
    }
}
```

## 使用 AssetDatabase.Refresh

在完成资源的修改后，应调用 AssetDatabase.Refresh 来提交对数据库的更改并使其在项目中可见。

