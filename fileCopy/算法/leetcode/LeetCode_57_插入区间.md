# 插入区间

> 给你一个 **无重叠的** *，*按照区间起始端点排序的区间列表。
>
> 在列表中插入一个新的区间，你需要确保列表中的区间仍然有序且不重叠（如果有必要的话，可以合并区间）。

## 结题思路

1. 标志是否添加了newInternal
2. 判断是否进入newInternal区间
3. 没有进入就直接添加temp
4. 否则更新newInternal的值
5. 如果temp的0大于newInternal的1，添加temp和nerInternal

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class DoubleArrayInsert
    {
        public int[][] Solve(int[][] intervals, int[] newInterval)
        {
            List<int[]> res = new List<int[]>();
            int length = intervals.Length;
            bool isAddNewInterval = false;
            for (int i = 0; i < length; i++)
            {
                int[] temp = intervals[i];
                if (isAddNewInterval || temp[1] < newInterval[0])
                {
                    res.Add(temp);
                    continue;
                }
                if (temp[0] > newInterval[1])
                {
                    res.Add(newInterval);
                    res.Add(temp);
                    isAddNewInterval = true;
                }
                else if (temp[1] >= newInterval[0])
                {
                    newInterval[0] = Math.Min(newInterval[0], temp[0]);
                    newInterval[1] = Math.Max(newInterval[1], temp[1]);
                }
            }
            if (!isAddNewInterval)
            {
                res.Add(newInterval);
            }
            return res.ToArray();
        }
    }
}

```



