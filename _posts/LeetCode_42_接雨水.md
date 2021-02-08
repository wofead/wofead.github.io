# 缺失的第一个正整数

> 给定一个数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。
>
> candidates 中的每个数字在每个组合中只能使用一次。
>
> 1. 所有数字（包括目标数）都是正整数。
> 2. 解集不能包含重复的组合。 

## 结题思路

1. 将负数转换为超过len的整数
2. 将nums[i]的正值的位置的值置为这个值的负数
3. 判断哪个位置不为负数，那么就缺少这个值

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class ArrayTrap
    {
        public int Solve(int[] height)
        {
            int n = height.Length;
            if (n == 0 || n == 1)
            {
                return 0;
            }
            int result = 0, left = 0, right = 1;
            while (right < n)
            {
                int tempSum = 0;
                int rightMax = 0;
                while (right < n - 1 && height[right] < height[left])
                {
                    if (height[right] > height[right - 1] && height[right] < height[right + 1])
                    {

                    }
                    tempSum += height[left] - height[right];
                    right++;
                }
                if (height[right] > height[right - 1])
                {
                    if (height[left] > height[right])
                    {
                        tempSum -= (height[left] - height[right]) * (right - left - 1);
                    }
                    result += tempSum;
                }
                left = right;
                right = left + 1;
            }
            return result;
        }
    }
}

```

## 解2

1. 将数字放到它应该在的index位置处
2. 直到`value >= 0 && value < len && value != i &&nums[value]!=value+1`

```c#
public int Solve2(int[] nums)
{
    int len = nums.Length;
    for (int i = 0; i < len; i++)
    {
        int value = nums[i] - 1;
        while (value >= 0 && value < len && value != i)
        {
            int temp = nums[value];
            if (temp == value + 1)
            {
                break;
            }
            nums[value] = value + 1;
            nums[i] = temp;
            value = temp - 1;
        }
    }
    for (int i = 0; i < len; i++)
    {
        if (nums[i] != i + 1)
        {
            return i + 1;
        }
    }
    return len + 1;
}
```

