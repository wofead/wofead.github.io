# UI 渲染批次合并指南

##  渲染流程

RenderFlow 会根据渲染过程中调用的频繁度划分出多个渲染状态，比如 Transform，Render，Children 等，而每个渲染状态都对应了一个函数。在 RenderFlow 的初始化过程中，会预先根据这些状态创建好对应的渲染分支，这些分支会把对应的状态依次链接在一起。

例如如果一个节点在当前帧需要更新矩阵，以及需要渲染自己，那么这个节点会更新他的 flag 为

`node._renderFlag = RenderFlow.FLAG_TRANSFORM | RenderFlow.FLAG_RENDER`。

RenderFlow 在渲染这个节点的时候就会根据节点的 `node._renderFlag` 状态进入到 **transform => render** 分支，而不需要再进行多余的状态判断。

![v2.x 流程](../image/Cocos UI渲染批次合并指南/render-flow-2.png)

