# 删除数组中的指定元素

> 给你一个数组 nums 和一个值 val，你需要 原地 移除所有数值等于 val 的元素，并返回移除后数组的新长度。
>
> 不要使用额外的数组空间，你必须仅使用 O(1) 额外空间并 原地 修改输入数组。
>
> 元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。
>

## 结题思路

1. 通过使用一个指针，来指向上次赋值的位置
2. 当不是目标值的时候就赋值，更新指针位置
3. 重新设置数组大小

## 解：

```c#
using System;
namespace CodeExercise
{
    public class ArrayRemoveElement
    {
        public static int Solve(int[] nums, int val)
        {
            int len = nums.Length;
            if (len == 0)
            {
                return 0;
            }
            int result = 0;
            for (int i = 0; i < len; i++)
            {
                if (nums[i] != val)
                {
                    nums[result] = nums[i];
                    result++;
                }
            }
            Array.Resize<int>(ref nums, result);
            return result;
        }
    }
}

```
