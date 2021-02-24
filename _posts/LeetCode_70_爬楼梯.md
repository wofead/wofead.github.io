# 爬楼梯

> 假设你正在爬楼梯。需要 *n* 阶你才能到达楼顶。
>
> 每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？
>
> **注意：**给定 *n* 是一个正整数。

## 结题思路

1. 斐波那契数列类似。
2. 一阶为1，二阶为2，三阶是1和2之和。
3. 4阶是2与3之和。


## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class ClimbStairs
    {
        public int Solve(int n)
        {
            if (n == 1) return 1;
            if (n == 2) return 2;
            int k = 1, j = 2;
            for (int i = 3; i <= n; i++)
            {
                int temp = k + j;
                k = j;
                j = temp;
            }
            return j;
        }
    }
}

```





