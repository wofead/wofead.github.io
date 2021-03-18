# 螺旋矩阵II

> 给你一个正整数 `n` ，生成一个包含 `1` 到 `n2` 所有元素，且元素按顺时针顺序螺旋排列的 `n x n` 正方形矩阵 `matrix` 。

## 结题思路

1. 可以将矩阵看成若干层，首先输出最外层的元素，其次输出次外层的元素，直到输出最内层的元素。	

   定义矩阵的第 k 层是到最近边界距离为 k 的所有顶点。例如，下图矩阵最外层元素都是第 1 层，次外层元素都是第 2 层，剩下的元素都是第 3 层。

2. 对于每层，从左上方开始以顺时针的顺序遍历所有元素。

3. 遍历完当前层的元素之后，将 leftTop增加 1，将 rightBottom 减少 1，进入下一层继续遍历，直到遍历完所有元素为止。

4. 这里要注意当leftTop==rightBottom 时是奇数，加上这个数字就ok了。


## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class GenerateMatrix
    {
        public int[][] Solve(int n)
        {
            int[][] res = new int[n][];
            for (int i = 0; i < n; i++)
            {
                res[i] = new int[n];
            }
            int turn = n % 2 == 1 ? n / 2 + 1 : n / 2;
            int value = 1;
            int leftTop = 0;
            int rightBottom = n - 1;
            int cir = 0;
            while (turn > 0)
            {
                if (rightBottom == leftTop)
                {
                    res[rightBottom][leftTop] = value;
                    break;
                }
                for (int i = 0; i < rightBottom - leftTop; i++)
                {
                    res[cir][i + leftTop] = value;
                    value++;
                }
                for (int i = 0; i < rightBottom - leftTop; i++)
                {
                    res[i + leftTop][n - cir - 1] = value;
                    value++;
                }
                for (int i = 0; i < rightBottom - leftTop; i++)
                {
                    res[n - cir - 1][rightBottom - i] = value;
                    value++;
                }
                for (int i = 0; i < rightBottom - leftTop; i++)
                {
                    res[rightBottom - i][cir] = value;
                    value++;
                }
                leftTop++;
                rightBottom--;
                turn--;
                cir++;
            }
            return res;
        }
    }
}

```



