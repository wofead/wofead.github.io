# Odin Validator

> *在项目中，我们总会在组件或者***[Scriptable Objects](https://www.jianshu.com/p/be864f9cc3eb)***中填写一些我们需要的字段，但是随着项目进度的不断进展，当时可能临时填写、不符合规则的字段会被遗弃在角落。这就成了一个定时炸弹，可能成为你在发布时寻找不可复现BUG，通宵加班的主要原因。*
> *而Odin验证器可以很大限度的解决这个问题，他可以批量的检查在项目中，按照你的指定的标记、指定的规则，批量的检查字段，让不符合规则的成员变量无所遁形。*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-916-5fb80d93a43f6.png)

##### 关于前言所说指定的标记，Odin验证器支持如下内置验证

- **[Require Component](https://aihailan.com/odin-inspector-系列教程-验证器入门指南【何为验证器validator】-2/[https://www.jianshu.com/p/41399298f992](https://www.jianshu.com/p/41399298f992))【Unity原生】**
- **[Assets Only](https://www.jianshu.com/p/194081e24e65)**
- **[File Path](https://www.jianshu.com/p/ea5cbcbaa047)【当RequireExistingPath字段设置为True时】**
- **[Folder Path](https://www.jianshu.com/p/1587dbd2f992)【当RequireExistingPath字段设置为True时】**
- **[Child Game Objects](https://www.jianshu.com/p/a5c84a257ac5)**
- **[Scene Objects Only](https://www.jianshu.com/p/d588a2a5e5a1)**
- **[Detailed Info Box](https://www.jianshu.com/p/ee83f0c151e9)【显示为警告或错误时】**
- **[Info Box](https://www.jianshu.com/p/b205e52c70ea)【显示为警告或错误时】**
- **[Required](https://www.jianshu.com/p/c2590cf8ab89)**
- **[Validate Input](https://www.jianshu.com/p/5cc8e00a6fd8)**
- **[Min Value](https://www.jianshu.com/p/9c7d1f750dac)**
- **[Max Value](https://www.jianshu.com/p/7fcf91a144bc)**
- **[Range](https://www.jianshu.com/p/108d4a85ed67)和[Property Range](https://www.jianshu.com/p/dfe24dcaeaa0)**
- **[Min Max Slider](https://www.jianshu.com/p/e9a72fcd13dc)**

## 对于标记的字段如何批量验证？

![img](https://aihailan.com/wp-content/uploads/2020/11/post-916-5fb80d93ccfb7.gif)

1. **选择 Tools->Odin Project Validator打开验证器**
2. **选择扫描规则文件Scan Entire Project（扫描整个工程），关于扫描规则会在后续的文章介绍**
3. **点击绿色按钮Run Scan Entire Project执行扫描**

## 扫描后验证器面板会有三部分信息显示

#### 1. 扫描列表

> 他会显示所有不符合规则的资源对应的节点，并显示有对应警告或者错误的数量

![img](https://aihailan.com/wp-content/uploads/2020/11/post-916-5fb80d9419f85.png)

#### 2. 对应的检查器面板，点击对应的Ping Oject对应的资源会高亮显示，点击Select Object会在Inspector中显示资源信息

![img](https://aihailan.com/wp-content/uploads/2020/11/post-916-5fb80d9462213.gif)

#### 3.错误或者警告信息列表

> 会显示所有检查出来的错误或警告信息，并提供所搜、分类、排序功能

![img](https://aihailan.com/wp-content/uploads/2020/11/post-916-5fb80d94b52b8.gif)

## 配置Validator

![img](https://aihailan.com/wp-content/uploads/2020/11/post-519-5fb7d5902203f.png)

> 在第一次打开验证器的时候会看到初始的五个配置,选择对应的扫描文件进行扫描，就可以对标记的字段进行验证了
>
> - **Scan Entire Project**
> - **Scan All Assets**
> - **Scan All Scenes**
> - **Scan Open Scenes**
> - **Scan Scenes From Build Options**
>
> 设置对应配置文件的参数可以从两个地方进入，一个是点击对应配置文件后方的笔图标，另一个点击配置文件进入扫描界面操作（笔者更喜欢这个）

![img](https://aihailan.com/wp-content/uploads/2020/11/post-519-5fb7d5909a755.gif)

> 运行多个配置文件扫描时，会在对应配置文件的下方分别显示扫描结果

![img](https://aihailan.com/wp-content/uploads/2020/11/post-519-5fb7d591754e0.gif)

### Scan Entire Project

> 扫描整个工程，包含**Scan All Assets**和**Scan All Scenes**，也就是说，使用Scan Entire Project会按照**Scan All Assets**和**Scan All Scenes**的规则进行扫描

### Scan All Assets

> 扫描所有Asset文件（Project中的文件），点击配置文件会在右侧数显设置对应参数的面板

![img](https://aihailan.com/wp-content/uploads/2020/11/post-519-5fb7d5922ad5d.png)

- **Name** 配置文件的名称
- **Description** 配置文件的描述
- **Search Filters**扫描文件的类型，可以参考Project中的设置，但基本上也就是**t:Prefab**与**t:ScriptableObject**
  ![img](https://aihailan.com/wp-content/uploads/2020/11/post-519-5fb7d5928a6d6.png)
- **Asset Path** 扫描文件的路径
- **Asset References**需要指定的扫描文件，也就是可以指定扫描不在Asset Path中的资源
- **Exclude Asset Paths** 不需要扫描的目录
- **Exclude Asset References** 指定不需要扫描的文件

### Scan All Scenes

> 扫描所有场景，这个配置文件中出现了3个新的设置

- **Include Scenes From Build Options** 扫描所有在Build中添加的场景
  ![img](https://aihailan.com/wp-content/uploads/2020/11/post-519-5fb7d59332b18.png)
- **Include OpenFrom Scenes** 扫描正在使用的场景，也就是在Hierarchy中
- **Include Asset Dependencies** 扫描场景中所依赖的资源

### **Scan Open Scenes**

> 仅仅扫描正在使用的场景，所以在选项中只勾选了**Include Open Scenes**

![img](https://aihailan.com/wp-content/uploads/2020/11/post-519-5fb7d593dccb5.png)

### Scan Scenes From Build Options

> 扫描在Build中添加和场景中依赖的资源

![img](https://aihailan.com/wp-content/uploads/2020/11/post-519-5fb7d594601a1.png)

## Automate Validation

![img](https://aihailan.com/wp-content/uploads/2020/11/post-519-5fb7d594ee60d.png)

> 自动验证包含三个分类，需要时勾选对应的自动验证模式即可

- **On Play** 运行时自动验证
- **On Build** 编译出包时自动验证
- **On Project StartUp** 打开Unity工程时自动验证

> 他们设置几乎一致，所以笔者只介绍一类对应的设置即可

- **Finish Validation On Failures** 默认不勾选（推荐）如果不勾选，当遇到任何需要处理的事件时就会执行指定的Actions。勾选会在完成整个验证时再执行Actions

![img](https://aihailan.com/wp-content/uploads/2020/11/post-519-5fb7d59589d70.gif)

- Actions

   

  当遇到错误或警告信息时执行对应的操作

  - **OpenValidatorIfError** 当遇到错误信息时打开验证器面板
  - **OpenValidatorIfWarning** 当遇到警告信息时打开验证器面板
  - **StopHookEventOnError** 当遇到错误信息时拦截运行（停止运行或构建）
  - **StopHookEventOnWarning** 当遇到警告信息时拦截运行（停止运行或构建）
  - **LogError** 打印错误信息
  - **LogWarning** 打印警告信息

![img](https://aihailan.com/wp-content/uploads/2020/11/post-519-5fb7d5961e3f8.png)

- **Profiles To Run** 需要运行的配置文件
  ![img](https://aihailan.com/wp-content/uploads/2020/11/post-519-5fb7d59692d44.png)

## 重置配置文件

> 频繁设置导致设置文件出错，无法运行？不怕，一键恢复默认设置，点击 **Manage Profiles>ReseDefault Profiles** 即可

![img](https://aihailan.com/wp-content/uploads/2020/11/post-519-5fb7d59720279.png)
![img](https://aihailan.com/wp-content/uploads/2020/11/post-519-5fb7d5978850e.png)

## 自定义验证

> 准备工作，创一个自定义类挂在对应物体上

```cs
using Sirenix.OdinInspector;
using UnityEngine;

public class TestCustomComponent : MonoBehaviour
{
    [Required("需要一个Obj", MessageType = InfoMessageType.Warning)]
    public GameObject tempObj;

    public enum CustomType
    {
        Node,One,Two
    }
    [TestValidatorAttribute]
    public CustomType customType = CustomType.Node;

}
```

## 自定义全局类型验证

> 全局类型验证：就是不需要向元素添加特性，例如:**[Required Attribute](https://www.jianshu.com/p/c2590cf8ab89)**，在项目中对所有指定类型按照我们想要的规则进行验证，如果不符合规则会弹出我们定义的错误或警告信息

#### 示例展示

> 你只需要编写验证代码、执行验证扫描即可

![img](https://aihailan.com/wp-content/uploads/2020/11/post-525-5fb7d5ee4dfac.gif)

#### 示例代码

```cs
using Sirenix.OdinInspector.Editor.Validation;
[assembly: RegisterValidator(typeof(CustomTypeValidator))]

public class CustomTypeValidator : ValueValidator
{
    protected override void Validate(TestCustomComponent.CustomType value, ValidationResult result)
    {
        if (value== TestCustomComponent.CustomType.Node)
        {
            result.ResultType = ValidationResultType.Warning;
            result.Message = "需要对CustomType使用除None以外的任何值";
        }
    }
}
```

## 自定义特性验证

> 其实就是我们自定义类似[Required Attribute](https://www.jianshu.com/p/c2590cf8ab89)的特性

#### 编写自定义验证特性和对应的验证规则

```cs
using Sirenix.OdinInspector.Editor.Validation;
using System;
using System.Reflection;

[assembly: RegisterValidator(typeof(CustomeAttributeValidator))]
public class TestValidatorAttribute : Attribute { }

/// 
/// 对此自定义特性的检测机制
/// 

public class CustomeAttributeValidator : AttributeValidator
{
    protected override void Validate(object parentInstance, TestCustomComponent.CustomType memberValue, MemberInfo member, ValidationResult result)
    {
        try
        {
            //Debug.Log($"parentInstance:{parentInstance}---memberValue:{memberValue}---member:{member}---result:{result}");
            if (memberValue == TestCustomComponent.CustomType.One)
            {
                throw new ArgumentException($"不能使用{memberValue}");
            }
        }
        catch (ArgumentException ex)
        {
            result.ResultType = ValidationResultType.Error;
            result.Message = "错误或无效的CustomType值: " + ex.Message;
        }
    }
}
```

#### 将特性添加到对应的字段上

```cs
    [TestValidatorAttribute]
    public CustomType customType = CustomType.Node;
```

#### 一键运行验证扫描

![img](https://aihailan.com/wp-content/uploads/2020/11/post-525-5fb7d5eea8b7a.gif)