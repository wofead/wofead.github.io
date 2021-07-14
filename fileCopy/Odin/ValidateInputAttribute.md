# ValidateInputeAttribute

> *Validate Input Attribute特性：用于任何属性，并允自定义检查器，灵活实现多种监测规则。使用此选项可强制执行正确的值（提供对应的返回值）。*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-708-5fb7dd567a99e.png)

##### 常规写法，实参输入一个方法的名称，一个对应的消息

![img](https://aihailan.com/wp-content/uploads/2020/11/post-708-5fb7dd56e9cee.png)

```cs
    [ValidateInput("MustBeNull", "这个字段应该为空。")]
    public MyScripty DefaultMessage;
    private bool MustBeNull(MyScripty scripty)
    {
        return scripty == null;
    }
```

##### 也可以使用$特殊标识符引用一个字段动态显示提示信息，而且也可以明确指出需要提示信息的类型

![img](https://aihailan.com/wp-content/uploads/2020/11/post-708-5fb7dd5792d38.png)

```cs
    [ReadOnly]
    public string dynamicMessage = "这个物体不应该为空！";
    [ValidateInput("CheckGameObject", "$dynamicMessage", InfoMessageType.None)]
    public GameObject targetObj_None = null;
    [ValidateInput("CheckGameObject", "$dynamicMessage", InfoMessageType.Info)]
    public GameObject targetObj_Info = null;
    [ValidateInput("CheckGameObject", "$dynamicMessage", InfoMessageType.Warning)]
    public GameObject targetObj_Warning = null;
    [ValidateInput("CheckGameObject", "$dynamicMessage", InfoMessageType.Error)]
    public GameObject targetObj_Error = null;

    private bool CheckGameObject(GameObject tempObj)
    {
        return tempObj != null;
    }
```

*当然也可以这么用*

![img](https://aihailan.com/wp-content/uploads/2020/11/post-708-5fb7dd5822d81.gif)

```cs
    [ValidateInput("IfNullIsFalse", "$Message", InfoMessageType.Warning)]
    public string Message = "Dynamic ValidateInput message";

    private bool IfNullIsFalse(string value)
    {
        return string.IsNullOrEmpty(value);
    }
```

##### 也可以覆盖默认的提示消息和消息类型

![img](https://aihailan.com/wp-content/uploads/2020/11/post-708-5fb7dd58e7105.gif)

```cs
 [ValidateInput("HasMeshRendererDynamicMessage", "对应的函数中已经有消息，所以这个默认消息已经没用")]
    public GameObject DynamicMessage;
    private bool HasMeshRendererDynamicMessage(GameObject gameObject, ref string errorMessage)
    {
        if (gameObject == null) return true;

        if (gameObject.GetComponentInChildren() == null)
        {
            errorMessage = "\"" + gameObject.name + "\" 这玩应必须有一个 MeshRenderer 组件";//如果设置消息，则默认消息会被覆盖
            return false;
        }
        return true;
    }

    [ValidateInput("HasMeshRendererDynamicMessageAndType", "对应的函数中已经有消息和类型，所以这个默认消息和类型已经没用")]
    public GameObject DynamicMessageAndType;

    [InfoBox("Change GameObject value to update message type", InfoMessageType.Info)]
    public InfoMessageType MessageType;
    private bool HasMeshRendererDynamicMessageAndType(GameObject gameObject, ref string errorMessage, ref InfoMessageType? messageType)
    {
        if (gameObject == null) return true;

        if (gameObject.GetComponentInChildren() == null)
        {
            errorMessage = "\"" + gameObject.name + "\" 要有一个 MeshRenderer 组件";//如果设置消息，则默认消息会被覆盖
            messageType = this.MessageType;//如果设置消息类型，则默认消息类型会被覆盖
            return false;
        }
        return true;
    }
```

```c#
using Sirenix.OdinInspector;
using UnityEngine;

public class ValidateInputAttributeExample : MonoBehaviour
{

    [ValidateInput("MustBeNull", "这个字段应该为空。")]
    public MyScripty DefaultMessage;
    private bool MustBeNull(MyScripty scripty)
    {
        return scripty == null;
    }

    [ReadOnly]
    public string dynamicMessage = "这个物体不应该为空！";
    [ValidateInput("CheckGameObject", "$dynamicMessage", InfoMessageType.None)]
    public GameObject targetObj_None = null;
    [ValidateInput("CheckGameObject", "$dynamicMessage", InfoMessageType.Info)]
    public GameObject targetObj_Info = null;
    [ValidateInput("CheckGameObject", "$dynamicMessage", InfoMessageType.Warning)]
    public GameObject targetObj_Warning = null;
    [ValidateInput("CheckGameObject", "$dynamicMessage", InfoMessageType.Error)]
    public GameObject targetObj_Error = null;

    private bool CheckGameObject(GameObject tempObj)
    {
        return tempObj != null;
    }

    [ValidateInput("IfNullIsFalse", "$Message", InfoMessageType.Warning)]
    public string Message = "Dynamic ValidateInput message";

    private bool IfNullIsFalse(string value)
    {
        return string.IsNullOrEmpty(value);
    }

    [ValidateInput("HasMeshRendererDynamicMessage", "对应的函数中已经有消息，所以这个默认消息已经没用")]
    public GameObject DynamicMessage;
    private bool HasMeshRendererDynamicMessage(GameObject gameObject, ref string errorMessage)
    {
        if (gameObject == null) return true;

        if (gameObject.GetComponentInChildren() == null)
        {
            errorMessage = "\"" + gameObject.name + "\" 这玩应必须有一个 MeshRenderer 组件";//如果设置消息，则默认消息会被覆盖
            return false;
        }
        return true;
    }

    [ValidateInput("HasMeshRendererDynamicMessageAndType", "对应的函数中已经有消息和类型，所以这个默认消息和类型已经没用")]
    public GameObject DynamicMessageAndType;

    [InfoBox("Change GameObject value to update message type", InfoMessageType.Info)]
    public InfoMessageType MessageType;
    private bool HasMeshRendererDynamicMessageAndType(GameObject gameObject, ref string errorMessage, ref InfoMessageType? messageType)
    {
        if (gameObject == null) return true;

        if (gameObject.GetComponentInChildren() == null)
        {
            errorMessage = "\"" + gameObject.name + "\" 要有一个 MeshRenderer 组件";//如果设置消息，则默认消息会被覆盖
            messageType = this.MessageType;//如果设置消息类型，则默认消息类型会被覆盖
            return false;
        }
        return true;
    }

    private bool HasMeshRendererDefaultMessage(GameObject gameObject)
    {
        if (gameObject == null) return true;
        return gameObject.GetComponentInChildren() != null;
    }
}

```

