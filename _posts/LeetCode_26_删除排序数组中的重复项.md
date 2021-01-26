# 删除排序数组中的重复项

> 给定一个排序数组，你需要在 原地 删除重复出现的元素，使得每个元素只出现一次，返回移除后数组的新长度。
>
> 不要使用额外的数组空间，你必须在 原地 修改输入数组 并在使用 O(1) 额外空间的条件下完成。
>

## 结题思路

1. 通过使用一个指针，来指向不重复的位置
2. 不断地往不重复的位置赋值就可以了
3. 重新设置数组大小

## 解：

```c#
namespace CodeExercise
{
    public class ArrayRemoveDuplicates
    {
        public static int Solve(int[] nums)
        {
            int len = nums.Length;
            if (len == 0)
            {
                return 0;
            }
            int result = 1;
            for (int i = 1; i < len; i++)
            {
                if (nums[i] != nums[i - 1])
                {
                    nums[result] = nums[i];
                    result ++;
                }
            }
            Array.Resize<int>(ref nums, result);
            return result;
        }
    }
}

```
