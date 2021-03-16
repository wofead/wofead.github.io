# 矩阵中最大的矩形

> 给定一个仅包含 `0` 和 `1` 、大小为 `rows x cols` 的二维二进制矩阵，找出只包含 `1` 的最大矩形，并返回其面积。

## 结题思路

上一题的柱子问题，这个题就是把矩阵分为n列柱子问题来解决。

首先算出每个点到左边的最大连续为1的大小，然后算出这一列能形成的最大面积。

就是上一题的单调栈问题了。

单调栈。栈中存放的元素具有单调性，这就是经典的数据结构「单调栈」了。

这里我们需要转换一下思维，意思就是以每个柱子为中心，它为最大的起点和终点在哪里，然后算出这个面积，所有这些算出来的进行比较大小，最大的那个就是满足条件的面积大小。

从左边开始算起，使用栈的思维，当准备进栈的值一直大于等于栈顶的值的时候就可以进栈，否则出栈，直到栈顶元素小于等于这个准备进栈的值。

**记录左侧柱子的编号。**

右侧再来一遍，就是准备进栈的值都大于等于栈顶的值可以进栈，否则出栈，直到栈顶元素小于等于这个准备进栈的值。

**记录右侧柱子的编号。**

用右侧柱子的编号减去左侧柱子的编号，然后乘以高度，就可以得到，这根柱子作为最矮的那根柱子的面积了，算出所有然后进行比较大小。

这里可以一次算出左侧柱子和右侧柱子的编号。

我们在对位置 i 进行入栈操作时，确定了它的左边界。从直觉上来说，与之对应的我们在对位置 i 进行出栈操作时可以确定它的右边界！

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class LargestRectangleArea
    {
        //单调栈，高度扩展
        public int Solve(int[] heights)
        {
            int n = heights.Length;
            int[] left = new int[n];
            int[] right = new int[n];
            Array.Fill(right, n);
            Stack<int> monoStack = new Stack<int>();
            for (int i = 0; i < n; i++)
            {
                while (!(monoStack.Count == 0) && heights[monoStack.Peek()] >= heights[i])
                {
                    right[monoStack.Peek()] = i;
                    monoStack.Pop();
                }
                left[i] = ((monoStack.Count == 0) ? -1 : monoStack.Peek());
                monoStack.Push(i);
            }
            int ans = 0;
            for (int i = 0; i < n; i++)
            {
                ans = Math.Max(ans, (right[i] - left[i] - 1) * heights[i]);
            }
            return ans;
        }

        public int SolveDoubleIterator(int[] heights)
        {
            int maxArea = 0;
            int length = heights.Length;
            for (int i = 0; i < length; i++)
            {
                if (heights[i] == 0)
                {
                    continue;
                }
                int minHeight = heights[i];
                for (int j = i; j < length; j++)
                {
                    if (heights[j] == 0)
                    {
                        break;
                    }
                    int width = j - i + 1;
                    int height = Math.Min(heights[j], minHeight);
                    minHeight = height;
                    maxArea = Math.Max(maxArea, width * height);
                }
            }
            return maxArea;
        }
    }
}

```
