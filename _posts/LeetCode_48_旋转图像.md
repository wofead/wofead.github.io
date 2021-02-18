# 旋转图像

> 给定一个 n × n 的二维矩阵 matrix 表示一个图像。请你将图像顺时针旋转 90 度。
>
> 你必须在 原地 旋转图像，这意味着你需要直接修改输入的二维矩阵。请不要 使用另一个矩阵来旋转图像。
>

## 结题思路

1. 旋转 n / 2轮

2. 每次旋转四个

3. 找到锚点进行旋转
   $$
   temp =matrix[row][col]\\
   matrix[row][col] =matrix[n−col−1][row]\\
   matrix[n−col−1][row] =matrix[n−row−1][n−col−1]\\
   matrix[n−row−1][n−col−1]=matrix[col][n−row−1]\\
   matrix[col][n−row−1]=temp\\
   $$
   

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class DoubleArrayRotate
    {
        public void Solve(int[][] matrix)
        {
            int n = matrix.Length;
            for (int i = 0; i < n / 2; i++)
            {
                for (int j = 0; j < 3; j++)
                {
                    bool flag = j % 2 == 0 ? true : false;
                    int point = j == 0 ? 1 : -1;
                    int[] markPos = { i, i };
                    int[] pos = GetStartPos(n, i, j);
                    for (int k = 1; k < n - 2 * i; k++)
                    {
                        int x2, y2;
                        if (flag)
                        {
                            x2 = pos[0] + k * point;
                            y2 = pos[1];
                        }
                        else
                        {
                            x2 = pos[0];
                            y2 = pos[1] + k * point;
                        }
                        int x1 = markPos[0];
                        int y1 = markPos[1] + k;
                        int temp = matrix[x1][y1];
                        matrix[x1][y1] = matrix[x2][y2];
                        matrix[x2][y2] = temp;
                    }
                }
            }
        }

        public int[] GetStartPos(int n, int turn, int index)
        {
            int[] pos = new int[2];
            if (index == 0)
            {
                pos[0] = turn;
                pos[1] = n - turn - 1;
            }
            else if (index == 1)
            {
                pos[0] = n - turn - 1;
                pos[1] = n - turn - 1;
            }
            else
            {
                pos[0] = n - turn - 1;
                pos[1] = turn;
            }
            return pos;
        }
    }
}

```

## 解2

上下 翻转，对角翻转
$$
matrix[row][col] =  matrix[n−row−1][col]\\
matrix[row][col] == matrix[col][row]\\
matrix[col][n−row−1]=matrix[row][col]
$$


```java
class Solution {
    public void rotate(int[][] matrix) {
        int n = matrix.length;
        // 水平翻转
        for (int i = 0; i < n / 2; ++i) {
            for (int j = 0; j < n; ++j) {
                int temp = matrix[i][j];
                matrix[i][j] = matrix[n - i - 1][j];
                matrix[n - i - 1][j] = temp;
            }
        }
        // 主对角线翻转
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < i; ++j) {
                int temp = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = temp;
            }
        }
    }
}

```

