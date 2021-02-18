# Pow(x,n)

> 实现 pow(x,n)，即计算 x 的 n 次幂函数（即，x^n）。
>
> - `-100.0 < x < 100.0`
> - `-231 <= n <= 231-1`
> - `-104 <= xn <= 104`

## 结题思路

1. 倍数放大法。
2. 使用负数的方法避免越界。

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class MyPow
    {
        public double Solve(double x, int n)
        {
            double res = 1;
            if (n == 0)
            {
                return 1;
            }
            bool isNeg = n < 0 ? true : false;
            if (!isNeg) n = -n;
            while (n <= -1)
            {
                double tempRes = x;
                long tempN = -1;
                while (tempN * 2 >= n)
                {
                    tempN *= 2;
                    tempRes *= tempRes;
                }
                n -= (int)tempN;
                res *= tempRes;
            }
            if (isNeg)
            {
                res = 1 / res;
            }
            return res;
        }
    }
}

```



