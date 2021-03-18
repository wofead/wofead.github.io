# 给一个对象创建一个子对象，并带上表示

给选择的那个对象创建一个子对象，并添加精灵贴图，这里获取贴图的方式用两种，因为我们是不资源放到Editor Default Resources目录下的，所以我们可以通过`EditorGUIUtility.Load`来加载我们所需要的Texture，然后将其转换成精灵，赋值给我们新创建的对象，在这之前我们需要给对象添加一个`SpriteRenderer`组件。

我们也可以从Assets目录下开始取texture，使用`AssetDatabase.LoadAssetAtPath`来获取Texture。

这里就会有个疑问？

我们不是可以直接将Texture在Unity中切成精灵么，直接取不好么，好吧？这个问题问住我了，我也不知道怎么取呀？我在找找吧。

```c#
using UnityEngine;
using UnityEditor;
public class DrawMark
{
    [MenuItem("GameObject/MyTools/Mark", false, 50)]
    public static void CreateMark()
    {
        GameObject activeObject = Selection.activeObject as GameObject;
        string markName = activeObject.name + "Mark";
        // 创建GameObject对象
        GameObject gameObj = new GameObject(markName);
        gameObj.transform.SetParent(activeObject.transform);
        // 获取SpriteRenderer对象
        SpriteRenderer spr = gameObj.AddComponent(typeof(SpriteRenderer)) as SpriteRenderer;
        Texture2D texture2D = AssetDatabase.LoadAssetAtPath("Assets/Editor Default Resources/Test/CirleMark.png", typeof(Texture2D)) as Texture2D;
        // 添加图片
        //Texture2D texture2D = EditorGUIUtility.Load("Test/CirleMark.png") as Texture2D;
        spr.sprite = Sprite.Create(texture2D, new Rect(0, 0, texture2D.width, texture2D.height), Vector2.zero);
        //spr.sprite = sprites[1];
        // 移动位置
        spr.transform.position = activeObject.transform.position;
    }

}


```

我又回来了，上面的问题我找了下，也有方法，三种,开心，首先使用`EditorGUIUtility.Load`这种方法我没有找到，还是太菜了，需要研究一哈。

1. 通过`Resources.LoadAll<Sprite>(path)`,这个path不带后缀名，资源必须在Resources目录下，比较快
2. `AssetDatabase.LoadAllAssetsAtPath(path).OfType().ToArray()`，path带后缀名，比较慢
3. 就是有个public sprite，直接在编辑器里面设置了，这个是最蠢的办法

```c#
using UnityEngine;
using UnityEditor;
using System.Linq;

public class DrawMark
{
    [MenuItem("GameObject/MyTools/Mark", false, 50)]
    public static void CreateMark()
    {
        GameObject activeObject = Selection.activeObject as GameObject;
        string markName = activeObject.name + "Mark";
        // 创建GameObject对象
        GameObject gameObj = new GameObject(markName);
        gameObj.transform.SetParent(activeObject.transform);
        // 获取SpriteRenderer对象
        SpriteRenderer spr = gameObj.AddComponent(typeof(SpriteRenderer)) as SpriteRenderer;
        Sprite[] sprites = AssetDatabase.LoadAllAssetsAtPath("Assets/Editor Default Resources/Test/CirleMark.png").OfType<Sprite>().ToArray();
        // 添加图片
        spr.sprite = sprites[1];
        // 移动位置
        spr.transform.position = activeObject.transform.position;
    }

}

```

