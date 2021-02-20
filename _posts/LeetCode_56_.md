# 跳跃游戏

> 给定一个非负整数数组 `nums` ，你最初位于数组的 **第一个下标** 。
>
> 数组中的每个元素代表你在该位置可以跳跃的最大长度。
>
> 判断你是否能够到达最后一个下标。

## 结题思路

1. maxPos的位置上的值不为0



## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class DoubleArrayMerge
    {
        public int[][] Solve(int[][] intervals)
        {
            List<int[]> res = new List<int[]>();
            Array.Sort(intervals, (arr1, arr2) => { return arr1[0] < arr2[0] ? -1 : 1; });
            int length = intervals.Length;
            int[] item = new int[2];
            item[0] = intervals[0][0];
            item[1] = intervals[0][1];
            for (int i = 1; i < length; i++)
            {
                int[] temp = intervals[i];
                if (temp[0] <= item[1])
                {
                    item[1] = Math.Max(item[1], temp[1]);
                }
                else
                {
                    res.Add(item);
                    item = new int[2];
                    item[0] = intervals[i][0];
                    item[1] = intervals[i][1];
                }
            }
            res.Add(item);
            return res.ToArray();
        }
    }
}

```



