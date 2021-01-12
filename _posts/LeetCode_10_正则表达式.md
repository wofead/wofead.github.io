# 正则表达式

> 给你一个字符串 `s` 和一个字符规律 `p`，请你来实现一个支持 `'.'` 和 `'*'` 的正则表达式匹配。
>
> **提示：**
>
> * `'.'` 匹配任意单个字符
> * `'*'` 匹配零个或多个前面的那一个元素
> * 所谓匹配，是要涵盖 **整个** 字符串 `s`的，而不是部分字符串。
> * `0 <= s.length <= 20`
> * `0 <= p.length <= 30`
> * `s` 可能为空，且只包含从 `a-z` 的小写字母。
> * `p` 可能为空，且只包含从 `a-z` 的小写字母，以及字符 `.` 和 `*`。
> * 保证每次出现字符 `*` 时，前面都匹配到有效的字符

## 解题思路

我们每次从字符串`p` 中取出一个字符或者「字符 + 星号」的组合，并在 `s` 中进行匹配。对于`p` 中一个字符而言，它只能在 `s`中匹配一个字符，匹配的方法具有唯一性；而对于 `p` 中字符 + 星号的组合而言，它可以在`s`中匹配任意自然数个字符，并不具有唯一性。因此我们可以考虑使用动态规划，对匹配的方案进行枚举。

字母 + 星号的组合在匹配的过程中，本质上只会有两种情况：

- 匹配 `s` 末尾的一个字符，将该字符扔掉，而该组合还可以继续进行匹配；
- 不匹配字符，将该组合扔掉，不再进行匹配。

> ```yaml
> 以一个例子详解动态规划转移方程：
> S = abbbbc
> P = ab*d*c
> 1. 当 i, j 指向的字符均为字母（或 '.' 可以看成一个特殊的字母）时，
>    只需判断对应位置的字符即可，
>    若相等，只需判断 i,j 之前的字符串是否匹配即可，转化为子问题 f[i-1][j-1].
>    若不等，则当前的 i,j 肯定不能匹配，为 false.
>    
>        f[i-1][j-1]   i
>             |        |
>    S [a  b  b  b  b][c] 
>    
>    P [a  b  *  d  *][c]
>                      |
>                      j
>    
> 
> 2. 如果当前 j 指向的字符为 '*'，则不妨把类似 'a*', 'b*' 等的当成整体看待。
>    看下面的例子
> 
>             i
>             |
>    S  a  b [b] b  b  c  
>    
>    P  a [b  *] d  *  c
>             |
>             j
>    
>    注意到当 'b*' 匹配完 'b' 之后，它仍然可以继续发挥作用。
>    因此可以只把 i 前移一位，而不丢弃 'b*', 转化为子问题 f[i-1][j]:
>    
>          i
>          | <--
>    S  a [b] b  b  b  c  
>    
>    P  a [b  *] d  *  c
>             |
>             j
>    
>    另外，也可以选择让 'b*' 不再进行匹配，把 'b*' 丢弃。
>    转化为子问题 f[i][j-2]:
> 
>             i
>             |
>    S  a  b [b] b  b  c  
>     
>    P [a] b  *  d  *  c
>       |
>       j <--
> 
> 3. 冗余的状态转移不会影响答案，
>    因为当 j 指向 'b*' 中的 'b' 时, 这个状态对于答案是没有用的,
>    原因参见评论区 稳中求胜 的解释, 当 j 指向 '*' 时,
>    dp[i][j]只与dp[i][j-2]有关, 跳过了 dp[i][j-1].
> ```



## 解：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CodeExercise
{
    public class IsMatch
    {
        public static bool ErrorSolve(string s, string p)
        {
            //1表示*状态，2表示.状态
            char k = ' ';
            int index = 0;
            int repeatNum = 0;
            int flag = 0;
            if (index == p.Length) flag = -1;
            else flag = ChangeFlag(p, ref index, ref k);
            for (int i = 0; i < s.Length; i++)
            {
                if (flag == -1) return false;
                if (flag == 1 && (k == '.' || s[i] == k))
                {
                    repeatNum++;
                    continue;
                }
                if (flag == 1 && s[i] != k)
                {
                    index++;
                    if (index == p.Length) flag = -1;
                    else flag = ChangeFlag(p, ref index, ref k);
                    repeatNum = 0;
                }
                if (flag == 1 && (k == '.' || s[i] == k))
                {
                    repeatNum++;
                    continue;
                }
                if (flag == 2)
                {
                    index++;
                    if (index == p.Length) flag = -1;
                    else flag = ChangeFlag(p, ref index, ref k);
                    continue;
                }
                else
                {
                    if (flag != -1 && s[i] == p[index])
                    {
                        index++;
                        if (index == p.Length) flag = -1;
                        else flag = ChangeFlag(p, ref index, ref k);
                    }
                    else
                    {
                        return false;
                    }
                }
            }
            if (flag == -1)
            {
                return true;
            }
            if (flag == 1)
            {
                for (int i = p.Length - 1, j = s.Length - 1; i > index; i--, j--)
                {
                    if (p[i] == '*')
                    {
                        repeatNum += 2;
                        j += 1;
                        i -= 1;
                        continue;
                    }
                    if (j < 0 || (p[i] != s[j] && p[i] != '.') || p.Length - i > repeatNum)
                    {
                        return false;
                    }
                }
                return true;

            }
            return false;
        }

        public static int ChangeFlag(string p, ref int index, ref char k)
        {
            int flag = 0;
            if (index < p.Length - 1)
            {
                if (p[index + 1] == '*')
                {
                    flag = 1;
                    k = p[index];
                    index++;
                    return flag;
                }
            }
            if (p[index] == '.')
            {
                flag = 2;
            }

            return flag;
        }

        public static bool Solve(string s, string p)
        {
            int m = s.Length;
            int n = p.Length;
            bool[,] f = new bool[m+1,n+1];
            //f[i,j]表示s的前i个字符与p的前j个字符是否能够匹配
            f[0,0] = true;
            for (int i = 0; i <= m; ++i)
            {
                for (int j = 1; j <= n; ++j)
                {
                    if (p[j - 1] == '*')
                    {
                        f[i,j] = f[i,j - 2];
                        if (matches(s, p, i, j - 1))
                        {
                            f[i,j] = f[i,j] || f[i - 1,j];
                        }
                    }
                    else
                    {
                        if (matches(s, p, i, j))
                        {
                            f[i,j] = f[i - 1,j - 1];
                        }
                    }
                }
            }
            return f[m,n];
        }

        public static bool matches(string s, string p, int i, int j)
        {
            if (i == 0)
            {
                return false;
            }
            if (p[j - 1] == '.')
            {
                return true;
            }
            return s[i - 1] == p[j - 1];
        }
    }
}



```

