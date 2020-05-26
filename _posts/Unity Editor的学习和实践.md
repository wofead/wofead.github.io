# Unity Editor的学习

[toc]

在Unity中我们可以通过代码来拓展编辑器，可以拓展Unity的5大视图：（Project、Inspector、Scene和Game），Unity中编辑器使用的代码应该仅限于编辑模式下，也就是说正式的游戏包不应该包含这些代码。Unity提供一个规则：如果属于编辑模式下的代码，需要放在Editor文件夹下；属于运行执行的代码，放到任意非Editor文件夹下即可。Editor文件夹的位置比较灵活，它还可以作为多个目录的子文件夹存在，这样开发者就可以按功能来划分，讲不同功能的编辑代码放在不同的Editor目录下。

## 拓展Project视图

自定义菜单的参数需要在MenuItem方法中写入显示的菜单路径。如果菜单条比较多，可以在第三个参数处输入表示排序的整数，数值越小它的排序就越靠前。我们可以使用Selection.activeObject来获取被选中的对象。

```c#
public static class ProjectEditor
{
    [MenuItem("Assets/My Tools/Tool 1", false, 2)]
    public static void MyTool1()
    {
        Debug.Log(Selection.activeObject.name);
    }

    [MenuItem("Assets/My Tools/Tool 2", false, 1)]
    public static void MyTool2()
    {
        Debug.Log(AssetDatabase.GetAssetPath(Selection.activeObject));
    }
    
    [MenuItem("Assets/My Create/Cube", false, 13)]
    public static void MyCreate1()
    {
        //直接作用于当前场景，在当前场景中添加一个Cube
        GameObject.CreatePrimitive(PrimitiveType.Cube);
    }

}
```

我们还可以扩展布局，例如当按钮选中一个资源后，右边将出现扩展后的click按钮。

```c#
[InitializeOnLoadMethod]
    public static void InitializeOnLoadMethod()
    {
        EditorApplication.projectWindowItemOnGUI = delegate (string guid, Rect selectionRect)
        {
            //在projec视图中选择一个资源
            if (Selection.activeObject && guid == AssetDatabase.AssetPathToGUID(AssetDatabase.GetAssetPath(Selection.activeObject)))
            {
                float width = 50f;
                selectionRect.x += (selectionRect.width - width);
                selectionRect.y += 2f;
                selectionRect.width = width;
                GUI.color = Color.gray;
                if (GUI.Button(selectionRect, "click"))
                {
                    Debug.LogFormat("click : {0}", Selection.activeObject.name);
                }
                GUI.color = Color.white;
            }
        };
    }
```

我们还可以通过监听Project中资源改变的事件来管理我们的资源，不经常使用，就先这样吧。

## 扩展Hierarchy视图

可以在Hierarchy中右键创建对象，也可以像上面的一样创建一个按钮。

```c#
[MenuItem("GameObject/My Create/Cube", false, 0)]
    public static void MyCreate2()
    {
        //直接作用于当前场景，在当前场景中添加一个Cube
        GameObject.CreatePrimitive(PrimitiveType.Cube);
    }
[InitializeOnLoadMethod]
    public static void InitializeOnLoadMethod()
    {
        EditorApplication.hierarchyWindowItemOnGUI = delegate (int instanceID, Rect selectionRect)
        {
            //在projec视图中选择一个资源
            if (Selection.activeObject && instanceID == Selection.activeObject.GetInstanceID())
            {
                float width = 50f;
                float height = 20f;
                selectionRect.x += (selectionRect.width - width);
                selectionRect.y += 2f;
                selectionRect.width = width;
                selectionRect.height = height;
                GUI.color = Color.gray;
                if (GUI.Button(selectionRect, AssetDatabase.LoadAssetAtPath<Texture>("Assets/unity.png")))
                {
                    Debug.LogFormat("click : {0}", Selection.activeObject.name);
                }
                GUI.color = Color.white;
            }
        };
    }
```

## 扩展Inspector视图

## Unity editor

