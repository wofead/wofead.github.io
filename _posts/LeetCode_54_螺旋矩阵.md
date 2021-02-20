# 螺旋矩阵

> 给你一个 `m` 行 `n` 列的矩阵 `matrix` ，请按照 **顺时针螺旋顺序** ，返回矩阵中的所有元素。

## 结题思路

1. 可以将矩阵看成若干层，首先输出最外层的元素，其次输出次外层的元素，直到输出最内层的元素。	

   定义矩阵的第 k 层是到最近边界距离为 k 的所有顶点。例如，下图矩阵最外层元素都是第 1 层，次外层元素都是第 2 层，剩下的元素都是第 3 层。

2. 对于每层，从左上方开始以顺时针的顺序遍历所有元素。假设当前层的左上角位于 (top,left)，右下角位于(bottom,right)，按照如下顺序遍历当前层的元素。

   1. 从左到右遍历上侧元素，依次为 (top,left) 到(top,right)。
   2. 从上到下遍历右侧元素，依次为 (*top*+1,*right*) 到 (*bottom*,*right*)。
   3. 如果 left<right 且 top<bottom，则从右到左遍历下侧元素，依次为 (bottom,right−1) 到 (bottom,left+1)，以及从下到上遍历左侧元素，依次为 (bottom,left) 到 (top+1,left)。

3. 遍历完当前层的元素之后，将 left 和 top 分别增加 11，将 right 和 bottom 分别减少 11，进入下一层继续遍历，直到遍历完所有元素为止。


## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class DoubleArraySpiralOrder
    {
        public IList<int> Solve(int[][] matrix)
        {
            IList<int> result = new List<int>();
            if (matrix == null || matrix.Length == 0) return result;
            int left = 0;
            int right = matrix[0].Length - 1;
            int top = 0;
            int bottom = matrix.Length - 1;
            int numEle = matrix.Length * matrix[0].Length;
            while (numEle >= 1)
            {
                for (int i = left; i <= right && numEle >= 1; i++)
                {
                    result.Add(matrix[top][i]);
                    numEle--;
                }
                top++;
                for (int i = top; i <= bottom && numEle >= 1; i++)
                {
                    result.Add(matrix[i][right]);
                    numEle--;
                }
                right--;
                for (int i = right; i >= left && numEle >= 1; i--)
                {
                    result.Add(matrix[bottom][i]);
                    numEle--;
                }
                bottom--;
                for (int i = bottom; i >= top && numEle >= 1; i--)
                {
                    result.Add(matrix[i][left]);
                    numEle--;
                }
                left++;
            }
            return result;
        }
    }
}

```



