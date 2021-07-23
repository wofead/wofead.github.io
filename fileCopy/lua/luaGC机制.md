# Lua GC

[toc]

## Lua GC 算法

lua gc采取的是标记-清除算法，即一次gc分为两步：

1. 从根节点开始遍历gc对象，如果可达，则标记
2. 遍历所有的gc对象，清除没有被标记的对象

### 二色标记法

![image](https://blog-1251569602.cos.ap-shanghai.myqcloud.com/blog/12/10.png)

lua 5.1之前采用的算法，二色回收法是最简单的标记-清除算法，缺点是gc的时候不能被打断，所以会严重卡住主线程

### 三色标记法

![image](https://blog-1251569602.cos.ap-shanghai.myqcloud.com/blog/12/1.png)

1. Lua5.1开始使用一种三色回收的算法

   * 白色：在gc开始阶段，所有对象颜色都为白色，如果遍历一遍之后，对象还是白色的将被清除。
   * 灰色：灰色用在分布遍历阶段，如果一直有对象为灰色，则遍历将不会停止。
   * 黑色：确实被引用的对象，将不会被清除，gc完成之后会重置为白色。

2. luajit使用状态机来执行gc算法，共有6种状态：

   * GCSpause：gc开始阶段，初始化一些属性，将一些根节点（主线程对象，主线程环境对象，全局对象等）push到灰色链表中。
   * GCSpropagate：分布进行扫描，每次从灰色链表pop一个对象，遍历该对象的子对象，例如如果该对象为table，并且value没有设置为week，则会遍历table所有table可达的value，如果value为gc对象且为白色，则被push到灰色链表中，这一步将一直持续到灰色链表为空的时候。
   * GCSatomic：原子操作，因为GCSpropagate是分布的，所以分步过程中可能有新的对象创建，这时候将再进行一次补充遍历，这遍历是不能被打断的，但因为绝大部分工作被GCSpropagate做了，所以过程会很快。新创建的没有被引用的userdata，如果该userdata自定义了gc元方法，则会加入到全局的userdata链表中，该链表会在最后一个GCSfinalize处理。
   * GCSsweepstring：遍历全局字符串hash表，每次遍历一个hash节点，如果hash冲突严重，会在这里影响gc。如果字符串为白色并且没有被设置为固定不释放，则进行释放。
   * GCSsweep：遍历所有全局gc对象，每次遍历40个，如果gc对象为白色，将被释放。
   * GCSfinalize：遍历GCSatomic生成的userdata链表。如果该userdata还存在gc元方法，每次处理一个。

   ## 什么时候会gc？

   1. luajit中有两个判断是否需要gc的宏，如果需要gc，则会直接进行一次gc的step操作

      ```c
      /* GC check: drive collector forward if the GC threshold has been reached. */
      #define lj_gc_check(L) \
        { if (LJ_UNLIKELY(G(L)->gc.total >= G(L)->gc.threshold)) \
            lj_gc_step(L); }
      #define lj_gc_check_fixtop(L) \
        { if (LJ_UNLIKELY(G(L)->gc.total >= G(L)->gc.threshold)) \
            lj_gc_step_fixtop(L); }
      ```

      * gc.total: 代表当前已经申请的内存
      * gc.threshold：代表当前设置gc的阈值

   2. 这两个宏会在各个申请内存的地方进行调用，所以当前申请的内存如果已经达到设备设置的阈值，则会申请的所有对象都会有gc消耗。

   

