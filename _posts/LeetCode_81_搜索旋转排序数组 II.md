# 搜索旋转数组 II

> 假设按照升序排序的数组在预先未知的某个点上进行了旋转。
>
> ( 例如，数组 [0,0,1,2,2,5,6] 可能变为 [2,5,6,0,0,1,2] )。
>
> 编写一个函数来判断给定的目标值是否存在于数组中。若存在返回 true，否则返回 false。
>

## 结题思路

之前的由于没有重复，而现在存在重复，所以当mid,left和right的值相等时，先去重，即left++，然后再比较。

```c#
if (nums[left] == nums[mid]) {
    left++;
    continue;
}
```



## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class ArrayFindTarget
    {
        public bool Solve(int[] nums, int target)
        {
            int len = nums.Length;
            if (len == 1)
            {
                return nums[0] == target;
            }
            int left = 0, right = len - 1;
            while (left <= right)
            {
                if (left == right)
                {
                    return nums[left] == target;
                }
                int mid = (left + right) / 2;
                if (nums[mid] == target || nums[left] == target || nums[right] == target)
                {
                    return true;
                }
                if (nums[mid] == nums[left] && nums[mid] == nums[right])
                {
                    left++;
                    while (left + 1 <= right && nums[left] == nums[left + 1])
                    {
                        left++;
                    }
                }
                else if (nums[mid] <= nums[right])
                {
                    if (nums[mid] > target)
                    {
                        right = mid - 1;
                    }
                    else
                    {
                        if (nums[right] > target)
                        {
                            left = mid + 1;
                        }
                        else
                        {
                            right = mid - 1;
                        }
                    }
                }
                else
                {
                    if (nums[mid] < target)
                    {
                        left = mid + 1;
                    }
                    else
                    {
                        if (nums[left] > target)
                        {
                            left = mid + 1;
                        }
                        else
                        {
                            right = mid - 1;
                        }
                    }
                }
            }
            return false;
        }
    }
}

```

## 

