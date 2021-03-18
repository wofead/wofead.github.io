# 排列顺序

> 给出集合 `[1,2,3,...,n]`，其所有元素共有 `n!` 种排列。
>
> 按大小顺序列出所有排列情况，并一一标记，当 `n = 3` 时, 所有排列如下：
>
> 1. `"123"`
> 2. `"132"`
> 3. `"213"`
> 4. `"231"`
> 5. `"312"`
> 6. `"321"`
>
> 给定 `n` 和 `k`，返回第 `k` 个排列。

## 结题思路

1. 回溯得到所有的排列
2. 数学公式
3. 每个数字存在(n-1)阶乘个排列
4. 然后一层一层的剥离
5. 注意k--;


## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class GetPermutation
    {
        int count;
        public string Solve(int n, int k)
        {
            count = 0;
            char[] str = new char[n];
            bool[] vis = new bool[n];
            for (int i = 0; i < n; i++)
            {
                str[i] = (char)((i + 1) + '0');
            }
            StringBuilder stringBuilder = new StringBuilder();
            return Permute(n, stringBuilder, k, str, vis);
        }

        public string Permute(int n, StringBuilder stringBuilder, int k, char[] str, bool[] vis)
        {
            if (stringBuilder.Length == n)
            {
                count++;
                if (count == k)
                {
                    return stringBuilder.ToString();
                }
                return "";
            }
            int index = stringBuilder.Length;
            for (int i = 0; i < n; i++)
            {
                if (vis[i])
                {
                    continue;
                }
                vis[i] = true;
                stringBuilder.Append(str[i]);
                string itemStr = Permute(n, stringBuilder, k, str, vis);
                if (itemStr.Length > 0)
                {
                    return itemStr;
                }
                vis[i] = false;
                stringBuilder.Remove(index, 1);
            }
            return "";
        }

        public string OffcialSove(int n, int k)
        {
            List<int> factorial = new List<int>(n);
            List<char> str = new List<char>(n);

            StringBuilder res = new StringBuilder();
            str.Add('1');
            factorial.Add(1);
            k--;
            for (int i = 1; i < n; i++)
            {
                factorial.Add(factorial[i - 1] * i);
                str.Add((char)(str[i - 1] + 1));
            }
            for (int i = n - 1; i >= 0; i--)
            {
                int index = k / factorial[i];
                k -= index * factorial[i];
                res.Append(str[index]);
                str.RemoveAt(index);
            }
            return res.ToString();
        }
    }
}

```



