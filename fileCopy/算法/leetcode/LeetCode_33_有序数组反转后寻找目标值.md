# 有序数组反转后寻找目标值

> 升序排列的整数数组 nums 在预先未知的某个点上进行了旋转（例如， [0,1,2,4,5,6,7] 经旋转后可能变为 [4,5,6,7,0,1,2] ）。
>
> 请你在数组中搜索 target ，如果数组中存在这个目标值，则返回它的索引，否则返回 -1 。
>
> **提示：**
>
> - `1 <= nums.length <= 5000`
> - `-10^4 <= nums[i] <= 10^4`
> - `nums` 中的每个值都 **独一无二**
> - `nums` 肯定会在某个点上旋转
> - `-10^4 <= target <= 10^4`

## 结题思路

1. 找到有序的那部分子数组
2. 判断值是否在这个数组当中
3. 不在则移动l和r指针

## 解：

```c#
namespace CodeExercise
{
    public class ArraySearch
    {
        public static int Solve(int[] nums, int target)
        {
            int len = nums.Length;
            if (len == 0)
            {
                return -1;
            }
            if (len == 1)
            {
                return nums[0] == target ? 0 : -1;
            }

            int left = 0, right = len - 1;
            while (left <= right)
            {
                int mid = (left + right) / 2;
                if (nums[mid] == target)
                {
                    return mid;
                }
                if (nums[0] <= nums[mid])
                {
                    if (nums[0] <= target && target < nums[mid])
                    {
                        right = mid - 1;
                    }
                    else
                    {
                        left = mid + 1;
                    }
                }
                else
                {
                    if (nums[mid] < target && target <= nums[len - 1])
                    {
                        left = mid + 1;
                    }
                    else
                    {
                        right = mid - 1;
                    }
                }

            }
            return -1;
        }
    }
}

```



