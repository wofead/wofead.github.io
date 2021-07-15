# HideInTables

> *Hide In Tables Attribute特性：用于TableList特性中隐藏对应的属性绘制的Inline*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-608-5fb7da3a97399.png)

```cs
using Sirenix.OdinInspector;
using System;
using System.Collections.Generic;
using UnityEngine;

public class HideInTablesAttributeExample : MonoBehaviour
{
    [PropertySpace(0,40)]
    public MyItem Item = new MyItem();

    [TableList]//以表格形式展示List中的成员
    public List TableItemList = new List()
     {
    new MyItem(),
    new MyItem(),
    new MyItem(),
     };

    [Serializable]
    public class MyItem
    {
        public string A;

        public int B;

        [HideInTables]//此字段在表格中不显示
        public int Hidden;
    }
}
```