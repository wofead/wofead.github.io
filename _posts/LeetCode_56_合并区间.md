# 合并区间

> 以数组 intervals 表示若干个区间的集合，其中单个区间为 intervals[i] = [start_i, end_i] 。请你合并所有重叠的区间，并返回一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间。
>

## 结题思路

1. 首先对intervals数组进行排序
2. 判断相邻的两个数组有，后面的那个的index 0位置的值是否小于等于前面的index 1的值
3. 小于等于合并
4. 否则另起一个新的item

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



