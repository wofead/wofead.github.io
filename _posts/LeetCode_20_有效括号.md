# 有效括号

> 给定一个只包括 `'('`，`')'`，`'{'`，`'}'`，`'['`，`']'` 的字符串 `s` ，判断字符串是否有效。
>
> 有效字符串需满足：
>
> - `左括号必须用相同类型的右括号闭合。`
> - 左括号必须以正确的顺序闭合。
> - `1 <= s.length <= 104`
> - `s 仅由括号 '()[]{}' 组成`
>

## 结题思路

1. 栈进行匹配计算
2. 匹配就pop
3. 否者就push
4. 如果最后栈为空，则满足匹配

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CodeExercise
{
    public class IsValidBrackets
    {
        public static bool Solve(string s)
        {
            Dictionary<char, char> dic = new Dictionary<char, char>();
            dic['('] = ')';
            dic['['] = ']';
            dic['{'] = '}';
            int len = s.Length;
            if (len % 2 == 1)
            {
                return false;
            }
            Stack<char> stack = new Stack<char>();
            stack.Push(s[0]);
            for (int i = 1; i < len; i++)
            {
                if (stack.Count > 0 && !dic.ContainsKey(stack.Peek()))
                {
                    return false;
                }
                if (stack.Count > 0 && dic[stack.Peek()].Equals(s[i]))
                {
                    stack.Pop();
                }
                else
                {
                    stack.Push(s[i]);
                }
            }
            if (stack.Count > 0)
            {
                return false;
            }
            else
            {
                return true;
            }
        }
    }
}

```
