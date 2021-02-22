# 不同路径II

> 一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为“Start” ）。
>
> 机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为“Finish”）。
>
> 现在考虑网格中有障碍物。那么从左上角到右下角将会有多少条不同的路径？
>

## 结题思路

1. 动态规划
2. 但是是记录型的动态规划
3. f(i,j)=f(i−1,j)+f(i,j−1)，边际如果不存在障碍物就为1，否则障碍物后面的都为0
4. 算出到每个点有多少种方式


## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class DoubleArrayUniquePathsWithObstacles
    {
        public int Solve(int[][] obstacleGrid)
        {
            int m = obstacleGrid.Length;
            int n = obstacleGrid[0].Length;
            if (obstacleGrid[0][0] == 1 || obstacleGrid[m - 1][n - 1] == 1) return 0;
            bool isExitObstacle = false;
            for (int i = 1; i < m; i++)
            {
                if (isExitObstacle) obstacleGrid[i][0] = 0;
                else if (obstacleGrid[i][0] == 1)
                {
                    isExitObstacle = true;
                    obstacleGrid[i][0] = 0;
                }
                else obstacleGrid[i][0] = 1;
            }
            isExitObstacle = false;
            for (int i = 1; i < n; i++)
            {
                if (isExitObstacle) obstacleGrid[0][i] = 0;
                else if (obstacleGrid[0][i] == 1)
                {
                    isExitObstacle = true;
                    obstacleGrid[0][i] = 0;
                }
                else obstacleGrid[0][i] = 1;
            }
            for (int i = 1; i < m; i++)
            {
                for (int j = 1; j < n; j++)
                {
                    if (obstacleGrid[i][j] == 1)
                    {
                        obstacleGrid[i][j] = 0;
                        continue;
                    }
                    else
                    {
                        obstacleGrid[i][j] = obstacleGrid[i - 1][j] + obstacleGrid[i][j - 1];
                    }
                }
            }
            return obstacleGrid[m - 1][n - 1];
        }
    }
}

```



