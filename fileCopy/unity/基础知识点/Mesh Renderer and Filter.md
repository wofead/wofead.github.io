# Mesh Renderer and Filter

创建网格面的核心就是为其添加2个组件：Mesh Renderer（网格渲染器）和Mesh Filter（网格过滤器）。

添加组件的方法有2种：

1. 选择一个游戏对象，然后执行Component→Mesh→Mesh Filter 和Mesh Renderer。

2. 代码创建：

   ```c#
   //添加MeshFilter
   gameObject.AddComponent<MeshFilter>();
   //添加MeshRenderer
   gameObject.AddComponent<MeshRenderer>();
   //获得Mesh
   mesh = GetComponent<MeshFilter>().mesh;
   ```

   

## 理论分析

由于网格面是由三角形组成的，所以我们有必要知道三角的三个顶点的位置。数组mesh.vertices就是用于存储三角形顶点坐标。

当顶点数超过3个的时候，我们连接点的顺序不同，就会绘制出不同形状的图形，所以我们必须要获得我们想要的连接点的顺序。数组mesh.triangles就是用来记录连接三角形的顺序的。由于每绘制一个三角形需要知道其三个顶点的顺序，那么绘制n个三角形就需要知道3*n个点的顺序。即数组mesh.triangles的长度是3的倍数。