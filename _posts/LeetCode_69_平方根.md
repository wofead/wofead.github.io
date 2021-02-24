# 平方根

> 实现 int sqrt(int x) 函数。
>
> 计算并返回 x 的平方根，其中 x 是非负整数。
>
> 由于返回类型是整数，结果只保留整数的部分，小数部分将被舍去。
>
> **说明:**
>
> - 单词是指由非空格字符组成的字符序列。
>- 每个单词的长度大于 0，小于等于 *maxWidth*。
>   - 输入单词数组 `words` 至少包含一个单词。

## 结题思路1

1. 二分法

## 结题思路2

**牛顿迭代法！！**

牛顿迭代法是一种可以用来快速求解函数零点的方法。

为了叙述方便，我们用 CC表示待求出平方根的那个整数。显然，C 的平方根就是函数：
$$
y = f(x) = x^2-C
$$
的零点。

牛顿迭代法的本质是借助泰勒级数，从初始值开始快速向零点逼近。

牛顿迭代法的本质是借助泰勒级数，从初始值开始快速向零点逼近。我们任取一个 x_0*x*0 作为初始值，在每一步的迭代中，我们找到函数图像上的点$(x_i,f(x_i))$,过该点作一条斜率为该点导数 $f'(x_i)$的直线与横轴的交点记为$x_{i+1}$。$x_{i+1}$相较于 $x_i$而言距离零点更近。在经过多次迭代后，我们就可以得到一个距离零点非常接近的交点。下图给出了从 x0 开始迭代两次，得到 x1 和 x2 的过程。

![](https://assets.leetcode-cn.com/solution-static/69/69_fig1.png)


## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class MySqrt
    {
        //二分法
        public int Solve(int x)
        {
            int l = 0, r = x, ans = -1;
            while (l <= r)
            {
                int mid = l + (r - l) / 2;
                if ((long)mid * mid <= x)
                {
                    ans = mid;
                    l = mid + 1;
                }
                else
                {
                    r = mid - 1;
                }
            }
            return ans;
        }
        //牛顿迭代法是一种可以用来快速求解函数零点的方法。
        //为了叙述方便，我们用 CC 表示待求出平方根的那个整数。显然，CC 的平方根就是函数
        public int Solve1(int x)
        {
            if (x == 0)
            {
                return 0;
            }

            double C = x, x0 = x;
            while (true)
            {
                double xi = 0.5 * (x0 + C / x0);
                if (Math.Abs(x0 - xi) < 1e-7)
                {
                    break;
                }
                x0 = xi;
            }
            return (int)x0;
        }
    }
}

```





