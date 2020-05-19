# unity的坐标系统

几经折磨，还是痛定思痛，决定好好的研究一哈游戏中的坐标系统，先从unity开始，可能很多，也可能不多，但是研究了你才会了解究竟是什么样子的。

[toc]

## 坐标系分类



我们的坐标系分为右手坐标系和左手坐标系，我是这么区分的，拿出右手，大拇指指向z轴正方向，手心法线方向和X正方向一致，握手指，手指指向y轴正方向为右手坐标系，相反，y轴负方向为左手坐标系。

## 坐标体系

Unity3D 当中基本的坐标体系主要有下面这四种：

1. 世界坐标系(World Space)
2. 屏幕坐标系(Screen Space)
3. 视口坐标系(Viewport Space)
4. GUI界面坐标系(GUI System)

![](http://liuqingwen.me/blog/2017/07/31/understanding-coordinate-system-in-unity3d/Blog-unity-coordinate-all.jpg)

### GUI界面的坐标体系

可以在ui界面中绘制一个button来测试一哈：

```c#
private void OnGUI()
{
    if (GUI.Button(new Rect(0f, 0f, 160f, 40f), "Click Me"))
    {
        //button clicked and do something here...
    }
}
```

它的原点 `(0, 0)` 在最左上角，因为屏幕宽度是 `Screen.width` ，高度是 `Screen.height` ，所以 GUI 体系右下角的坐标为： `(Screen.width, Screen.height)` 。

![unity-gui-coordinate-a.jpg](http://liuqingwen.me/blog/2017/07/31/understanding-coordinate-system-in-unity3d/unity-gui-coordinate-a.jpg)

### 视口viewport坐标体系

https://blog.csdn.net/u012371712/article/details/82789618