# 字母异位词分组

> 给定一个字符串数组，将字母异位词组合在一起。字母异位词指字母相同，但排列不同的字符串。
>
> - 所有输入均为小写字母。
>- 不考虑答案输出的顺序。

## 结题思路

1. 使用一个26位的`stringBuilder`
2. 存放字符的个数
3. 利用字典进行存储
4. 判断和添加

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class StringArrayGroupAnagrams
    {
        public IList<IList<string>> Solve(string[] strs)
        {
            IList<IList<string>> res = new List<IList<string>>();
            Dictionary<string, List<string>> dic = new Dictionary<string, List<string>>();
            foreach (string item in strs)
            {
                StringBuilder str = new StringBuilder();
                str.Append('0', 26);
                for (int i = 0; i < item.Length; i++)
                {
                    int index = item[i] - 'a';
                    str[index]++;
                }
                if (!dic.ContainsKey(str.ToString()))
                {
                    List<string> resItem = new List<string>();
                    dic[str.ToString()] = resItem;
                    resItem.Add(item);
                }
                else
                {
                    dic[str.ToString()].Add(item);
                }
            }
            foreach (var pairs in dic)
            {
                res.Add(pairs.Value);
            }
            return res;
        }
    }
}

```



