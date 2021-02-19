# 螺旋矩阵

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
    public class DoubleArraySpiralOrder
    {
        public IList<int> Solve(int[][] matrix)
        {
            IList<int> res = new List<int>();
            int rawNum = matrix.Length;
            int colNum = matrix[0].Length;
            int rawCount = 0;
            int colCount = 0;
            int turn = 0;
            int x = 0, y = 0;
            while (rawCount < rawNum)
            {
                bool isOperateY = turn % 2 == 0 ? true : false;
                int value = turn % 4 > 1 ? -1 : 1;
                int count = 0;
                if (isOperateY)
                {
                    count = colNum - turn / 3 - 1;
                }
                else
                {
                    count = rawNum - turn / 3 - 1;
                }
                for (int i = 0; i < count; i++)
                {
                    res.Add(matrix[x][y]);
                    if (isOperateY)
                    {
                        y += value;
                    }
                    else
                    {
                        x += value;
                    }
                }
                if (isOperateY)
                {
                    colCount++;
                }
                else
                {
                    rawCount++;
                }
                turn++;
            }
            return res;
        }
    }
}

```



