# Xlua和Fairy GUI

[toc]

## 项目过程中遇到的问题

1. fairy中，做关联的时候记住对每个组件都做好关联，包好高级组和每个组件，这样子才不会导致分辨率变化时位置不对(结算的那个tipContainer)
2. 当给fairy组件设置的属性是一个委托的时候，记得在c#层进行转换，例如GList中的itemRender和itemProvider。
3. Fairy中，动画播放完之后，记得还原组件之前的位置，或者每次播放这个动画的时候，第一帧为关键帧而且就是原始的位置。
4. 在项目中，给组件进行赋值url的时候，最好直接给normalLize之后的值，虽然也可以使用包名加上组件名，但是可能会混淆，特别是项目改了获取组件的底层代码之后，很有可能有些地方漏掉了，导致出现问题。
5. 对于GList如果使用AddItemFromPool的方式往里面添加item的时候，记得在退出界面的时候RemoveChildrenToPool或者设置numItems等于0，这样下次进来list里面的东西才是空的。
6. 当需要根据index来获取item的时候使用GetChildAt(index)
7. ChildIndexToItemIndex(index),这个函数用来获取真实item的index。
8. ItemIndexToChildIndex(index),这个函数用来获取数据的index。
9. fairy中插入unityprefab，通过使用wrapper，SetNativeObject，设置wrapTarget来实现。
10. 使用GameObjectEx中的addUIToSelf来将fairy组件添加到gameobject中。

## XLua

生成Wrap文件的配置在文件XLuaCustomExport中，这里面有两个list非常重要，一个是CSharpCallLuaList，这个里面放的是回调函数，即等同于c#中的delegate函数，一个是LuaCallCSharpListUnity，这个是lua层调用csharp层。

## Fairy Wrapper

1. 创建一个wrapper
2. 给holder设置SetNativeObject为上面这个wrapper
3. new unity prefab：LuaClass.SpineAnimation:NewGameObjectFromPrefab(path)，这个函数的返回值一个是spin一个是gameobject，这个spin就是我们封装的lua动画操作组件
4. 给wrapper设置wrapTarget
5. 设置这个gameobject的scale和position
6. 通过调用spin这个组件中的函数来播放spin动画

## 创建unity gameobject

可以通过下面的方式创建一个gameobject并添加到这个父容器内：

```lua
LuaClass.BaseLuaComponent:NewGameObject({__cname = "EnergyBall"}):addTo(self)
```

NewGameObject：表示创建一个gameobject，里面表示名称，addTo表示添加到那里

我们可以通过GetComponent来获取unity prefab中的组件，然后对其进行操作。

## 今日工作总结

1. 和服务器进行了联调，基本已经达成目的
2. itemProvider这个问题已经解决，这个是大头
3. 敏感字这个文件打不开，好坑啊，然后就没有测试这个功能
4. ChatView终于像个样子了，剩下的就是接入周边功能了

   