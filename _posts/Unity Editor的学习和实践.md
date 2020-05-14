# 像素的学习

**图像的尺寸：**表示图像的宽度和高度以像素（或 cm、mm、inch 等）为单位，表示在图像横边和竖边上各有多少个像素

**图像的分辨率：**表示单位面积内的像素数量，即像素密度。单位通常是 `dpi`，即像素点与长度单位英寸的比值。

**像素转换成毫米：**需要知道 `DPI(dots per inch)` 参数，即每英寸多少点。另外，一英寸 = `25.4 mm`

## 像素的理解

像素是组成图象的最基本单元要素：点。辨率是指在长和宽的两个方向上各拥有的像素个数。

而我们所熟悉的屏幕分辨率就是由宽和高上的点的个数来表示的。显然单位面积上像素点越多即像素点越小，这图片就越清晰细腻。

举个例子：我们以一款手机为例来说明这个问题。其主屏尺寸：4寸，主屏分辨率：800x480像素，通过勾股定理计算可知其长宽为3.430寸X2.058寸（87.1毫米X52.3毫米）。800/3.430=233，即每英寸长度有233个像素，每一个像素有87.1/800=0.109毫米大。

这个每英寸长度上的像素数个数叫做**影像分辨率**，简称PPI（pixeleperinch英文缩写）。如每英寸长度上有82个像素点，即用82PPI来表示。

像素组成的图像叫位图或者光栅图像，点阵图，像素图形，网格图。（光栅一词源于模拟电视技术，我们的电视信号就是模拟信号。

在一般情况下，像素它是一块正方形，带有高度、色调、色相、色温、灰度等的颜色信息，一定数量的颜色有别的正方形小块排列组合，用以表示一幅点阵图像，也就是位图图像。通过数码相机拍摄、扫描仪扫描或位图软件输出的图像都是位图。

当图片的分辨率大于显示屏的分辨率时，显示屏会把图片按比例相对的宿小。相当于把图片的两个或多个像素在显示屏上以一个像素显示出来。所以我们的图片分辨率越大，看到的图片就越清晰细腻逼真。所以我们照相的时候，一定要用最大的分辨率来照相。到数码冲印店打印出来的图象也是比低像素的清晰明了呀！分辨率太低，有时想用来做墙纸都不行。

打印精度，就是每英寸长度方向上打印机打印的像素点数dpi，一般都是300dpi为标准。120dpi是最低要求,150dpi是安全达标下限。一般来说有250dpi就行了。所以说一般都是用250dpi来计算打印相片所要求图片的最低分辨率。如果相片的分辨率的dpi放大很大的图片上就会导致模糊。



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

