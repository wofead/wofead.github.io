# 配置.asset文件

创建的ScriptableObject类文件，他不属于编辑器脚本，运行时也可以用，相当于配置文件：

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
 
public class Setting : ScriptableObject
{
    public string serverIp = "127.0.0.1";
    public string serverPort = "5566";
    public bool isDebug = true;
}
```

创建asset配置文件需要注意：

生成Asset文件的路径需要使用相对路径，绝对路径无效，从Assets路径开始的相对路径，CreateAssetFile类要放在Editor文件下，属于编辑器脚本：

```c#
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;
using System.IO;
 
public class CreateAssetFile : UnityEditor.Editor {
 
 
    const string assetsFolderName = "AssetFiles";
 
    [MenuItem("Editor/CreateSetingAsset")]
    static void CreateAsset()
    {
        // 实例化类
        Setting asset = ScriptableObject.CreateInstance<Setting>();
 
        // 如果实例化 Bullet 类为空，返回
        if (!asset)
        {
            Debug.LogWarning("Bullet not found");
            return;
        }
        // 自定义资源保存路径
        string path = Application.dataPath + "/"+ assetsFolderName +"/";
 
        // 如果项目总不包含该路径，创建一个
        if (!Directory.Exists(path))
        {
            Directory.CreateDirectory(path);
        }
        //将类名 Setting 转换为字符串
        //拼接保存自定义资源（.asset） 路径
        path = string.Format("Assets/"+assetsFolderName+"/{0}.asset", (typeof(Setting).ToString()));
        // 生成自定义资源到指定路径
        AssetDatabase.CreateAsset(asset, path);
        AssetDatabase.SaveAssets();
        AssetDatabase.Refresh();
    }
}
```

读取asset配置文件：

```c#
//第一种：通过AssetDatabase.LoadAssetAtPath加载
Setting assetFromEditor =   AssetDatabase.LoadAssetAtPath<Setting>("Assets/AssetFiles/Setting.asset");
Debug.Log(assetFromEditor.serverIp);
 
 
//第二种：通过Resources.Load加载
Setting assetFromResources = Resources.Load<Setting>("Setting");
Debug.Log(assetFromResources.serverPort);
```



---

在游戏开发中，经常会用到一些配置文件保存一些数据，然后项目运行中读取这些配置文件中的数据在游戏中使用。

如：配置血条：根据角色类型（人物、动物、怪物等）配置不同的血条，包括血条大小，血条名或血条预设，血条颜色等一些简单数据。

如：配置子弹：子弹类型（真子弹、假子弹、追踪子弹等），子弹速度，伤害数值，子弹关联的特效等。

诸如此类的配置很多种，可创建一个可序列化的类存储数据，或者创建 XML 、JSON 文件保存数据，创建 Excel 文件，创建 TXT 文件，皆可完成需求，灵活使用这些方法保存配置数据。

在此介绍一下使用可序列化类保存配置，并且将可序列化类保存成Unity的自定义文件（.asset）,然后配置自定义文件（.asset）。

**优点：**

可以保存数据类型多样（int、string、Vector3、GameObject、Transform、Texture等）如关联预设，关联图片等资源数据，而XML、TXT等只能保存（int、string、Vector3 等基本数据类型）。



**缺点：**

如果配置数据中保存了（GameObject、Texture）等资源数据，当关联的资源被删除时，配置数据将丢失，需要重新将新的资源再次关联到配置数据上。

生成asset文件：

```c#
[Serializable]
public class Bullet : ScriptableObject { // Bullet 类直接继承自 ScriptableObject  
 
}
 
 
public class CreateAsset : Editor {
 
    // 在菜单栏创建功能项
    [MenuItem("CreateAsset/Asset")]
    static void Create()
    {
        // 实例化类  Bullet
        ScriptableObject bullet = ScriptableObject.CreateInstance<Bullet>();
        // 如果实例化 Bullet 类为空，返回
        if (!bullet)
        {
            Debug.LogWarning("Bullet not found");
            return;
        }
        // 自定义资源保存路径
        string path = Application.dataPath + "/BulletAeeet";
        // 如果项目总不包含该路径，创建一个
        if (!Directory.Exists(path))
        {
            Directory.CreateDirectory(path);
        }
 
        //将类名 Bullet 转换为字符串
        //拼接保存自定义资源（.asset） 路径
        path = string.Format("Assets/BulletAeeet/{0}.asset", (typeof(Bullet).ToString()));
        // 生成自定义资源到指定路径
        AssetDatabase.CreateAsset(bullet, path);
    }
}
```



创建一个继承于ScriptableObject 类的对象

```cs

[System.Serializable]
public class DemoConfig : ScriptableObject {
 
    public string userName;
 
    public int userID;
 
    public GameObject userObject;
 
    public Sprite sprite;
 
    public AudioClip audioClip;
 
    public void print() {
        Debug.Log(string.Format("name: {0} id: {1}" , userName , userID));
    }
}
```

使用UConfig创建对应的Asset文件


配置各项属性 

创建Asset文件

创建Asset文件的核心代码只有一句, 那就是通过ScriptableObject.CreateInstance()方法, 这个方法允许传入一个类名并创建对应的资源配置文件,以下代码如下.

```c#

static void CreateAsset()
        {
            //class_Name表示类名 , config_Name表示要创建的配置文件名称 , file_floder表示创建路径
            if (class_Name.Length == 0 || config_Name.Length == 0 || file_floder.Length == 0)
            {
                Debug.LogError(string.Format("[UConfig]: 创建失败 , 信息不完整!"));
                return;
            }
            ScriptableObject config = ScriptableObject.CreateInstance(class_Name);
 
            if (config == null)
            {
                Debug.LogError(string.Format("[UConfig]: 创建失败 , 类名无法识别! --> {0}", class_Name));
                return;
            }
            // 自定义资源保存路径
            string path = file_floder;
            //如果项目总不包含该路径，创建一个
            if (!Directory.Exists(path))
            {
                Directory.CreateDirectory(path);
            }
            config_Name = config_Name.Replace(".asset", "");
            path = string.Format("{0}/{1}.asset", file_floder, config_Name);
            string defFilePath = "Assets/" + config_Name + ".asset";
            // 生成自定义资源到指定路径
            AssetDatabase.CreateAsset(config, defFilePath);
            File.Move(Application.dataPath + string.Format("/{0}.asset" , config_Name), path);
            AssetDatabase.Refresh();
            Debug.Log(string.Format("<color=yellow>[UConfig]: 创建成功 ! --> {0}</color>", path));
            configWindow.Close();
        }
```

读取配置好的Asset文件

通过以上方式创建的配置文件可以作为一个与Prefab资源对待, 并且可以随其他prefab资源一样被打成AssetBundle包动态加载 , 理所当然的可以实现所谓的热更配置表.

加载的代码就用 Resources.Load(“fileName”); 就可以了,也可以预先挂着到某一个代码的属性中,然后直接通过as强转.