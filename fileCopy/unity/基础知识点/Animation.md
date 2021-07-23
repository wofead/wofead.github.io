# Animation

现版本unity提供自带的两种动画状态机Animation和Animator用来控制场景中动画的运行,其实就是前面的是旧版的后面是新版的，所以大同小异。

> 步骤一：选中待提添加动画的物体，Window--->Animation(Ctrl+6)，弹出下图视图窗口。

注意：如果选中的物体无Animation/Animator组件，会自动添加Animator组件。如使用旧版Animation动画，可先添加Animation组件后再进行此操作。 

>  步骤二：单击Create按钮，在弹出的对话框中为待新建的Animation动画命名并选择保存路径，单击保存。

保存后Animation视图变为可编辑状态。同时在对应路径会生成Animation文件。

如未在物体添加Animator组件，会自动在物体检视视图添加此组件，并生成同名Animator文件。

> 步骤三：通过Add Property添加制作动画

单击后面加号可添加要更改的物体属性。 

更改不同属性相应的参数，即可制作动画。

在录制状态下，可以通过直接设置更改场景视图或检视视图的属性参数来制作动画。

设置精灵切换时，可以直接将精灵拖拽到对应帧的位置即可。

****

  （Preview ）预览：启用/禁用场景预览模式。

   （红点）录制：启用/禁用关键帧记录模式。

  （左双箭头） 转到动画剪辑开头。

   （左箭头）转到上一个keyframe。

  （播放） 播放动画剪辑。

   （右箭头）转到下一个keyframe。

   （右双箭头）转到动画剪辑末尾。

  （60） 当前帧。

   （CubeA）当前动画名字，下拉可以创建新的Animation动画。

  （Sample 60） 样本，每秒分的帧数。

   （菱形+）添加关键帧。

  （下指向加） 添加事件。

   时间轴。

   控制删除属性或增减键。

   要设置动画的键。

   简报。

   曲线

****

旧版：

![img](https://img-blog.csdnimg.cn/20190422162921328.png) ![img](https://img-blog.csdnimg.cn/20190422161920862.png)

| Length                                         | 长度（动画时长）                                             |
| ---------------------------------------------- | ------------------------------------------------------------ |
| Wrap ModeDefaultOnceLoopClamp ForeverPing Pong | 贴图间拼接模式（动画播放模式）默认一次循环播放完一次后循环最后一帧乒乓 |

新版：

![img](https://img-blog.csdnimg.cn/20190422163009552.png) ![img](https://img-blog.csdnimg.cn/201904221630452.png)

| Length       | 长度（动画时长）                     |
| ------------ | ------------------------------------ |
| Loop Time    | 循环                                 |
| Loop Pose    | 循环动作（使循环时头部尾部衔接平滑） |
| Cycle Offset | 平滑度                               |

> Animation组件

![img](https://img-blog.csdnimg.cn/20190422155649968.png) ![img](https://img-blog.csdnimg.cn/20190422155726816.png)

| Animation                               | 动画（默认）                         |
| --------------------------------------- | ------------------------------------ |
| Animations<br />Size<br />Element<br /> | 动画（默认）循环<br />大小<br />元素 |
| Play Automatically                      | 自动播放                             |
| Animate Physics                         | 动画物理                             |
| Culling Type                            | 剔除类型                             |