```c#
using UnityEditor;
using UnityEngine;
using System.IO;
using BombMan;
using System.Text;

/// <summary>
/// 这个类用来导出hero、npc、bomb一些属性
/// rigidbody中的Mass、Gravity Scale
/// collider 中的Is Trigger、Offset、Size、Friction、Bounciness
/// </summary>
/// File.Exists(Application.persistentDataPath + "/onMobileSavedScreen.png")
/// 如果以后调用很慢的话，尝试生成一个，然后删除添加
public static class RoleExportDataTools
{
    public static readonly string HERO_FILE = Application.dataPath + "/GameAssets/Prefabs/Heroes/";
    public static readonly string NPC_FILE = Application.dataPath + "/GameAssets/Prefabs/Npc/";
    public static readonly string BOMB_FILE = Application.dataPath + "/GameAssets/Prefabs/Bomb/";
    static string[] heroFilePathes;
    static string[] npcFilePathes;
    static string[] bombFilePathes;
    static StringBuilder builder;
    [MenuItem("Tools/导出角色和炮弹信息", false)]
    public static void ExportInfo()
    {
        //获取所有的prefab的文件地址
        heroFilePathes = Directory.GetFiles(HERO_FILE, "*.prefab", SearchOption.AllDirectories);
        npcFilePathes = Directory.GetFiles(NPC_FILE, "*.prefab", SearchOption.AllDirectories);
        bombFilePathes = Directory.GetFiles(BOMB_FILE, "Bomb*.prefab", SearchOption.AllDirectories);
        Debug.Log("文件信息: " + "  hero: " + heroFilePathes.Length + "; npc: " + npcFilePathes.Length + "; bomb: " + bombFilePathes.Length);
        ExportLua();
        Debug.Log("炮弹和角色信息导出完成！！！");
    }

    [MenuItem("Assets/导出单个角色和炮弹信息", false)]
    public static void ExportOneInfo()
    {
        GameObject selectedObject = Selection.activeGameObject;
        string path = AssetDatabase.GetAssetPath(selectedObject);
        path = path.Substring(path.IndexOf("GameAssets"));
        builder = new StringBuilder();
        exportPrefabInfo(selectedObject, path);
        Debug.Log(path);
        
        Debug.Log("--tagGameAssets/Prefabs/Npc/enemy1/enemy1.prefab".Contains("tag" + "GameAssets/Prefabs/Npc/enemy1/enemy1.prefab"));
        var luaPath = AppConfig.Instance.assetsExportPath + "/Src/Common/Config/RoleAndBombInfo/RoleAndBombInfo.lua";
        string[] lines = File.ReadAllLines(luaPath);
        bool flag = false;
        StringBuilder newBuilder = new StringBuilder();
        foreach (string line in lines)
        {
            if (line.Contains("tag" + path))
            {
                if (flag) flag = false;
                else flag = true;
                continue;
            }
            if (!flag)
            {
                newBuilder.AppendLine(line);
            }
        }
        StringBuilder removeBuilder = new StringBuilder();
        removeBuilder.AppendLine("}");
        removeBuilder.AppendLine("return RoleAndBombInfo");
        newBuilder.Remove(newBuilder.Length - removeBuilder.Length, removeBuilder.Length);
        newBuilder.Append(builder.ToString());
        newBuilder.Append(removeBuilder.ToString());
        string dir = Path.GetDirectoryName(luaPath);
        if (!Directory.Exists(dir))
            Directory.CreateDirectory(dir);
        File.Delete(luaPath);

        using (FileStream fileStream = new FileStream(luaPath, FileMode.Create, FileAccess.Write, FileShare.ReadWrite)) // 不会锁死, 允许其它程序打开
        {
            lock (fileStream)
            {
                StreamWriter writer = new StreamWriter(fileStream);
                writer.Write(newBuilder.ToString());
                writer.Flush();
                writer.Close();
            }
        }
    }

    private static void ExportLua()
    {
        builder = new StringBuilder();
        builder.AppendLine("local RoleAndBombInfo = {");

        GeneratePrefabAndSaveInfo(heroFilePathes);
        GeneratePrefabAndSaveInfo(npcFilePathes);
        GeneratePrefabAndSaveInfo(bombFilePathes);
        builder.AppendLine("}");
        builder.AppendLine("return RoleAndBombInfo");
        var luaPath = AppConfig.Instance.assetsExportPath + "/Src/Common/Config/RoleAndBombInfo/RoleAndBombInfo.lua";
        string dir = Path.GetDirectoryName(luaPath);
        if (!Directory.Exists(dir))
            Directory.CreateDirectory(dir);
        File.Delete(luaPath);

        using (FileStream fileStream = new FileStream(luaPath, FileMode.Create, FileAccess.Write, FileShare.ReadWrite)) // 不会锁死, 允许其它程序打开
        {
            lock (fileStream)
            {
                StreamWriter writer = new StreamWriter(fileStream);
                writer.Write(builder.ToString());
                writer.Flush();
                writer.Close();
            }
        }
    }
    static void GeneratePrefabAndSaveInfo(string[] filePathes)
    {
        for (int i = 0; i < filePathes.Length; i++)
        {
            string filePath = filePathes[i];
            filePath = filePath.Substring(filePath.IndexOf("Assets"));

            GameObject _prefab = AssetDatabase.LoadAssetAtPath(filePath, typeof(GameObject)) as GameObject;
            exportPrefabInfo(_prefab, filePath);
        }
    }

    private static void exportPrefabInfo(GameObject prefab, string filePath)
    {
        string fileKey = filePath.Substring(filePath.IndexOf("GameAssets"));
        fileKey = fileKey.Replace('\\', '/');
        builder.AppendLine("--tag" + fileKey);
        builder.AppendFormat("\t[\"{0}\"] = {{\n", fileKey);
        Collider2D[] colliders = prefab.GetComponents<Collider2D>();
        builder.AppendFormat("\t\tcolliders = {{\n");
        for (int j = 0; j < colliders.Length; j++)
        {
            Collider2D collider = colliders[j];
            if (collider is BoxCollider2D)
            {
                appendBoxCollider(collider.transform, collider as BoxCollider2D, "\t\t\t", j + 1);
            }
            if (collider is CircleCollider2D)
            {
                appendCircleCollider(collider.transform, collider as CircleCollider2D, "\t\t\t", j + 1);
            }

            if (collider is PolygonCollider2D)
            {
                appendPolygonCollider(collider.transform, collider as PolygonCollider2D, "\t\t\t", j + 1);
            }
        }
        builder.AppendLine("\t\t},");
        Rigidbody2D rigidbody = prefab.GetComponent<Rigidbody2D>();
        if (rigidbody != null)
        {
            appendRigidbody(rigidbody, "\t\t");
        }
        else
        {
            Debug.Log("Rigidbody2D 为空 !!!! " + filePath);
        }
        builder.AppendLine("\t},");
        builder.AppendLine("--tag" + fileKey);
    }

    private static void appendBoxCollider(Transform root, BoxCollider2D box, string ident, int childIndex)
    {
        var offsetPos = box.transform.position - root.position;
        builder.AppendFormat("{0}[{1}] = {{\n", ident, childIndex);
        builder.AppendFormat("{0}\ttype = \"box\",\n", ident);
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "width", enlargeNumber(box.size.x));
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "height", enlargeNumber(box.size.y));
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "offsetX", enlargeNumber(box.offset.x + offsetPos.x));
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "offsetY", enlargeNumber(box.offset.y + offsetPos.y));
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "isTrigger", box.isTrigger.ToString().ToLower());
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "friction", enlargeNumber(box.friction));
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "bounciness", enlargeNumber(box.bounciness));
        builder.AppendFormat("{0}}},\n", ident);
    }

    private static void appendCircleCollider(Transform root, CircleCollider2D circle, string ident, int childIndex)
    {
        var offsetPos = circle.transform.position - root.position;
        builder.AppendFormat("{0}[{1}] = {{\n", ident, childIndex);
        builder.AppendFormat("{0}\ttype = \"circle\",\n", ident);
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "radius", enlargeNumber(circle.radius));
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "offsetX", enlargeNumber(circle.offset.x + offsetPos.x));
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "offsetY", enlargeNumber(circle.offset.y + offsetPos.y));
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "isTrigger", circle.isTrigger.ToString().ToLower());
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "friction", enlargeNumber(circle.friction));
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "bounciness", enlargeNumber(circle.bounciness));
        builder.AppendFormat("{0}}},\n", ident);
    }

    private static int enlargeNumber(float num)
    {
        return Mathf.FloorToInt(num * 10000);
    }

    private static void appendPolygonCollider(Transform root, PolygonCollider2D polygon, string ident, int childIndex)
    {
        var offsetPos = polygon.transform.position - root.position;
        builder.AppendFormat("{0}[{1}] = {{\n", ident, childIndex);
        builder.AppendFormat("{0}\ttype = \"polygon\",\n", ident);

        builder.AppendFormat("{0}\tpoints = {{\n", ident);
        var points = polygon.points;
        for (int i = 0; i < points.Length; i++)
        {
            builder.AppendFormat("{0}\t\t[{1}] = {{x = {2}, y = {3} }},\n", ident, i, enlargeNumber(points[i].x + offsetPos.x), enlargeNumber(points[i].y + offsetPos.y));
        }
        builder.AppendFormat("{0}\t}},\n", ident);

        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "isTrigger", polygon.isTrigger.ToString().ToLower());
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "friction", enlargeNumber(polygon.friction));
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "bounciness", enlargeNumber(polygon.bounciness));
        builder.AppendFormat("{0}}},\n", ident);
    }

    private static void appendRigidbody(Rigidbody2D rigidbody, string ident)
    {
        builder.AppendFormat("{0}rigidbody = {{\n", ident);
        //builder.AppendFormat("{0}\ttype = \"rigidbody\",\n", ident);
        builder.AppendFormat("{0}\t{1} = \"{2}\",\n", ident, "type", rigidbody.bodyType);
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "mass", enlargeNumber(rigidbody.mass));
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "drag", enlargeNumber(rigidbody.drag));
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "angularDrag", enlargeNumber(rigidbody.angularDrag));
        builder.AppendFormat("{0}\t{1} = {2},\n", ident, "gravityScale", enlargeNumber(rigidbody.gravityScale));
        builder.AppendFormat("{0}}},\n", ident);
    }

}

```

