# Unity Renderer Order

使用 SpriteRenderer 可直接调整Sorting Layer来决定Render顺序后，很好奇为什么可以改一个数值便能调整sprites在rende结果的先后顺序，而不用调整距离相机的距离来达到。

## Renderer’s rendering order

假设杀的人中关闭深度机制的判断（ZTest Always），或者在render场景物件过程中，都不写入深度（ZWrite Off）,既没有z-buffering机制，rendering order会决定成像的结果，越晚写的物件永远都在其他较早写的物件智商。

而在SpriteRenderer直接修改sorting layer以及order inlayer来改变rendering order，就能调整该物件在算出结果物件的前后，便是给予此缘故，更多细节可查看Sprites/Defaut shader程式码

在Unity中，rendering order是根据以下参数进行排序：Camera depth >  Material type > Sorting layer > Order in layer > Material render queue > Camera order algorithm.

* Camera Depth
  * 数字越大越晚
  * 通常无法搭配Clear Flags:Don‘t Clear,因为不会清楚depth buffer(z-buffer)
  * 程序设定camera.depth
  * 选择场景中camera编辑
* Marterial type
  * 先画不透明物件（opaque），再画透明物件（transparent）
  * 根据marerial render queue来决定，数值小于等于2500为不透明物件，数值大于2500为透明物件
  * Sorting layer
    * 数字越晚画
    * 大多数renderere都有支援，但仅有SpriteRenderer以及ParticleSystemRenderer能在预设的Inspector编辑
    * 程序设定renderer.sortingOrder
    * 选择场景的SpriteRenderer或是ParticleSystemRenderer编辑Sorting layer
  * Order in layer
    * 数字越大越晚画
    * 大多数的renderer都有支持此参数，但仅有SpriteRenderer以及ParticleSystemRenderer能在预设的Inspector编辑
    * 程序设定renderer.sortingOrder
    * 选择场景的SpriteRenderer或是ParticleSystemRenderer编辑Sorting layer
  * Material render Queue
    * 数字越大越晚画
    * 預設值會從 Shader 取得，但可自行定義
    * 不透明物件 (Opaque)、半透明物件 (AlphaTest)、透明物件 (Transparent) 預設值分別 2000、2450、以及 3000
    * 通常只有透明物件會關閉 ZWrite
    * 程式設定 `material.renderQueue`
    * 選擇專案中的 material 編輯
  * **Camera render algorithm**
    * 無法在預設編輯器修改，使用程式調整
    * 非透明物件排序演算法 `camera.opaqueSortMode`

