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
    public class ArrayCanJump
    {
        public bool Solve(int[] nums)
        {
            int length = nums.Length;
            if (length == 1)
            {
                return true;
            }
            if (nums[0] == 0)
            {
                return false;
            }
            int maxPos = nums[0];
            int remindStep = nums[0];
            for (int i = 1; i < length && maxPos < length - 1; i++)
            {
                remindStep--;
                if (maxPos == i && nums[i] == 0)
                {
                    return false;
                }
                if (remindStep < nums[i])
                {
                    remindStep = nums[i];
                    maxPos = i + nums[i];
                }
            }
            return true;
        }
    }
}

```



