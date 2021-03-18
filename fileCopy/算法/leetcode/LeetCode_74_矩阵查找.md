# 矩阵查找

> 编写一个高效的算法来判断 m x n 矩阵中，是否存在一个目标值。该矩阵具有如下特性：
>
> * 每行中的整数从左到右按升序排列。
> * 每行的第一个整数大于前一行的最后一个整数。

## 结题思路1

1. 首先对行进行二分查找，再对列进行二分查找

## 结题思路2

1. 将矩阵看成一行的数组，对其进行二分查找

   ```c#
   int left = 0, right = m * n - 1;
   pivotIdx = (left + right) / 2;
   pivotElement = matrix[pivotIdx / n][pivotIdx % n];
   
   ```

   

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class DoubleArraySearchMatrix
    {
        public bool Solve(int[][] matrix, int target)
        {
            int rawLen = matrix.Length;
            int colLne = matrix[0].Length;
            int left = 0, right = colLne - 1, top = 0, bottom = rawLen - 1;
            while (left <= right && top <= bottom)
            {
                int rawMid = (top + bottom) / 2;
                int colMid = (left + right) / 2;
                int value = matrix[rawMid][colMid];
                if (value == target)
                {
                    return true;
                }
                if (target < matrix[rawMid][0]) bottom = rawMid - 1;
                else if (target > matrix[rawMid][colLne - 1]) top = rawMid + 1;
                else
                {
                    top = rawMid;
                    bottom = rawMid;
                    while (left <= right)
                    {
                        int mid = (left + right) / 2;
                        if (matrix[rawMid][mid] == target) return true;
                        else if (matrix[rawMid][mid] > target) right = mid - 1;
                        else left = mid + 1;
                    }
                }
            }
            return false;
        }

        public bool Official(int[][] matrix, int target)
        {
            int m = matrix.Length;
            if (m == 0) return false;
            int n = matrix[0].Length;

            // 二分查找
            int left = 0, right = m * n - 1;
            int pivotIdx, pivotElement;
            while (left <= right)
            {
                pivotIdx = (left + right) / 2;
                pivotElement = matrix[pivotIdx / n][pivotIdx % n];
                if (target == pivotElement) return true;
                else
                {
                    if (target < pivotElement) right = pivotIdx - 1;
                    else left = pivotIdx + 1;
                }
            }
            return false;
        }
    }
}

```





