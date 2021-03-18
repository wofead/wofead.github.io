# 四数之和等于目标值

> 给定一个包含 n 个整数的数组 nums 和一个目标值 target，判断 nums 中是否存在四个元素 a，b，c 和 d ，使得 a + b + c + d 的值与 target 相等？找出所有满足条件且不重复的四元组。
>
> **注意：**
>
> 答案中不可以包含重复的四元组。
>
> - `链表中结点的数目为 sz`
> - `1 <= sz <= 30`
> - `0 <= Node.val <= 100`
> - `1 <= n <= sz`
>

## 结题思路

1. 和之前的三数之和很类似
2. 判断前两个数是否相同
3. 最后两个数使用双指针的方式进行移动
4. 要注意判断，相同组合

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CodeExercise
{
    public class FourSumEqualTarget
    {
        public static IList<IList<int>> Solve(int[] nums, int target)
        {
            IList<IList<int>> result = new List<IList<int>>();
            Array.Sort(nums);
            int len = nums.Length;

            for (int i = 0; i < len; i++)
            {
                if (i > 0 && nums[i - 1] == nums[i])
                {
                    continue;
                }
                for (int j = i + 1; j < len; j++)
                {
                    if (j > i + 1 && nums[j - 1] == nums[j])
                    {
                        continue;
                    }
                    int f = j + 1, b = len - 1;
                    int lastF = -1;
                    while (f < b)
                    {
                        int value = nums[i] + nums[j] + nums[f] + nums[b];
                        if (value == target)
                        {
                            if (lastF < 0 || nums[lastF] != nums[f])
                            {
                                var item = new List<int>();
                                item.Add(nums[i]);
                                item.Add(nums[j]);
                                item.Add(nums[f]);
                                item.Add(nums[b]);
                                result.Add(item);
                                lastF = f;
                            }
                            f++;
                        }
                        else if (nums[i] + nums[j] + nums[f] + nums[b] > target)
                        {
                            b--;
                        }
                        else if (nums[i] + nums[j] + nums[f] + nums[b] < target)
                        {
                            f++;
                        }
                    }
                }
            }
            return result;
        }
    }
}

```
