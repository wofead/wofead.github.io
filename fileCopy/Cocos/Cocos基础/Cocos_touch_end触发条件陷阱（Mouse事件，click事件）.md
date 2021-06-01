



# Touch_End的触发条件

[toc]

目标：虚拟摇杆，在摇杆UI上按下时摇杆运作，在任意舞台位置松开时摇杆停止。

问题：无法检测到舞台松开的touch_end事件。

![img](../image/Cocos_touch_end触发条件陷阱（Mouse事件，click事件）/aHR0cHM6Ly9pbWcyMDIwLmNuYmxvZ3MuY29tL2Jsb2cvMTA2MjE3NC8yMDIwMDYvMTA2MjE3NC0yMDIwMDYwNDIyMzYwNTc0OS0xMjM1OTMyODYxLnBuZw)

```typescript
 //在摇杆上按下
 this.vj.on(cc.Node.EventType.TOUCH_START, this.onTouchStart, this);
 //在舞台上移动
 this.canvas.on(cc.Node.EventType.TOUCH_MOVE, this.onTouchMove, this);
 //在舞台上松开
 this.canvas.on(cc.Node.EventType.TOUCH_END, this.onTouchEnd, this);
 //在舞台上松开
 this.canvas.on(cc.Node.EventType.TOUCH_CANCEL, this.onTouchCancel, this);
```

结果是：

在摇杆区域内： touch_start 有效  -->  touch_move有效  --> touch_end有效 --> touch_cancel无效。

在摇杆区域外： touch_start 无效 -->  touch_move有效 ---> touch_end有效 -->touch_cancel无效。

在摇杆区域内点击，移动到摇杆区域外松开： touch_start有效 --> touch_move有效 --> touch_end无效 --> touch_cancel有效。



玩家操作虚拟摇杆，然后手指移动到了摇杆区域外松开，就检测不到touch_end事件！

所以最终玩家松开的操作得用touch_end和touch_cancel共同检测.

用touch_end事件来监听摇杆区域内的松开情况.

用touch_cancel事件来监听摇杆区域外的松开情况.



### 具体原因

* TOUCH_START:按下即会触发该事件；
* TOUCH_MOVE:手指在屏幕上移动会触发该事件；
* TOUCH_CANCEL:在某些特定情况下，CocosCreator会判定该事件失效，即不能正常完成START-END的流程，这时会触发该事件，这些情况已知包括：
  1. 手指按下（TOUCH_START）——手指滑动了较长的距离，但没有离开接收事件的节点（TOUCH_MOVE）——手指离开屏幕，本次判定为（TOUCH_CANCEL）
  2. 手指按下（TOUCH_START）——手指滑动离开了接收事件节点的感知范围（TOUCH_MOVE）——手指离开屏幕，判定为（TOUCH_CANCEL）

TOUCH_END:本次触摸基本按照START-END的顺序结束了，手指离开屏幕时会触发该事件。
