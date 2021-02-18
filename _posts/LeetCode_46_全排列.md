# 全排列

> 给定一个 **没有重复** 数字的序列，返回其所有可能的全排列。
>

## 结题思路

1. 动态规划
2. 通过维护一个动态的list来移动指针，从而获取下次应该选取那个数字
3. 将已经摘取的数字放到左边，未摘取的放到右边，知道first等于数组的长度
4. 动态规划就是遍历每个数字，让每个数字都能成为当前剩余数字中的第一个。



## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class ArrayPermute
    {
        public IList<IList<int>> Solve(int[] nums)
        {
            IList<IList<int>> res = new List<IList<int>>();
            List<int> output = new List<int>();
            foreach (var num in nums)
            {
                output.Add(num);
            }
            int n = nums.Length;
            Backtrack(n, output, res, 0);
            return res;
        }

        public void Backtrack(int n, List<int> output, IList<IList<int>> res, int first)
        {
            if (first == n)
            {
                res.Add(new List<int>(output));
            }
            for (int i = first; i < n; i++)
            {
                int temp = output[first];
                output[first] = output[i];
                output[i] = temp;
                Backtrack(n, output, res, first + 1);
                temp = output[first];
                output[first] = output[i];
                output[i] = temp;
            }
        }
    }
}

```


