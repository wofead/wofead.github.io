# 最小覆盖子串

> 给你一个字符串 s 、一个字符串 t 。返回 s 中涵盖 t 所有字符的最小子串。如果 s 中不存在涵盖 t 所有字符的子串，则返回空字符串 "" 。
> 
>注意：如果 s 中存在这样的子串，我们保证它是唯一的答案。
> 

## 结题思路

这里存在两种解题思路，都是使用滑动窗口。

他们的区别是在到达临界值之后的处理，一种是临界值之后，左边的指针指向去除一个元素之后的元素位置。不断的更新startIndex和length。

一种是达到临界值之后，下次再遇到第一个元素，更新位置信息，即去除之前的冗余元素。

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class StringMinWindow
    {
        public string Solve(string s, string t)
        {
            int length = s.Length;
            int target = t.Length;
            int startIndex = -1;
            int len = s.Length + 1;
            Dictionary<char, int> tDic = new Dictionary<char, int>();
            foreach (var c in t)
            {
                if (tDic.ContainsKey(c))
                {
                    tDic[c] += 1;
                }
                else
                {
                    tDic[c] = 1;
                }
            }
            int count = 0;
            List<int> res = new List<int>();
            for (int i = 0; i < length; i++)
            {
                char c = s[i];
                if (tDic.ContainsKey(c))
                {
                    tDic[c] -= 1;
                    res.Add(i);
                    if (tDic[c] >= 0)
                    {
                        count++;
                    }
                    else if (c == s[res[0]])
                    {
                        while (tDic[s[res[0]]] < 0)
                        {
                            tDic[s[res[0]]] += 1;
                            res.RemoveAt(0);
                        }
                    }
                    if (count == target && res[0] != startIndex)
                    {
                        if (i - res[0] + 1 < len)
                        {
                            startIndex = res[0];
                            len = i - res[0] + 1;
                        }
                    }
                }
            }
            if (count == target)
            {
                return s.Substring(startIndex, len);
            }
            return "";
        }
    }
}

```



```java
class Solution {
    Map<Character, Integer> ori = new HashMap<Character, Integer>();
    Map<Character, Integer> cnt = new HashMap<Character, Integer>();

    public String minWindow(String s, String t) {
        int tLen = t.length();
        for (int i = 0; i < tLen; i++) {
            char c = t.charAt(i);
            ori.put(c, ori.getOrDefault(c, 0) + 1);
        }
        int l = 0, r = -1;
        int len = Integer.MAX_VALUE, ansL = -1, ansR = -1;
        int sLen = s.length();
        while (r < sLen) {
            ++r;
            if (r < sLen && ori.containsKey(s.charAt(r))) {
                cnt.put(s.charAt(r), cnt.getOrDefault(s.charAt(r), 0) + 1);
            }
            while (check() && l <= r) {
                if (r - l + 1 < len) {
                    len = r - l + 1;
                    ansL = l;
                    ansR = l + len;
                }
                if (ori.containsKey(s.charAt(l))) {
                    cnt.put(s.charAt(l), cnt.getOrDefault(s.charAt(l), 0) - 1);
                }
                ++l;
            }
        }
        return ansL == -1 ? "" : s.substring(ansL, ansR);
    }

    public boolean check() {
        Iterator iter = ori.entrySet().iterator(); 
        while (iter.hasNext()) { 
            Map.Entry entry = (Map.Entry) iter.next(); 
            Character key = (Character) entry.getKey(); 
            Integer val = (Integer) entry.getValue(); 
            if (cnt.getOrDefault(key, 0) < val) {
                return false;
            }
        } 
        return true;
    }
}
```

