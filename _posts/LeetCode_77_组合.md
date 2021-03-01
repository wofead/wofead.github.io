# 组合

> 给定两个整数 *n* 和 *k*，返回 1 ... *n* 中所有可能的 *k* 个数的组合。
> 

## 结题思路

使用动态规划的方式。

1. 当remind等于0的时候，说明组合满了
2. 从1开始，剩下的每个数字都有机会当第一个数字
3. 因为是组合，后面选择的数字永远比前面大。

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class Combine
    {
        public IList<IList<int>> Solve(int n, int k)
        {
            IList<IList<int>> res = new List<IList<int>>();
            List<int> combine = new List<int>();
            ComibeArr(combine, res, n, k, 0, k);
            return res;
        }

        public void ComibeArr(List<int> combine, IList<IList<int>> res, int n, int k, int index, int remind)
        {
            if (remind == 0)
            {
                res.Add(new List<int>(combine));
                return;
            }
            for (int i = 1; i <= n - index; i++)
            {
                combine.Add(index + i);
                ComibeArr(combine, res, n, k, index + i, remind - 1);
                combine.Remove(index + i);
            }
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

