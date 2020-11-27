# Unity Navigation

我们可以通过使用Unity的Navigation系统来构造简单的AI自动寻路功能，这里简单的介绍一下。

[toc]

## 构建导航网格

将需要加入到导航网格的物体选中，然后勾选位于Inspector中Static右侧向下箭头里面的Navigation Static，这样我们就可以看到在Inspector旁边有Navigation组件了。

在Bake中调整代理尺寸，然后点击Bake按钮创建网格。

## 创建导航代理

接下来创建一个capsule用来作为玩家，给它添加一个`Nav Mesh Agent`组件，这个就是可以导航移动的角色，然后创建一个球体作为我们需要导航到那里的目标。给我们的Player添加导航脚本。

```c#
using UnityEngine;
using UnityEngine.AI;

public class MoveTo : MonoBehaviour
{
    public Transform goal;
    // Start is called before the first frame update
    void Start()
    {
        NavMeshAgent agent = GetComponent<NavMeshAgent>();
        agent.destination = goal.position;
    }
}
```

然后运行，你就会发下我们的Player会自动移动到球体那里。

## 创建导航障碍物

我们添加一个箱子放到刚才Player到球体路径的中间，给这个箱子添加`Nav Mesh Obstacle`组件，并且勾选Carve。

然后再次运行，你就会法发现，球体会自动避开这个箱子。

## 创建网格链接

网格链接用于创建跨越可行走的导航网格表面的路径。

我们在我们的Cube上添加`Off Mesh Link`组件，然后在这个组件上添加链接的两个物体，然后别忘记设置Drop Height和Jump Distance，之后Bake。

## 导航区与移动成本

我们可以在Area中，添加新的导航区域，然后给这个物体设置它属于那个导航区域，导航区域可以设置移动成本，通过设置移动成本，导航的时候回考虑到在这个区域移动值不值得，不值得的话会绕路。而不是直线穿过。