# 全排列II

> 给定一个可包含重复数字的序列 `nums` ，**按任意顺序** 返回所有不重复的全排列。
>

## 结题思路

1. 动态规划
2. 首先对`nums`进行排序,这样在处理重复组合的时候比较便捷
3. 让每个剩余的数字都能够选到，不管有没有访问过
4. 如果访问过，就继续，否者添加，直到`ind`等于数组长度
5. 这里排除重复的方案为，连续重复的数字只允许前面的添加完，才能添加后面的，确保只能添加一次。
6. `vis[i] || (i > 0 && nums[i] == nums[i - 1] && vis[i - 1])`

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class ArrayPermuteUnique
    {
        public IList<IList<int>> Solve(int[] nums)
        {
            IList<IList<int>> res = new List<IList<int>>();
            List<int> perm = new List<int>();
            Array.Sort(nums);
            int n = nums.Length;
            bool[] vis = new bool[n];
            Backtrack(nums, res, perm, 0, vis);
            return res;
        }

        public void Backtrack(int[] nums, IList<IList<int>> res, List<int> perm, int idx, bool[] vis)
        {
            if (idx == nums.Length)
            {
                res.Add(new List<int>(perm));
                return;
            }
            for (int i = 0; i < nums.Length; i++)
            {
                if (vis[i] || (i > 0 && nums[i] == nums[i - 1] && vis[i - 1]))
                {
                    continue;
                }
                perm.Add(nums[i]);
                vis[i] = true;
                Backtrack(nums, res, perm, idx + 1, vis);
                perm.RemoveAt(idx);
                vis[i] = false;
            }
        }
    }
}

```


