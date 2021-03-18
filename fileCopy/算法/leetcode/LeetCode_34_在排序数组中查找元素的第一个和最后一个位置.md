# 在排序数组中查找元素的第一个和最后一个位置

> 给定一个按照升序排列的整数数组 nums，和一个目标值 target。找出给定目标值在数组中的开始位置和结束位置。
>
> 如果数组中不存在目标值 target，返回 [-1, -1]。
>
> **提示：**
>
> - `0 <= nums.length <= 105`
> - `-109 <= nums[i] <= 109`
> - `nums` 是一个非递减数组
> - `-109 <= target <= 109`

## 结题思路

1. 二分法查找目标值
2. 左右指针确定范围

## 解：

```c#
namespace CodeExercise
{
    public class ArraySearchRange
    {
        public static int[] Solve(int[] nums, int target)
        {
            int[] result = new int[2] { -1, -1 };
            int len = nums.Length;
            if (len == 0)
            {
                return result;
            }
            int left = 0, right = len - 1, mid = -1;
            while (left <= right)
            {
                mid = (left + right) / 2;
                int midValue = nums[mid];
                if (target == midValue)
                {
                    break;
                }
                else if (target < midValue)
                {
                    right = mid - 1;
                }
                else left = mid + 1;
            }
            if (target != nums[mid])
            {
                return result;
            }
            left = right = mid;
            result[0] = mid;
            result[1] = mid;
            while (left > 0 || right < len - 1)
            {
                if (left > 0)
                {
                    if (nums[left - 1] == target)
                    {
                        left--;
                        result[0] = left;
                    }
                    else
                    {
                        result[0] = left;
                        left = -1;
                    }
                }
                if (right < len - 1)
                {
                    if (nums[right + 1] == target)
                    {
                        right++;
                        result[1] = right;
                    }
                    else
                    {
                        result[1] = right;
                        right = len;
                    }
                }
            }
            return result;
        }
    }
}

```
