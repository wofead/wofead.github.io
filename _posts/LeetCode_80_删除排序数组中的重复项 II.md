# 删除排序数组中的重复项 II

> 
>给定一个增序排列数组 `nums` ，你需要在 **[原地 ](http://baike.baidu.com/item/原地算法)**删除重复出现的元素，使得每个元素最多出现两次，返回移除后数组的新长度。
> 
>不要使用额外的数组空间，你必须在 **[原地](https://baike.baidu.com/item/原地算法) 修改输入数组** 并在使用 O(1) 额外空间的条件下完成。

## 结题思路

放到前面，然后删除。

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class ArrayRemoveDuplicates
    {
        public int Solve(int[] nums)
        {
            int length = nums.Length;
            int index = 1;
            int repeat = 0;
            for (int i = 1; i < length; i++)
            {
                if (nums[i] == nums[i - 1])
                {
                    if (repeat != 1)
                    {
                        repeat++;
                        nums[index] = nums[i];
                        index++;
                    }
                }
                else
                {
                    repeat = 0;
                    nums[index] = nums[i];
                    index++;
                }
            }
            Array.Resize(ref nums, index);
            return index;
        }
    }
}

```

## 

