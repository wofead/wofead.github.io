# 跳跃游戏II

> 给定一个非负整数数组，你最初位于数组的第一个位置。
>
> 数组中的每个元素代表你在该位置可以跳跃的最大长度。
> 
> 你的目标是使用最少的跳跃次数到达数组的最后一个位置。
> 

## 结题思路

1. 动态规划

2. #### 正向查找可到达的最大位置

3. 如果我们「贪心」地进行正向查找，每次找到可到达的最远位置，就可以在线性时间内得到最少的跳跃次数。

4. ![](https://assets.leetcode-cn.com/solution-static/45/45_fig1.png)

5. 在具体的实现中，我们维护当前能够到达的最大下标位置，记为边界。我们从左到右遍历数组，到达边界时，更新边界并将跳跃次数增加 1。

6. 在遍历数组时，我们不访问最后一个元素，这是因为在访问最后一个元素之前，我们的边界一定大于等于最后一个位置，否则就无法跳到最后一个位置了。如果访问最后一个元素，在边界正好为最后一个位置的情况下，我们会增加一次「不必要的跳跃次数」，因此我们不必访问最后一个元素。



## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class ArrayJump
    {
        public int Solve(int[] nums)
        {
            if (nums.Length == 1)
            {
                return 0;
            }
            return Jump(nums, 0, 0, 0);
        }

        public int Jump(int[] nums, int curIndex, int totalStep, int remindStep)
        {

            if (curIndex + nums[curIndex] >= nums.Length - 1)
            {
                return totalStep + 1;
            }
            if (remindStep < 0)
            {
                return int.MaxValue;
            }
            if (remindStep == 0)
            {
                return Jump(nums, curIndex + 1, totalStep + 1, nums[curIndex] - 1);
            }
            return Math.Min(Jump(nums, curIndex + 1, totalStep, remindStep - 1), Jump(nums, curIndex + 1, totalStep + 1, nums[curIndex] - 1));
        }

        public int OfficialSolve(int[] nums)
        {
            int length = nums.Length;
            int end = 0;
            int maxPosition = 0;
            int steps = 0;
            for (int i = 0; i < length - 1; i++)
            {
                maxPosition = Math.Max(maxPosition, i + nums[i]);
                if (i == end)
                {
                    end = maxPosition;
                    steps++;
                }
            }
            return steps;
        }

    }
}

```





