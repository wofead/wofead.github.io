# 在排序数组中查找元素的第一个和最后一个位置

> 给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。
>
> 你可以假设数组中无重复元素。

## 结题思路

1. 二分法查找目标值
2. 当没有找到时，right大于left，返回right
3. left大于right，返回left。

## 解：

```c#
using System;

namespace CodeExercise
{
    public class ArraySearchInsert
    {
        public static int Solve(int[] nums, int target)
        {
            int len = nums.Length;
            int left = 0, right = len - 1;
            while (left <= right)
            {
                int mid = (left + right) / 2;
                int midValue = nums[mid];
                if (midValue == target)
                {
                    return mid;
                }
                else if (midValue < target)
                {
                    left = mid + 1;
                }
                else
                {
                    right = mid - 1;
                }
            }
            return Math.Max(left, right);
        }
    }
}

```







