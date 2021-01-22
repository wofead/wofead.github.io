# 电话号码字母组合

> 给定一个仅包含数字 `2-9` 的字符串，返回所有它能表示的字母组合。
>
> 给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

## 结题思路

1. 算出一共多少种组合
2. 取整，然后求余，算出在每一个应该取那个字符。

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CodeExercise
{
    public class LetterCombinations
    {
        // public IList<string> LetterCombinations(string digits) {
        public static IList<string> Solve(string digits)
        {
            IList<string> result = new List<string>();
            if (digits.Length == 0)
            {
                return result;
            }
            Dictionary<char, List<char>> dic = new Dictionary<char, List<char>>();
            dic['2'] = new List<char> { 'a', 'b', 'c' };
            dic['3'] = new List<char> { 'd', 'e', 'f' };
            dic['4'] = new List<char> { 'g', 'h', 'i' };
            dic['5'] = new List<char> { 'j', 'k', 'l' };
            dic['6'] = new List<char> { 'm', 'n', 'o' };
            dic['7'] = new List<char> { 'p', 'q', 'r', 's' };
            dic['8'] = new List<char> { 't', 'u', 'v' };
            dic['9'] = new List<char> { 'w', 'x', 'y', 'z' };
            int len = digits.Length;
            int strLen = 1;
            Dictionary<int, int> dicCharLen = new Dictionary<int, int>();
            dicCharLen[len] = 1;
            for (int i = len -1; i >= 0; i--)
            {
                strLen *= dic[digits[i]].Count;
                dicCharLen[i] = strLen;
            }
            StringBuilder str = new StringBuilder();
            for (int i = 0; i < strLen; i++)
            {
                for (int j = 0; j < len; j++)
                {
                    int index = (i / dicCharLen[j + 1]) % dic[digits[j]].Count;
                    str.Append(dic[digits[j]][index]);
                }
                result.Add(str.ToString());
                str.Clear();
            }
            return result;
        }
    }
}

```
