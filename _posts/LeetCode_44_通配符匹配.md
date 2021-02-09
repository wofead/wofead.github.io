# 通配符匹配

> 给定一个字符串 (`s`) 和一个字符模式 (`p`) ，实现一个支持 `'?'` 和 `'*'` 的通配符匹配。
>
> ```
> '?' 可以匹配任何单个字符。
> '*' 可以匹配任意字符串（包括空字符串）。
> ```
>
> 两个字符串**完全匹配**才算匹配成功。
>
> **说明**
>
> 1. `s` 可能为空，且只包含从 `a-z` 的小写字母。
> 2. `p 可能为空，且只包含从 a-z 的小写字母，以及字符 ? 和 *。`

## 结题思路

1. 类似于`leet_code_10`.


## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class StringIsMatch
    {
        public bool Solve(string s, string p)
        {
            if (s.Length == 0 && p.Length == 0)
            {
                return true;
            }
            if (s.Length == 0)
            {
                int index = 0;
                while (index < p.Length && p[index] == '*')
                {
                    index++;
                }
                if (index < p.Length)
                {
                    return false;
                }
                else
                {
                    return true;
                }
            }
            // ?匹配一个字符串，*可以匹配任意字符串，动态规划，关于*的动态规划，匹配几个字符的问题
            return Match(s, p, 0, 0);
        }

        public bool Match(string s, string p, int indexS, int indexP)
        {
            if (indexS >= s.Length && indexP >= p.Length)
            {
                return true;
            }
            if (indexS >= s.Length && p[indexP] == '*')
            {
                return Match(s, p, indexS, indexP + 1);
            }
            if (indexS >= s.Length || indexP >= p.Length)
            {
                return false;
            }
            if (p[indexP] == '?' || p[indexP] == s[indexS])
            {
                return Match(s, p, indexS + 1, indexP + 1);
            }
            else if (p[indexP] == '*')
            {
                if (!Match(s, p, indexS + 1, indexP))
                {
                    if (!Match(s, p, indexS, indexP + 1))
                    {
                        return Match(s, p, indexS + 1, indexP + 1);
                    }
                    else
                    {
                        return true;
                    }
                }
                else
                {
                    return true;
                }
            }
            else
            {
                return false;
            }
        }

        public bool OffcialSolve(string s, string p)
        {
            int m = s.Length;
            int n = p.Length;
            bool[,] dp = new bool[m + 1, n + 1];
            dp[0, 0] = true;
            for (int i = 1; i <= n; i++)
            {
                if (p[i - 1] == '*')
                {
                    dp[0, i] = true;
                }
                else
                {
                    break;
                }
            }
            for (int i = 1; i <= m; i++)
            {
                for (int j = 1; j <= n; j++)
                {
                    if (p[j - 1] == '*')
                    {
                        dp[i, j] = dp[i, j - 1] | dp[i - 1, j];
                    }
                    else if (p[j - 1] == '?' || s[i - 1] == p[j - 1])
                    {
                        dp[i, j] = dp[i - 1, j - 1];
                    }
                }
            }
            return dp[m, n];
        }

        //public bool GreedSolve(string s, string p)
        //{

        //}
    }
}

```





