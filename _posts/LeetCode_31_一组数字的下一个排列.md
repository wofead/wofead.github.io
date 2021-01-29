# 一组数字的下一个排列

> 实现获取 下一个排列 的函数，算法需要将给定数字序列重新排列成字典序中下一个更大的排列。
>
> 如果不存在下一个更大的排列，则将数字重新排列成最小的排列（即升序排列）。
>
> **提示：**
>
> - `1 <= nums.length <= 100`
> - `0 <= nums[i] <= 100`

## 结题思路

![](https://assets.leetcode-cn.com/solution-static/31/31.gif)

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CodeExercise
{
    public class ArrayNextPermutation
    {
        public static void Solve(int[] nums)
        {
            int len = nums.Length;
            if (len == 1)
            {
                return;
            }
            int left = len - 2, right = len - 1;
            while (left >= 0)
            {
                if (nums[left] < nums[left + 1])
                {
                    break;
                }
                else
                {
                    left--;
                }
            }
            int temp;
            if (left >= 0)
            {
                while (right > 0)
                {
                    if (nums[left] < nums[right])
                    {
                        break;
                    }
                    else
                    {
                        right--;
                    }
                }
                temp = nums[left];
                nums[left] = nums[right];
                nums[right] = temp;
            }
            left++;
            right = len - 1;
            while (left < right)
            {
                temp = nums[right];
                nums[right] = nums[left];
                nums[left] = temp;
                left++;
                right--;
            }
            foreach (var item in nums)
            {
                Console.WriteLine(item);
            }
        }
    }
}

```
