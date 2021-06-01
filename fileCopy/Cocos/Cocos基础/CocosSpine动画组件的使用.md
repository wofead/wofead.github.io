# Cocos Spine动画组件的使用

## spine动画在cocos的使用

spine骨骼动画工具

1. 骨骼动画：把动画打散，通过工具，调整骨骼的运动等来形成动画。
2. spine是一个非常流行的2D骨骼动画制作工具
3. spine动画美术人员导出三个文件：
   1. .png：动画的骨骼的图集
   2. .atlas：每个骨骼在图集里面的位置和大小。
   3. .json：骨骼动画的anim控制文件，以及骨骼位置等信息。
4. 骨骼动画的导入：直接把三个文件拷贝到项目的资源目录下即可；
5. 使用骨骼动画；
   1. 直接拖动到场景；
   2. 创建一个节点添加sp.Skeleton组件。

## sp.Skeleton

### 控制面板属性

* Skeleton Data: 骨骼的控制文件.json文件;
* Default Skin: 默认皮肤;
* Animation: 正在播放的动画;
* Loop: 是否循环播放;
* Premuliplied Alpha 是否使用贴图预乘;
* TimeScale: 播放动画的时间比例系数;
* Debug Slots: 是否显示 Slots的调试信息;
* Debug Bone: 是否显示Bone的调试信息;

### sp.Skeleton的重要方法

 Skeleton是以管道的模式来播放动画，管道用整数编号，管道可以独立播放动画，Track;

1. clearTrack(trackIndex): 清理对应Track的动画
2. clearTracks(); 清楚所有Track动画
3. setAnimation(trackIndex, “anim_name”, is_loop)清除管道所有动画后，再从新播放
4. addAnimation(trackIndex, “anim_name”, is_loop);往管道里面添加一个动画;

## 动画事件监听

1.  setStartListener: 设置动画开始播放的事件;
2. setEndListener : 设置动画播放完成后的事件;
3. setCompleteListener: 设置动画一次播放完成后的事件;

```typescript
// use this for initialization

onLoad: function () {

    // 代码获取

    var spine = this.node.getChildByName("spine");

    var ske_com = spine.getComponent(sp.Skeleton);

    this.ske_com = ske_com;

    this.ske_com.setStartListener(function() {

        console.log("anim started");

    });

    this.ske_com.setEndListener(function() {

        console.log("anim end");

    });

    this.ske_com.setCompleteListener(function() {

        console.log("play once");

    });

}
```

