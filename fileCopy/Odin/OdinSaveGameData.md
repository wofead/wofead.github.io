# Odin Save Game Data

> 这次笔者介绍Odin-Serializer来进行游戏中的数据序列化与反序列化，做了一个类似PlayerPrefs的简单示例方便大家参考

> 虽然Odin-Serializer也可进行Unity Object引用的先关序列化，不过建议使用SerializedMonoBehaviour的方式，更简单（**[详情见：Odin Inspector 系列教程 — 初识Odin序列化](https://www.jianshu.com/p/15dae2b764fe)**）
> Odin-Serializer的另一个优势就是速度快且GC小(序列化格式为二进制时)，所以当非测试时，强烈建议把序列化格式更改为二进制，因为使用Json格式会慢很多。

##### 相关数据如下

![img](https://aihailan.com/wp-content/uploads/2020/11/post-517-5fb7d5683da19.png)

![img](https://aihailan.com/wp-content/uploads/2020/11/post-517-5fb7d56898d11.png)

##### 示例代码

```c#
using Sirenix.Serialization;
using System;
using System.Collections.Generic;
using System.IO;
using UnityEngine;

public sealed class OdinPlayerPrefs
{

    #region Singleton
    public const string Name = "OdinPlayerPrefs";
    static OdinPlayerPrefs()
    {
        absoluteDirectoryPath = Path.Combine(Application.persistentDataPath, "User");
        if (!Directory.Exists(absoluteDirectoryPath))
        {
            Directory.CreateDirectory(absoluteDirectoryPath);
        }
        fileFullName = Path.Combine(absoluteDirectoryPath, fileName);
        LoadData();
    }
    public OdinPlayerPrefs() { }

    #endregion

    public static string absoluteDirectoryPath;
    public const string fileName = "UserConfig";
    public static string fileFullName;

    public static DataFormat dataFormat = DataFormat.Binary;
    private static UserInfo userInfo;

    private const string defaultString = "";
    private const float defaultFloat = 0;
    private const int defaultInt = 0;

    [Serializable]
    public class UserInfo
    {
        public Dictionary keyValuePairs_String = new Dictionary();
        public Dictionary keyValuePairs_Float = new Dictionary();
        public Dictionary keyValuePairs_Int = new Dictionary();
    }

    private static void SaveData()
    {
        byte[] bytes = SerializationUtility.SerializeValue(userInfo, dataFormat);
        File.WriteAllBytes(fileFullName, bytes);
    }
    private static void LoadData()
    {
        if (!File.Exists(fileFullName))
        {
            userInfo = new UserInfo();
            return;
        }
        byte[] bytes = File.ReadAllBytes(fileFullName);
        userInfo = SerializationUtility.DeserializeValue(bytes, dataFormat);
    }

    public static void DeleteAll()
    {
        userInfo.keyValuePairs_String.Clear();
        userInfo.keyValuePairs_Float.Clear();
        userInfo.keyValuePairs_Int.Clear();
        SaveData();
    }
    public static void DeleteKey(string key)
    {
        bool isNeedSaveData = false;
        if (userInfo.keyValuePairs_String.ContainsKey(key))
        {
            userInfo.keyValuePairs_String.Remove(key);
            isNeedSaveData = true;
        }
        if (userInfo.keyValuePairs_Float.ContainsKey(key))
        {
            userInfo.keyValuePairs_Float.Remove(key);
            isNeedSaveData = true;
        }
        if (userInfo.keyValuePairs_Int.ContainsKey(key))
        {
            userInfo.keyValuePairs_Int.Remove(key);
            isNeedSaveData = true;
        }
        if (isNeedSaveData)
        {
            SaveData();
        }
        else
        {
            Debug.LogWarning($"删除失败，没有找到指定Key:{key}");
        }
    }
    public static float GetFloat(string key)
    {
        return GetFloat(key, defaultFloat);
    }
    public static float GetFloat(string key, float defaultValue)
    {
        if (userInfo.keyValuePairs_Float.ContainsKey(key))
        {
            return userInfo.keyValuePairs_Float[key];
        }
        else
        {
            return defaultValue;
        }
    }
    public static int GetInt(string key)
    {
        return GetInt(key, defaultInt);
    }
    public static int GetInt(string key, int defaultValue)
    {
        if (userInfo.keyValuePairs_Int.ContainsKey(key))
        {
            return userInfo.keyValuePairs_Int[key];
        }
        else
        {
            return defaultValue;
        }
    }
    public static string GetString(string key)
    {
        return GetString(key, defaultString);
    }
    public static string GetString(string key, string defaultValue)
    {
        if (userInfo.keyValuePairs_String.ContainsKey(key))
        {
            return userInfo.keyValuePairs_String[key];
        }
        else
        {
            return defaultValue;
        }
    }
    public static bool HasKey(string key)
    {
        if (userInfo.keyValuePairs_String.ContainsKey(key))
        {
            return true;
        }
        if (userInfo.keyValuePairs_Float.ContainsKey(key))
        {
            return true;
        }
        if (userInfo.keyValuePairs_Int.ContainsKey(key))
        {
            return true;
        }
        return false;
    }
    public static void SetFloat(string key, float value)
    {
        userInfo.keyValuePairs_Float[key] = value;
        SaveData();
    }
    public static void SetInt(string key, int value)
    {
        userInfo.keyValuePairs_Int[key] = value;
        SaveData();
    }
    public static void SetString(string key, string value)
    {
        userInfo.keyValuePairs_String[key] = value;
        SaveData();
    }
}
```