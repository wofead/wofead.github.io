# 删除排序链表中的重复元素 II

> 给定一个排序链表，删除所有含有重复数字的节点，只保留原始链表中 *没有重复出现* 的数字。
>

## 结题思路

使用快慢指针，判断是否需要删除快指针指向的那个节点。

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class LargestRectangleArea
    {
        public int Solve(int[] heights)
        {
            int length = heights.Length;
            if (length == 1)
            {
                return heights[0];
            }
            int mid = length / 2;
            int left = mid, right = mid + 1;
            int maxArea = heights[mid];
            int minHeight = heights[mid];
            while (left >= 0 && right < length)
            {
                int width = right - left + 1;
                int height = Math.Min(Math.Min(heights[left], heights[right]), minHeight);
                maxArea = Math.Max(maxArea, width * height);
            }
            return 0;
        }
    }
}

```

## 

