# 最小路径和

> 给定一个包含非负整数的 `*m* x *n*` 网格 `grid` ，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。
>
> **说明：**每次只能向下或者向右移动一步。

## 结题思路

1. 动态规划
2. 但是是记录型的动态规划
3. 从左上到右下，一直保留到点最近的那个路径值


## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class DoubleArrayMinPathSum
    {
        public int Solve(int[][] grid)
        {
            int m = grid.Length;
            int n = grid[0].Length;
            for (int i = 1; i < m; i++)
            {
                grid[i][0] += grid[i - 1][0];
            }
            for (int i = 1; i < n; i++)
            {
                grid[0][i] += grid[0][i - 1];
            }
            for (int i = 1; i < m; i++)
            {
                for (int j = 1; j < n; j++)
                {
                    grid[i][j] += Math.Min(grid[i - 1][j], grid[i][j - 1]);
                }
            }
            return grid[m - 1][n - 1];
        }
    }
}

```



