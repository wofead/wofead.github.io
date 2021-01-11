# 回文数

> 判断一个整数是否是回文数。回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。
>
> **提示：**
>
> * `0 <= s.length <= 200`
> * `s` 由英文字母（大写和小写）、数字、`' '`、`'+'`、`'-'` 和 `'.'` 组成

## 解题思路
- 将每个数字拆出来。

- 然后在乘回去，判断是否相等。



## 解：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CodeExercise
{
    public class IsPalindrome
    {
        public static bool Solve(int x)
        {
            if (x < 0) return false;
            if (x < 10) return true;
            int c = x;
            List<int> list = new List<int>();
            while (c != 0)
            {
                list.Add(c % 10);
                c = c / 10;
            }
            for (int i = 0; i < list.Count; i++)
            {
                c = c * 10 + list[i];
            }
            if (c == x)
            {
                return true;
            }
            return false;
        }
    }
}
using System.Collections.Generic;

namespace CodeExercise
{
    public class MyAtoi
    {
        public static int Solve(string s)
        {
            int len = s.Length;
            List<char> canMeet = new List<char>();
            bool isNeg = false;
            canMeet.Add(' ');
            canMeet.Add('+');
            canMeet.Add('-');
            int ret = 0;
            for (int i = 0; i < len; i++)
            {
                if (s[i] <= '9' && s[i] >= '0')
                {
                    if (canMeet.Count > 1)
                    {
                        canMeet.Remove(' ');
                        canMeet.Remove('+');
                        canMeet.Remove('-');
                    }
                    int pop = s[i] - '0';
                    if (isNeg)
                    {
                        pop = -pop;
                    }
                    if (ret > int.MaxValue / 10 || (ret == int.MaxValue / 10 && pop > 7)) return int.MaxValue;
                    if (ret < int.MinValue / 10 || (ret == int.MinValue / 10 && pop < -8)) return int.MinValue;
                    ret = ret * 10 + pop;
                    continue;
                }
                if (!canMeet.Contains(s[i]) || s[i] == '.')
                {
                    return ret;
                }
                if (s[i] == '+')
                {
                    canMeet.Remove(' ');
                    canMeet.Remove('+');
                    canMeet.Remove('-');
                }
                if (s[i] == '-')
                {
                    canMeet.Remove(' ');
                    canMeet.Remove('+');
                    canMeet.Remove('-');
                    isNeg = true;
                }
            }
            return ret;
        }
    }
}


```

