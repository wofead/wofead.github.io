# 矩阵置零

> 给定一个 *m* x *n* 的矩阵，如果一个元素为 0，则将其所在行和列的所有元素都设为 0。请使用**[原地](http://baike.baidu.com/item/原地算法)**算法**。**
>

## 结题思路1

1. 使用额外的m+n的空间，记录被标志的行和列
2. 再次遍历矩阵，置0

## 结题思路2

1. 使用第一行和第一列作为标志空间
2. 然后使用额外的一位，来标记第一列是否为被标记的。

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class DoubleArraySetZeroes
    {
        public void Solve(int[][] matrix)
        {
            int rawLen = matrix.Length;
            int colLen = matrix[0].Length;
            bool[] rawFlags = new bool[rawLen];
            bool[] colFlags = new bool[colLen];
            for (int i = 0; i < rawLen; i++)
            {
                for (int j = 0; j < colLen; j++)
                {
                    if (matrix[i][j] == 0)
                    {
                        rawFlags[i] = true;
                        colFlags[j] = true;
                    }
                }
            }

            for (int i = 0; i < rawLen; i++)
            {
                bool rawFlag = rawFlags[i];
                for (int j = 0; j < colLen; j++)
                {
                    if (rawFlag || colFlags[j]) matrix[i][j] = 0;
                }
            }
        }

        public void OfficialSolve(int[][] matrix)
        {
            int rawLen = matrix.Length;
            int colLen = matrix[0].Length;
            bool isCol = false;
            for (int i = 0; i < rawLen; i++)
            {
                if (matrix[i][0] == 0) isCol = true;
                for (int j = 1; j < colLen; j++)
                {
                    if (matrix[i][j] == 0)
                    {
                        matrix[i][0] = 0;
                        matrix[0][j] = 0;
                    }
                }
            }

            for (int i = 1; i < rawLen; i++)
            {
                for (int j = 1; j < colLen; j++)
                {
                    if (matrix[i][0] == 0 || matrix[0][j] == 0) matrix[i][j] = 0;
                }
            }
            for (int i = 0; i < colLen; i++)
            {
                if (matrix[0][0] == 0) matrix[0][i] = 0;
            }

            for (int i = 0; i < rawLen; i++)
            {
                if (isCol) matrix[i][0] = 0;
            }
        }
    }
}

```





