# 不同路径

> 一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为 “Start” ）。
>
> 机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。
>
> 问总共有多少条不同的路径？
>

## 结题思路1

1. 递归的方式，直到边际就加1，否则分裂

## 结题思路2

1. 和1一样都是动态规划
2. 但是是记录型的动态规划
3. f(i,j)=f(i−1,j)+f(i,j−1)，边际都是1
4. 算出到每个点有多少种方式

## 解题思路3

1. 数学组合
2. 从左上角到右下角的过程中，我们需要移动 m+n-2 次，其中有 m-1次向下移动，n-1 次向右移动。
3. 就等于从 m+n-2次移动中选择 m-1次向下移动的方案数，即组合数。


## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class GraphUniquePaths
    {
        int res = 0;
        public int Solve(int m, int n)
        {
            Move(1, 1, m, n);
            return res;
        }

        public void Move(int raw, int col, int m, int n)
        {
            if (raw == m || col == n)
            {
                res++;
                return;
            }
            if (raw < m)
            {
                Move(raw + 1, col, m, n);
            }
            if (col < n)
            {
                Move(raw, col + 1, m, n);
            }
        }
        //f(i,j)=f(i−1,j)+f(i,j−1)
        public int OfficialSolve1(int m, int n)
        {
            int[,] f = new int[m, n];
            for (int i = 0; i < m; ++i)
            {
                f[i, 0] = 1;
            }
            for (int j = 0; j < n; ++j)
            {
                f[0, j] = 1;
            }
            for (int i = 1; i < m; ++i)
            {
                for (int j = 1; j < n; ++j)
                {
                    f[i, j] = f[i - 1, j] + f[i, j - 1];
                }
            }
            return f[m - 1, n - 1];
        }
        /// <summary>
        /// 从左上角到右下角的过程中，我们需要移动 m+n-2m+n−2 次，
        /// 其中有 m-1m−1 次向下移动，n-1n−1 次向右移动。因此路径的总数，
        /// 就等于从 m+n-2m+n−2 次移动中选择 m-1m−1 次向下移动的方案数，即组合数

        /// </summary>
        /// <param name="m"></param>
        /// <param name="n"></param>
        /// <returns></returns>
        public int OfficialSove2(int m, int n)
        {
            long ans = 1;
            for (int x = n, y = 1; y < m; ++x, ++y)
            {
                ans = ans * x / y;
            }
            return (int)ans;
        }
    }
}

```



