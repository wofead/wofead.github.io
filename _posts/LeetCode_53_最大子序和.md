# 最大子序和

> 给定一个整数数组 `nums` ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

## 结题思路

1. 根据res的值，来更新maxRes。

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class ArrayMaxSubArray
    {
        public int Solve(int[] nums)
        {
            int length = nums.Length;
            int right = 1;
            int maxRes = nums[0];
            int res = nums[0];
            while (right < length)
            {
                if (res <= 0)
                {
                    if (nums[right] > res)
                    {
                        res = nums[right];
                        maxRes = Math.Max(maxRes, res);
                    }
                    right++;
                }
                else
                {
                    if (nums[right] >= 0)
                    {
                        res += nums[right];
                        maxRes = Math.Max(maxRes, res);
                        right++;
                    }
                    else
                    {
                        if (res + nums[right] <= 0)
                        {
                            res = 0;
                        }
                        else
                        {
                            res += nums[right];
                        }
                        right++;
                    }
                }
            }
            return maxRes;
        }
    }
}

```



