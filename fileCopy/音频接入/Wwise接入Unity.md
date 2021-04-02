# 接入Unity

[toc]

## Wwise的下载

在官网中下载Wwise的Launcher，然后现在比较最新版本，在我们的项目是Unity2018.4.3，由于2021版本存在问题，这里选择2019版本的Wwise。

## 将Wwise SDK嵌入到Unity中

下载之后打开Unity选项，选择将Wwise嵌入到Unity中。

这个时候在Unity的Assets目录下会多一个Wwise的目录。

如果要卸载Wwise，将Wwise这个目录删除就ok了。

## 配置Wwise

首先我们需要把Unity自己的声音禁用掉。在Edit中的第一个Audio，**Disable Unity Audio**。

在Edit菜单中，选择项目设置，里面可以看到Wwise的设置，一般选择默认就OK了。如果出现警告可以把里面的`In Editor Warning`关掉.

在Wwise Initialization中，我们可以设置Base Path，这些相对目录都是`streamingassets`目录的。

## 使用事项

游戏中，我们一般通过使用`AudioEngine`来派发声音事件，但是在这里要注意，不能再XLua中导出`AudioEngine`这个类，这个类在Editor状态下是屏蔽Android的代码的，只能让xlua去反射取，或者后续采用其它手段进行操作，**这里一般选择拓展的方式**，可以像拓展Unity API那样对Wwise进行拓展。

还有就是由于我们的游戏资源都在Res目录下，但是Audio可能会不同步，这里我们需要做一件事情，就是设置目录，这样既可以解决目录问题，还可以解决热更新问题。