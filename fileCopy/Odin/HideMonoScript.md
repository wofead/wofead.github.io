# HideMonoScript

> Hide Mono Script Attribute特性

![img](https://aihailan.com/wp-content/uploads/2020/11/post-612-5fb7da593add4.gif)

```cs
[CreateAssetMenu(fileName = "HideMonoScript_ScriptableObject", menuName = "CreatScriptableObject/HideMonoScript")]
[HideMonoScript]
public class HideMonoScript : ScriptableObject
{
    public string Value;
}
using System.Collections.Generic;
using UnityEngine;

[CreateAssetMenu(fileName = "ShowMonoScript_ScriptableObject", menuName = "CreatScriptableObject/ShowMonoScript")]
public class ShowMonoScript : ScriptableObject
{
    public string Value;
}
```