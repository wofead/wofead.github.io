# Unity四元数和向量相乘作用及其运算规则

**作用：**四元数和向量相乘表示这个向量按照这个四元数进行旋转之后的到的新的向量。

eg：向量vector3(0,0,10),绕着Y轴旋转90度，得到新的向量是vector3(10,0,0)。

在Unity中表示为：

```c#
Vector3 v = new Vector3(0,0,10);
Quaternion q = Quaternion.Euler(0,90,0);
Vector3 newV = q*v;
//newV = (10,0,0)
```

**符合旋转：**四元数依次相乘，最后乘以向量。

再来一个复杂一点的例子

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class QuaternionTest : MonoBehaviour
{
    public Transform target;
    public Transform a;
    public Transform b;
    // Start is called before the first frame update
    void Start()
    {
        //A正前方10m
        Vector3 fw = Vector3.forward;
        target.position = a.position + fw * 10f;
        Debug.DrawRay(target.position, target.up, Color.blue, 100f);

        //A前，Y45度方向10米
        Quaternion q = Quaternion.Euler(0, 45, 0);
        Vector3 v = q * fw * 10f;
        target.position = a.position + v;
        Debug.DrawRay(target.position, target.up, Color.red, 100f);

        //A到B方向10米
        Vector3 dir = b.position - a.position;
        q = Quaternion.LookRotation(dir);
        target.position = a.position + q * fw * 10f;
        Debug.DrawRay(target.position, target.up, Color.green, 100f);

        Quaternion q1 = Quaternion.Euler(0, 30, 0);
        target.position = a.position + q * q1 * fw * 10f;
        Debug.DrawRay(target.position, target.up, Color.yellow, 100f);
    }
}

```

![](..\img\quaternion.png)

根据公式来算四元数旋转：
$$
{\overline q =({\vec sin(\frac \theta 2)}\vec n,cos(\frac \theta 2))} \\
{\overline p^{,} =\overline q\overline p\overline q^{*}   }
$$
这里的 $\overline p^*$写成 $\overline p^{-1}$也是可以的，因为根据前述对逆的计算，对于此时模为1的 $\overline p$来说，它们结果相同的。

将四元数的四个值分别计为：（w,x,y,z），*unity中的四元数中的四个数字是（x,y,z,w），不影响下面的计算过程。*

那么绕着Y轴旋转90度的四元数就是**q = (√2/2 , 0 ， √2/2 ， 0)**；

*unity中这个Quaternion.Euler(0,90,0)打debug的话是（0，√2/2 , 0 ， √2/2 ），因为排列顺序是（x,y,z,w），不影响下面的计算过程*）.



q = (√2/2 , 0 ， √2/2 ， 0)；

v，将v向量扩充为四元数（0，v）,也就是v = （0 , 0，0 , 10）；

 $\overline p^{-1}$四元数q的逆，求逆过程如下：

- 共轭四元数：q*=(w,-x,-y,-z)，也就是（√2/2 , 0 ， -√2/2 ， 0）
- 四元数的模：N(q) = √(x^2 + y^2 + z^2 +w^2)，即四元数到原点的距离，计算结果为1
- 四元数的逆：q−1=q*/N(q)，也就是q−1 = （√2/2 , 0 ， -√2/2 ， 0）

q * v = q * v * q−1 = (√2/2 , 0 ， √2/2 ， 0) * （0 , 0，0 , 10）*（√2/2 , 0 ， -√2/2 ， 0）；

![img](https://img-blog.csdn.net/20180916163340910?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NhcHJpY29ybjEyNDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

按照上述计算公式： q * v = q * v * q−1

 (√2/2 , 0 ， √2/2 ， 0) * （0 , 0，0 , 10） = （0,5√2，0，5√2）

（0,5√2，0，5√2） * （√2/2 , 0 ， -√2/2 ， 0）=（0,10,0,0）；