# 括号生成

> 数字 `n` 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合。
>
> - `1 <= n <= 8`
>

## 结题思路

1. 递归

## 解：

```c#
using System.Collections.Generic;
using System.Text;

namespace CodeExercise
{
    public class GenerateParenthesis
    {
        public static IList<string> Solve(int n)
        {
            IList<string> result = new List<string>();
            StringBuilder stringBuilder = new StringBuilder();
            stringBuilder.Append('(');
            int leftRemind = n, rightRemind = n;
            Item(stringBuilder, leftRemind - 1, rightRemind, result);
            return result;
        }

        public static void Item(StringBuilder s, int left, int right, IList<string> result)
        {
            if (left == 0 && right == 0)
            {
                result.Add(s.ToString());
                return;
            }
            if (left == right)
            {
                s.Append('(');
                Item(s, left - 1, right, result);
            }
            else if (left == 0)
            {
                s.Append(')');
                Item(s, left, right - 1, result);
            }
            else
            {
                StringBuilder stringBuilder = new StringBuilder();
                stringBuilder.Append(s);
                s.Append('(');
                Item(s, left - 1, right, result);
                stringBuilder.Append(')');
                Item(stringBuilder, left, right - 1, result);
            }

        }
    }
}

```
