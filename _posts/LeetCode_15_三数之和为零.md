#### 三数之和为0

> 给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有和为 0 且不重复的三元组。
>
> **注意：**答案中不可以包含重复的三元组。
>
>  `0 <= nums.length <= 3000`
>
> - `-105 <= nums[i] <= 105`
>

## 结题思路

1. 首先对数组排序，然后第一个数字只用遍历一次`i > 0 && nums[i] == nums[i - 1]`
2. 使用前后指针，之和小于0，则前指针动，大于0，后指针动。
3. 如果等于零，要判断，是否和之前的满足条件的第二个数值是否相同
4. 然后前指针动。

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class ThreeNumSumZero
    {
        public static IList<IList<int>> Solve(int[] nums)
        {
            Array.Sort(nums);
            int len = nums.Length;
            IList<IList<int>> result = new List<IList<int>>();
            int f, b;
            for (int i = 0; i < len - 2; i++)
            {
                if (i > 0 && nums[i] == nums[i - 1])
                {
                    continue;
                }
                f = i + 1;
                b = len - 1;
                int lastZeroF = -1;
                while (f < b)
                {
                    if (nums[i] + nums[f] + nums[b] < 0)
                    {
                        f++;
                    }
                    else if (nums[i] + nums[f] + nums[b] > 0)
                    {
                        b--;
                    }
                    else
                    {
                        f++;
                        if (lastZeroF != -1 && nums[lastZeroF] == nums[f - 1]) continue;
                        IList<int> item = new List<int>();
                        item.Add(nums[i]);
                        item.Add(nums[f - 1]);
                        item.Add(nums[b]);
                        result.Add(item);
                        lastZeroF = f - 1;
                    }
                }
            }
            return result;
        }
    }
}


```
