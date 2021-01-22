#### 三数之和最接近目标值

> 给定一个包括 n 个整数的数组 nums 和 一个目标值 target。找出 nums 中的三个整数，使得它们的和与 target 最接近。返回这三个数的和。假定每组输入只存在唯一答案。
>
> - `3 <= nums.length <= 10^3`
>
> - `-10^3 <= nums[i] <= 10^3`
> - `-10^4 <= target <= 10^4`
>

## 结题思路

1. 首先对数组排序，然后第一个数字只用遍历一次`i > 0 && nums[i] == nums[i - 1]`
2. 使用前后指针，之和小于0，则前指针动，大于0，后指针动。
3. 判断谁更相近。
4. 然后前指针动。

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CodeExercise
{
    public class ThreeSumClosest
    {
        public static int Solve(int[] nums, int target)
        {
            int result = nums[0] + nums[1] + nums[2];
            Array.Sort(nums);
            int len = nums.Length;
            for (int i = 0; i < len; i++)
            {
                if (i > 0 && nums[i] == nums[i - 1])
                {
                    continue;
                }
                int f = i + 1, b = len - 1;
                while (f < b)
                {
                    int value = nums[i] + nums[f] + nums[b];
                    if (value < target)
                    {
                        f++;
                    }
                    else if (value > target)
                    {
                        b--;
                    }
                    else
                    {
                        return target;
                    }
                    if (Math.Abs(result - target) > Math.Abs(value - target))
                    {
                        result = value;
                    }
                }
            }
            return result;
        }
    }
}

```
