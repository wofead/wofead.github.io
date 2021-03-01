# 子集

> 给你一个整数数组 `nums` ，数组中的元素 **互不相同** 。返回该数组所有可能的子集（幂集）。
>
> 解集 **不能** 包含重复的子集。你可以按 **任意顺序** 返回解集。

## 结题思路1

记原序列中元素的总数为 n。原序列中的每个数字 ai 的状态可能有两种，即「在子集中」和「不在子集中」。我们用 1 表示「在子集中」，0 表示不在子集中，那么每一个子集可以对应一个长度为 n 的 0/1 序列，第 i 位表示ai 是否在子集中。

| 0/1 **序列** | **子集** | 0/1 **序列对应的二进制数** |
| ------------ | -------- | -------------------------- |
| 000          | {}       | 0                          |
| 001          | {9}      | 1                          |
| 010          | {2}      | 2                          |
| 011          | {2,9}    | 3                          |
| 100          | {5}      | 4                          |
| 101          | {5,9}    | 5                          |
| 110          | {5,2}    | 6                          |
| 111          | {5,2,9}  | 7                          |

## 结题思路2

按照上一道题的结题思路进行扩展

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class ArraySubsets
    {
        public IList<IList<int>> Solve(int[] nums)
        {
            int len = nums.Length;
            IList<IList<int>> res = new List<IList<int>>();
            List<int> combine = new List<int>();
            res.Add(new List<int>(combine));
            for (int i = 1; i <= len; i++)
            {
                ComibeArr(combine, res, nums, len, i, 0, i);
            }
            return res;
        }

        public void ComibeArr(List<int> combine, IList<IList<int>> res, int[] nums, int n, int k, int index, int remind)
        {
            if (remind == 0)
            {
                res.Add(new List<int>(combine));
                return;
            }
            for (int i = 0; i < n - index; i++)
            {
                combine.Add(nums[index + i]);
                ComibeArr(combine, res, nums, n, k, index + i + 1, remind - 1);
                combine.RemoveAt(combine.Count - 1);
            }
        }

        public IList<IList<int>> OfficialSolve(int[] nums)
        {
            List<int> t = new List<int>();
            IList<IList<int>> ans = new List<IList<int>>();

            int n = nums.Length;
            for (int mask = 0; mask < (1 << n); ++mask)
            {
                t.Clear();
                for (int i = 0; i < n; ++i)
                {
                    if ((mask & (1 << i)) != 0)
                    {
                        t.Add(nums[i]);
                    }
                }
                ans.Add(new List<int>(t));
            }
            return ans;
        }
    }
}

```

## 结题思路3

```java

class Solution {
    List<Integer> t = new ArrayList<Integer>();
    List<List<Integer>> ans = new ArrayList<List<Integer>>();

    public List<List<Integer>> subsets(int[] nums) {
        dfs(0, nums);
        return ans;
    }

    public void dfs(int cur, int[] nums) {
        if (cur == nums.length) {
            ans.add(new ArrayList<Integer>(t));
            return;
        }
        t.add(nums[cur]);
        dfs(cur + 1, nums);
        t.remove(t.size() - 1);
        dfs(cur + 1, nums);
    }
}
```

