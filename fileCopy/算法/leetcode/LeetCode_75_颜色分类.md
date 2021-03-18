# 颜色分类

> 
> 给定一个包含红色、白色和蓝色，一共 `n` 个元素的数组，**[原地](https://baike.baidu.com/item/原地算法)**对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。
>
> 此题中，我们使用整数 `0`、 `1` 和 `2` 分别表示红色、白色和蓝色。

## 结题思路

1. 使用两个指针，一个指向0要交换的位置，一个指向2要交换的位置。
2. 对于遇到0的使用，交换，然后curIndex和left都加1。
3. 遇到1，则curIndex加1。
4. 2的话，继续判断，不增加。



## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class ArraySortColors
    {
        public void Solve(int[] nums)
        {
            int length = nums.Length;
            int left = 0, right = length - 1;
            int curIndex = 0;
            while (curIndex <= right)
            {
                int value = nums[curIndex];
                if (value == 1)
                {
                    curIndex++;
                }
                else if (value == 0)
                {
                    nums[curIndex] = nums[left];
                    nums[left] = 0;
                    left++;
                    curIndex++;
                }
                else
                {
                    nums[curIndex] = nums[right];
                    nums[right] = 2;
                    right--;
                }
            }
        }
    }
}

```





