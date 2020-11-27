# Unity旋转四元数(Quaternion)与欧拉角

旋转一个物体的方法一般有三种，矩阵旋转，四元数旋转以及欧拉角旋转，他们各有利弊，在Unity中，我们常接触到的是欧拉角和四元数，所以接下来我们主要说明四元数和欧拉角。

在初接触Unity的时候，有没有一种感觉，为什么我的旋转总是报错，为什么我的旋转总是好像有点不对，仿佛来到了十万个为什么的世界？今天我们就一起探究一下，Unity中的旋转，究竟是怎么样子的。

[toc]

## 欧拉角

莱昂哈德·欧拉用欧拉角来描述刚体在三维欧几里得空间的取向。对于任何参考系，一个刚体的取向，是依照顺序，从这参考系，做三个欧拉角的旋转而设定的。所以，刚体的取向可以用三个基本旋转矩阵来决定。换句话说，任何关于刚体旋转的旋转矩阵是由三个基本旋转矩阵复合而成的。

引用百度百科里面的定义：

> 用来确定定点转动刚体位置的3个一组独立角参量，由[章动](https://baike.baidu.com/item/章动/695762)角θ、[旋进](https://baike.baidu.com/item/旋进/8795453)角（即进动角）ψ和自转角φ组成，为欧拉首先提出而得名。

如下图，定点O作为固定坐标系Oxyz以及固连于刚体的坐标系Ox'y'z'。以轴Oz和Oz‘为基本轴，其垂直面Oxy和Ox'y'为基本面，轴Oz量到Oz'的角度称为章动角。ON为基本面的交线，同时它也是垂直于ZOz'的。在右手坐标系中，由ON的正端看，角θ应按逆时针方向计量。由固定轴Ox量到节线ON的角度ψ称为进动角，由节线ON量到动轴Ox'的角度φ称为自转角。由轴Oz和Oz'正端看，角ψ和φ也都按逆时针方向计量。欧拉角（ψ,θ,φ）的名称来源于天文学。

若令Ox'y'z'的原始位置重合于Oxyz，经过相继绕Oz、ON和Oz'的三次转动Z（ψ）、N（θ）、Z'（φ）后，刚体将转到图示的任意位置（见刚体定点转动）。

![img](https://bkimg.cdn.bcebos.com/pic/242dd42a2834349b00a41638cdea15ce37d3bee8?x-bce-process=image/resize,m_lfit,w_194,limit_1)

**zxz 顺规**的欧拉角可以静态地这样定义, zxz这么叫的原因是，绕Oz,ON,和Oz’这三个轴旋转的原因：

> α 是 x-轴与交点线的夹角，β 是 z-轴与Z-轴的夹角，γ 是交点线与X-轴的夹角。

![](https://bkimg.cdn.bcebos.com/pic/962bd40735fae6cdfc54c75b0cb30f2442a70f23?x-bce-process=image/resize,m_lfit,w_220,limit_1)

### 在Unity中的定义

> xyz代表了三个角度，它们定义了一组有序的旋转，即围绕z轴旋转z度，然后围绕x轴旋转x度，然后围绕y轴旋转y度。 你应该只去读取或者直接设置这些数值，不要增加它们，因为当角度超过360度将会失败。应该使用Transform.Rotate去替代执行旋转操作。

在Unity中，使用的是四元数去执行旋转，不会存储欧拉角的累计值，因此超过360度会失败是没有问题的，意思是不会存过程，而是只存结果，举个例子:当值是361度的时候，你希望旋转361度，但是四元数只存结果1度，结果就是旋转了1度。



### Unity中的旋转轴

Unity中每次执行欧拉旋转，都是使用“当前轴”。

```c#
public void Rotate(Vector3 eulerAngles, Space relativeTo = Space.Self);
```

这个函数提供了可选的空间坐标参考系：

- Space.Self 局部坐标系，意味着本次欧拉旋转以物体当前的局部坐标朝向为基础出发执行旋转。
- Space.World 世界坐标系，意味着本次欧拉旋转以物体当前的世界坐标朝向为基础出发执行旋转。



**最重要的是：在本次欧拉旋转过程中，它的相对轴是始终不变的，不变的，不变的…**

比如我们可以指定一组欧拉旋转(90,60,30)，通过前述的顺规我们知道，先绕Z轴旋转30度，再绕X轴旋转90度，再绕Y轴旋转60度，虽然有这样的顺序，但是Z旋转后相对X轴、Y轴，都是执行本组欧拉旋转前的那个轴向，它没有发生变动，所以我称它为**“当前轴”**。

```c#
Transform.Rotate(new Vector3(90,60,30))
```

在执行

```c#
Transform.Rotate(new Vector3(0,0,30))；
Transform.Rotate(new Vector3(90,0,0));
Transform.Rotate(new Vector3(0,60,0))；
```

的结果是不一样的。第一种情况，只执行了一组欧拉旋转，第二种情况，执行了三组欧拉旋转，后两组欧拉旋转的相对轴在旋转时已经发生了变动。

### 万向节死锁（Gimbal Lock）问题

### 陀螺仪

平衡环架（英语：Gimbal）为一具有枢纽的装置，使得一物体能以单一轴旋转。由彼此垂直的枢纽轴所组成的一组三只平衡环架，则可使架在最内的环架的物体维持旋转轴不变，而应用在船上的陀螺仪、罗盘、饮料杯架等用途上，而不受船体因波浪上下震动、船身转向的影响。

下图就是一个Gimbal装置了，它是一个陀螺仪。中间有一根竖轴，穿过一个金属圆盘。金属圆盘称为转子，竖轴称为旋转轴。转子用金属制成，应该是了增加质量，从而增大惯性。竖轴外侧是三层嵌套的圆环，它们互相交叉，带来了三个方向自由度的旋转。

![陀螺仪结构](https://bkimg.cdn.bcebos.com/pic/ac6eddc451da81cb733b39145266d0160824310d?x-bce-process=image/resize,m_lfit,w_220,h_220,limit_1)

### Pitch、Yaw、Roll

在解释陀螺仪的工作原理之前，我先介绍一些转动的术语。在飞行器的航行中，进行XYZ三个方向旋转的旋转有专业的术语，见下图：

![Pitch、Yaw、Roll](https://img-blog.csdn.net/20170311161959258?/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQW5kcmV3RmFu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

沿着机身右方轴（Unity中的+X）进行旋转，称为**pitch**，中文叫**俯仰**。

沿着机头上方轴（Unity中的+Y）进行旋转，称为**Yaw**，中文叫**偏航**。

沿着机头前方轴（Unity中的+Z）进行旋转，称为**Roll**，中文叫**桶滚**。



当飞机桶滚的时候，只有一个轴旋转，当飞机pitch的时候两个轴旋转，当飞机yaw的时候三个轴都会发生旋转。

如下图：

![陀螺仪示意图](https://img-blog.csdn.net/20170311161231129?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQW5kcmV3RmFu/font/5a6L5L2T/fontsize/200/fill/I0JBQkFCMA==/dissolve/70/gravity/NorthEast)

![桶滚平衡](https://img-blog.csdn.net/20170311162818761?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQW5kcmV3RmFu/font/5a6L5L2T/fontsize/200/fill/I0JBQkFCMA==/dissolve/70/gravity/NorthEast)

![俯仰平衡](https://img-blog.csdn.net/20170311163443872?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQW5kcmV3RmFu/font/5a6L5L2T/fontsize/200/fill/I0JBQkFCMA==/dissolve/70/gravity/NorthEast)

![偏航平衡](https://img-blog.csdn.net/20170311163738829?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQW5kcmV3RmFu/font/5a6L5L2T/fontsize/200/fill/I0JBQkFCMA==/dissolve/70/gravity/NorthEast)

### 陀螺仪中的万向节死锁

当飞机pitch90度的时候：

![死锁开始](https://img-blog.csdn.net/20170311165114162?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQW5kcmV3RmFu/font/5a6L5L2T/fontsize/200/fill/I0JBQkFCMA==/dissolve/70/gravity/NorthEast)

飞机再次roll，你就会发现转子失去平衡了，没错，三个连接头，提供的自由度只对应了俯仰和偏航两个自由度，桶滚自由度丢失了。这就是陀螺仪上的“万向节死锁”问题：

![死锁的陀螺仪](https://img-blog.csdn.net/20170311165659086?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQW5kcmV3RmFu/font/5a6L5L2T/fontsize/200/fill/I0JBQkFCMA==/dissolve/70/gravity/NorthEast)

## 欧拉旋转、四元数、矩阵旋转之间的差异

除了欧拉旋转以外，还有两种表示旋转的方式：矩阵旋转和四元数旋转。接下来我们比较它们的优缺点。

### 欧拉角

- 优点：三个角度组成，直观，容易理解。
- 优点：可以进行从一个方向到另一个方向旋转大于180度的角度。
- 弱点：死锁的问题。

### 四元数

内部由四个数字（在Unity中称为x,y,z和w)组成，然而这些数字不表示角度或轴，并且通常不需要直接访问他们。

- 优点:四元数不存在万向节锁问题。
- 优点：存储空间小，计算效率高。
- 确定：单个四元数不能表示在任何方向上超过180度的旋转。
- 弱点：四元数的数字表示不直观。

### 矩阵旋转

- 优点：和四元数一样，不存在万向节锁的问题。
- 优点：可以表示围绕任意轴的旋转，四元数的旋转轴均通过物体中心的轴，矩阵则不受限。
- 缺点：矩阵旋转使用4*4矩阵，记录16个值，而四元数只需要4个数值。计算复杂，效率低。

## 四元数的数学

复数，在我们学习过的复数中，是为了解决被开的数为负数的情况的，即i^2等于-1.

在坐标系中：

![复数](https://img-blog.csdn.net/20170315131624089?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQW5kcmV3RmFu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

上图中，左右方向的轴（X轴）称为实数轴，上下的轴(Y轴)称为虚数轴。
任意一个二维矢量，都可以使用一个复数形式进行表示。也就是：**e = a + bi**

其中a是实数，i是虚数，**i \* i=-1**。

### 复数的运算

- 加法：(a+bi)+(c+di)=(a+c)+(b+d)i
-  (a+bi)-(c+di)=(a-c)+(b-d)i
- (a+bi)(c+di)=ac+bci+adi+bdi^{2}=(ac-bd)+(bc+ad)i

### 欧拉旋转定理

![欧拉旋转定理](https://img-blog.csdn.net/20170314222000480?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQW5kcmV3RmFu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

为了方便讨论旋转，我们避开矢量长度的影响，也就是假设问题是基于长度为1的矢量去讨论。由上图可以看出，当长度为1时，矢量落在长度为1的圆形上，此时实数轴上的a = cos（φ），虚数轴上的b = sin（φ），其中φ为旋转角度。

此时的表示形式为 **e = cos（φ） + sin（φ）i**

当圆上的一个矢量进行了连续的旋转时时，假设先旋转φ，再旋转θ，则结果应该是两个旋转的角的和的复数形式，即 **e = cos（φ+θ）+sin（φ+θ）i**。
$$
cos（φ+θ）+sin（φ+θ）i \\
= （cos（φ）cos（θ）-sin（φ）sin（θ））+(sin（θ）cos（φ）+cos（θ）sin（φ）)i\\
= cos（φ）cos（θ）+cos（φ）sin（θ）i+cos（θ）sin（φ）i- sin（φ）sin（θ）\\
= (cos（φ） + sin（φ）i)*(cos（θ） + sin（θ）i)\\
= e1 * e2\\
$$

### 四元数定义

四元数定义i、j、k三个虚数单位参与运算，并有以下运算规则：
$$
j∗k=i,k∗j=−i;\\
j∗k=i,k∗j=−i;\\
k∗i=j,i∗k=−j;\\
i∗i=j∗j=k∗k=i∗j∗k=−1\\
$$
i、j、k仍然理解为旋转，其中：

- i旋转代表X轴与Y轴相交平面中X轴正向向Y轴正向的旋转
- j旋转代表Z轴与X轴相交平面中Z轴正向向X轴正向的旋转
- k旋转代表Y轴与Z轴相交平面中Y轴正向向Z轴正向的旋转
- -i、-j、-k分别代表i、j、k旋转的反向旋转

一个普通四元数可以写成如下形式：
$$
\overline q=a+bi+cj+dk
$$
四元数的i、j、k之间乘法的性质与向量之间的叉积结果形式很类似，于是四元数有了另外一种表示形式:
$$
\overline q=(\vec{（x,y,z）},w)=(\vec u,w)
$$

### **加法定义**

$$
{\displaystyle \overline q_1+\overline q_2=w_1+w_2+{\vec {u_1}}+{\vec {u_2}}=(w_1+w_2)+(x_1+x_2)i+(y_1+y_2)j+(z_1+z_2)k}
$$

### **格拉斯曼积定义**

四元数的乘法有很多种，最常见的一种定义，与数学多项式乘法相同，称为格拉斯曼积。(注意，下面乘积的式子是由多项式形如a+bi+cj+dk的多项式进行多对多乘法(比如4项x4项=16项)计算后，使用矢量的点积和叉积替代部分计算项后形成)：
$$
\overline q_1\overline q_2=w_1w_2-{\vec  {u_1}}\cdot {\vec  {u_2}}+w_1{\vec  {u_2}}+w_2{\vec  {u_1}}+{\vec  {u_1}}\times {\vec  {u_2}}
$$
把向量部分和实数部分分开，可以写成：
$$
\overline q_1\overline q_2=((w_1{\vec  {u_2}}+w_2{\vec  {u_1}}+{\vec  {u_1}}\times {\vec  {u_2}}),(w_1w_2-{\vec  {u_1}}\cdot {\vec  {u_2}}))
$$
注意，格拉斯曼积符合结合律，但不符合交换律。

### 点积的定义

点积也叫做欧几里得内积，四元数的点积等同于一个四维矢量的点积。点积的值是 q1中每个元素的数值与 q2 中相应元素的数值的一对一乘积(比如4项x4项=4项)的和。这是四元数之间的可换积，并返回一个标量。
$$
{\overline q_1\cdot \overline q_2=w_1w_2+{\vec {u_1}}\cdot {\vec {u_2}}=w_1w_2+x_1x_2+y_1y_2+z_1z_2}
$$

### 叉积定义

四元数叉积也称为奇积。它和矢量叉积等价，并且只返回一个矢量值：
$$
{\overline p\times \overline q={\frac {\overline p\overline q-\overline q\overline p}{2}}}\\
{\overline p\times \overline q={\vec {u}}\times {\vec {v}}}\\
{\overline p\times \overline q=(cz-dy)i+(dx-bz)j+(by-xc)k}
$$

### 共轭定义

四元数的共轭的定义。即实部相同，虚部相反。（与复数概念类似）
$$
{\overline q^{*}=(\vec{（-x,-y,-z）},w)=(-\vec v,w)}
$$

### 定义推论

从以上三个定义，可以得出一个推论，一个四元数与其共轭的格拉斯曼积等于此四元数与其自身的点积：
$$
{\overline q\overline q^{*}}={{\overline q^{*}\overline q}} ={\overline q\cdot \overline q= {x^{2}+y^{2}+z^{2}+w^{2}}}\\
\overline q\overline q^{*}=ww+{\vec  {u}}\cdot {\vec  {u}}={\overline q\cdot \overline q}
$$

### **模的定义**

四元数的模的定义。与矢量的模类似，都表示长度或者绝对值的意思。四元数的绝对值是四元数到原点的距离。
$$
{ |\overline q|={\sqrt {x^{2}+y^{2}+z^{2}+w^{2}}}}={\sqrt {w^{2}+\vec u \cdot \vec u}}={\sqrt {\overline q\cdot \overline q}}={\sqrt {\overline q\overline q^{*}}}
$$

### 逆的定义

四元数的逆通过  $ \overline q^{-1}\overline q =1$来定义。在此式左右同时乘以$\overline q^{*}$,可以得到：
$$
\overline q^{-1}\overline q\overline q^{*} = \overline q^{*} \\
\overline q^{-1}(\overline q\overline q^{*}) = \overline q^{*} \\
\overline q^{-1}(\overline q\overline q) = \overline q^{*} \\
\overline q^{-1} = \frac {\overline q^{*}} {\overline q \cdot \overline q}\\
$$

### **单位四元数的定义**

所谓的单位量，都是指，任意原始量乘以单位量之后保持原始量不变。如对于实数而言，任意原始实数乘以实数1等于原始实数，因此1被定义为单位.单位四元数为:
$$
\overline q_i = (\overline 0, 1)
$$

### 四元数归一化

归一化也叫单位化，与矢量的概念一样，就是为了把长度调整到单位长度。现在单位四元数已经得知，其单位长度显然是1。我们也知道了模的计算方法，因此把四元数的归一化方法如下：
$$
{\overline q^{,} =\frac {\overline q}{|\overline q|}=\frac {\overline q}{{\sqrt {w^{2}+\vec u \cdot \vec u}}}}
$$
前述的单位复数表示形式为：e = cos(φ）+ sin(φ)i，因为此时，复数的长度为$cos(φ)^2+sin(φ)^2=1$,我们使用类似形式表示归一化的四元数：
$$
{\overline q =({\vec sin(φ)}\vec n,cos(φ))}
$$
也就是实部现在使用$cos(φ)$表示，虚部使用$sin(φ)\vec n$表示，其中$\vec n$是归一化的三维矢量，由于$\vec a \cdot \vec b=|\vec a||\vec b|cos(\theta_{ab})$,因此有n向量的平方等于1.
$$
{ |\overline q|={\sqrt {w^{2}+\vec u \cdot \vec u}}={\sqrt {cos(φ)^{2}+sin(φ)^{2}(\vec n \cdot \vec n)}}}=1
$$

### 四元数映射_实数

我们可以将实数映射到四元数空间，所谓的映射，也就是说1对1的找到相应的存在，并且在不同空间施加施加相对应运算的法则时获得相同的结果。
例如，对应实数w，尝试进行如下映射：
$$
\overline q=(\vec 0,w)
$$
那么是否能够遵守相应的运算法则？也就是说，如果实数 wawa与实数 wbwb均被映射到了四元数空间，那么它们的加法和乘法的结果是否还能映射回去，与原始空间的运算获得相同的结果？加法显然可以，这里进行乘法计算测试：
$$
\overline q_a = (\vec 0,w_a)\\
\overline q_b = (\vec 0,w_b)\\
\overline q_a\overline q_b = (\vec 0,w_a) (\vec 0,w_b)=w_aw_b-{\vec  {0}}\cdot {\vec  {0}}+w_a{\vec  {0}}+w_b{\vec  {0}}+{\vec  {0}}\times {\vec  {0}}\\
\overline q_a\overline q_b =w_aw_b+\vec  {0}=(\vec 0,w_aw_b)
$$

### **四元数映射_矢量直接映射**

$$
\overline q_a = (\vec v_a,0)\\
\overline q_b = (\vec v_b,0)\\
\overline q_a\overline q_b = (\vec v_a,0) (\vec v_b,0)=00-{\vec  {v_a}}\cdot {\vec  {v_b}}+0{\vec  {v_b}}+0{\vec  {v_a}}+{\vec  {v_a}}\times {\vec  {v_b}}\\
\overline q_a\overline q_b = (\vec  {v_a}\times \vec  {v_b}, -\vec v_a \cdot\vec v_b)
$$

可以看到，此时出现了之前两个四元数中均不存在的实部。因此这个映射法则是错误的。（本身两个向量之间如果直接使用多对多的格拉斯曼积式乘法也是没有意义的，所以结果可想而知。那么，是不是就不能映射矢量了？不是，只是映射法则错误。回想之前在在复数中也曾使用过格拉斯曼积式的乘法 如果矢量用$e = cos(φ)+ sin(φ)i$表示,那么此时的 e表示了一个二维旋转，向量的旋转可以使用格拉斯曼积形式的复数乘法e1∗e2表示，因此，映射法则应该与角度相关。）

### **四元数映射_矢量间接映射**

#### 同态

先考虑一个同态概念：假设M，M′是两个乘集，也就是说M和M′是两个各具有一个封闭的具有结合律的运算*与*的代数系统。φ是M射到M′的映射，并且任意两个元的乘积的像是这两个元的像的乘积，即对于M中任意两个元a,b,满足：

φ(a*b)=φ(a)*φ(b)；
也就是说，当a→φ(a)，b→φ(b)时，a*b→φ(a)*φ(b)，
那么这映射φ就叫做M到M′上的同态。我前面所解释的可以互相映射大致意思也就是同态。

### 四元数中的同态函数

为什么我们研究同态？因为这里涉及到我们希望出现的向量的旋转，一个三维空间的向量被旋转之后，它应该还是一个三维矢量。这个旋转函数应该满足以下要求：

1. 旋转完的矢量长度应该保持不变。
2. 假如对向量A和B都执行φ旋转，那么向量A、B的夹角θ应该与φ(A)、φ(B)的夹角应该相同。由矢量点积我们知道$A\cdot B=|A||B|cos(\theta)$,，在满足1的情况下，也就是模的乘积已经不变，那么如果点积结果相同，也就能保证角度不变，也就是：$φ(A) \cdot φ(B)=A \cdot B$.
3. 假如对向量A和B都执行φ旋转，那么旋转完毕的φ(A)、φ(B)的相对方向关系与A、B应该保持一致，也就是所谓的手向性应该保持一致（比如原来A的正左侧恰好B，旋转完毕应该还是在正左侧，仅有夹角是不够的，正右侧也是90度关系）。假如满足下面的式子，则手向性不变:$φ(A) \times φ(B)=A \times B$

我们通过间接映射将矢量V映射到四元数，先如直接映射那样，映射成一个纯虚四元数，即 $(\vec V,0)$，在执行φ变换。$φ(V)$,由于,A和B都是纯虚四元数，因此$w_a=w_b=0$，$A\cdot B = u_a \cdot u_b$:
$$
A B=w_aw_b-u_a \cdot u_b+w_a \cdot u_b+w_b \cdot u_a+u_a \times u_b\\
A B=-u_a \cdot u_b+u_a \times u_b\\
A B=-A \cdot B+A \times B\\
φ(A) φ(B)=φ(AB)
$$

### 旋转四元数

最后，给出旋转四元数，也就是一直在追求的φ()函数。
$$
{\overline q =({\vec sin(\frac \theta 2)}\vec n,cos(\frac \theta 2))}
$$
它的含义是围绕 $\vec n$轴旋转 θ角度。 具体旋转的方法使用以下函数：
$$
{\overline p^{,} =\overline q\overline p\overline q^{*}   }
$$
其中$\overline p$是被旋转四元数， $\overline p^,$是结果四元数。这里的 $\overline p^*$写成 $\overline p^{-1}$也是可以的，因为根据前述对逆的计算，对于此时模为1的 $\overline p$来说，它们结果相同的。 具体推究方法以及证明方法过于复杂，就不在这里给出了。



## Unity中关于四元数的API详解

### Quaternion类

Quaternion（四元数）用于计算Unity旋转。它们计算紧凑高效，不受万向节锁的困扰，并且可以很方便快速地进行球面插值。 Unity内部使用四元数来表示所有的旋转。

Quaternion是基于复数，并不容易直观地理解。 不过你几乎不需要访问或修改单个四元数参数（x，y，z，w）; 大多数情况下，你只需要获取和使用现有的旋转（例如来自“Transform”），或者用四元数来构造新的旋转（例如，在两次旋转之间平滑插入）。

大部分情况下，你可能会使用到这些函数：

- Quaternion.LookRotation，
- Quaternion.Angle
- Quaternion.Euler
- Quaternion.Slerp
- Quaternion.FromToRotation
- Quaternion.identity。

Quaternion 是一个结构体，本身成员变量相对简单，可以作为函数参数高效传递。

### Unity的默认方向

Unity中使用左手坐标系，假如把世界坐标系跟东南西北进行结合起来看，大致如下图所示：

![方向](https://img-blog.csdn.net/20170327005655077?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQW5kcmV3RmFu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

默认的方向对应如下表：

| 坐标轴 | 对应方向 |
| ------ | -------- |
| +x     | 右(东)   |
| -x     | 左(西)   |
| +y     | 上       |
| -y     | 下       |
| +Z     | 前(北)   |
| -Z     | 后(南)   |

假设以你自己身体为例，你站立在地面上，面朝北方，此时就是默认方向，也就是Unity中的方向就是面向+Z轴方向，那么此时+X轴在东方，+Y轴对应正上方。此时对应的欧拉角是(0,0,0),此时对应的前方矢量是(0,0,1)，上方矢量是(0,1,0)。

这里我区分了左右上下前后的概念，因为这些概念同时也对应了Vector3类、Transform类中的相应的方向函数。

### 成员变量

- eulerAngles 欧拉角，返回当前四元数所对应的欧拉角
- this[int] 可以使用类似数组和下标的形式从四元数中获取四个四元数参数
- x、y、z、w 分别代表x、y、z、w 参数。

### 静态成员

 identity单位四元数，也就是默认的无旋转状态，此时与世界坐标相同，前方指向+Z,上方指向+Y

### 成员函数

| 函数形式                                                     | 解释                                  |
| ------------------------------------------------------------ | ------------------------------------- |
| void Set(float new_x, float new_y, float new_z, float new_w) | 设置x、y、z、w 分量，与this[]功能相同 |
| void SetFromToRotation(Vector3 fromDirection, Vector3 toDirection) | 设置成静态函数FromToRotation的结果    |
| void SetLookRotation(Vector3 view, Vector3 up = Vector3.up)  | 设置成静态函数LookRotation的结果      |
| void ToAngleAxis(out float angle, out Vector3 axis)          | 设置成静态函数AngleAxis的结果         |

说明：成员函数几个set方法多用于将当前四元数设置成目标四元数，目标四元数的构建方法与对应名称的静态函数相同。

### 方向的表示法

1. **欧拉角表示法**：假如你使用一组欧拉角表示旋转,XYZ三个参数代表相应轴向按照顺归YZX的旋转，因此(0、90、90)代表先进行+Z轴旋转90度，再沿着+Y轴进行90度旋转。
2. **前方上方矢量界定法**：编程过程中，大部分需要明确指定方位的时候就需要使用这个方法。要确定一个朝向，我们可以使用两个向量来确定：即前方矢量和上方矢量。当一个朝向的前方和上方确定之后，这个朝向也就完全确定了。
   举例来说，如果现在只提供一个朝向，就是你现在面朝北方，那么这个方向已经完全确定了吗？显然没有。因为你右侧躺在地上，看向北方，还是在面朝北方，这时候就需要另外一个矢量，也就是上方。当给出上方之后，这个朝向就完全确定了。
3. **绕轴旋转界定法**：第三种定义旋转的方法就是围绕某个指定的轴向旋转一定的角度。这个方法也可以确定一个相对旋转，它以从默认方向(此时前方+Z，上方+Y)出发，沿着指定的轴向进行指定角度的旋转，旋转后的前方和上方是确定的。因此这个方法也可以用来确定朝向。
4. **A向到B向相对旋转表示法**：还有一种方法就是从A向到B向的相对旋转，这种表示了一个旋转的相对变化。比如A为(0,1,0)，B为(0,0,1)，也就是相对旋转量代表原来的上方被旋转到了前方，这样的一个四元数也可以用欧拉角表示成(90,0,0)，也就是沿着+X轴旋转了90度。

### 静态函数

| 函数形式                                                     | 解释                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| static float Angle(Quaternion a, Quaternion b)               | 计算两个四元数前方矢量之间的夹角度数                         |
| static Quaternion AngleAxis(float angle, Vector3 axis)       | 构建一个四元数，它表示沿着一个轴旋转固定角度，即上述表示法③  |
| static float Dot(Quaternion a, Quaternion b)                 | 计算两个四元数之间的点积，返回一个标量，这个函数一般用不到，它的点积不代表什么具体的物理含义，具体定义方法见我的前述文章 |
| static Quaternion Euler(float x, float y, float z)           | 构建一个四元数，它用欧拉旋转表示，即上述表示法①              |
| static Quaternion FromToRotation(Vector3 fromDirection, Vector3 toDirection) | 构建一个四元数，它表示从指向fromDirection方向到指向toDirection方向的相对旋转量，见上述表示法④ |
| static Quaternion Inverse(Quaternion rotation)               | 构建一个四元数，它是指定的四元数的逆，也就是逆向旋转，比如原四元数表示相对+X轴旋转了90度，那么此函数结果就是相对+X轴旋转了-90度 |
| static Quaternion Lerp(Quaternion a, Quaternion b, float t)  | 构建一个四元数，表示从四元数a到b的球面插值，所谓的插值也就是中间旋转量，从a作为起点，此时对应t为0，到b为终点，此时对应t为1。当t取0-1之间的小数时，就代表了中间的插值结果。这个方法与Slerp相同，计算速度快，但是精度低，如果相对旋转变化量很小，则效果不理想 |
| static Quaternion LerpUnclamped(Quaternion a, Quaternion b, float t) | 与Lerp相同，区别是，Lerp的t值会被钳制在[0,1]之间，而此方法则不会，t允许超出计算 |
| static Quaternion LookRotation(Vector3 forward, Vector3 upwards = Vector3.up) | 构建一个四元数，使用前方上方矢量确定朝向，也就是上述表示法②  |
| static Quaternion RotateTowards(Quaternion from, Quaternion to, float maxDegreesDelta) | 构建一个四元数，表示从一个四元数from(的前方)向着另外一个四元数(的前方)旋转，但不能超出指定的角度，也就是如果两个前方矢量夹角超过指定角度，则旋转到达指定角度时就停止，若是夹角本身不足的话，则结果直接为目标四元数to，与上述表示法④的意思很接近 |
| static Quaternion Slerp(Quaternion a, Quaternion b, float t) | 球面插值，与Lerp功能相同，t值也被钳制，计算精度高，但是速度相对较慢 |
| static Quaternion SlerpUnclamped(Quaternion a, Quaternion b, float t) | 与Slerp功能相同，只是t值不被钳制，允许超出计算               |
| static Quaternion operator * (Quaternion lhs, Quaternion rhs) | 乘法运算符重载，当表示两个连续的旋转时，可以使用lhs * rhs的形式得出连续旋转的结果,lhs为左值，rhs为右值。注意左值是先进行的旋转，叠加右值旋转。用法示例:lhs = lhs * rhs; |
| static Vector3 operator *(Quaternion rotation, Vector3 point) | 乘法运算符重载，表示对一个矢量point施加旋转rotation，得出旋转后的结果矢量。用法示例:Vector3 result=rotation * point |

## 引用

[【Unity编程】Unity中的欧拉旋转](https://blog.csdn.net/AndrewFan/article/details/60866636)

[【Unity编程】欧拉角与万向节死锁](https://blog.csdn.net/andrewfan/article/details/60981437)

[【Unity编程】四元数(Quaternion)与欧拉角](https://blog.csdn.net/AndrewFan/article/details/62057519)

