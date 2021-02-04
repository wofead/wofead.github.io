# 解答的数独

> 给定一个无重复元素的数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。
>
> candidates 中的数字可以无限制重复被选取。
>
> 1. 所有数字（包括 `target`）都是正整数。
> 2. 解集不能包含重复的组合。 
>

## 结题思路

1. 穷举各种情况
2. 想着每个组合都是从最小值开始累加
3. 大于了，移除顶部两个继续计算
4. 如果超限了，一样的移除，重置index

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class ArrayCombinationSum
    {
        public IList<IList<int>> Solve(int[] candidates, int target)
        {
            IList<IList<int>> result = new List<IList<int>>();
            Array.Sort(candidates);
            int len = candidates.Length;
            for (int i = 0; i < len; i++)
            {
                int value = candidates[i];
                int maxCount = target / value;
                if (maxCount == 0 || (maxCount == 1 && value > target))
                {
                    return result;
                }
                if (maxCount == 1 && value == target)
                {
                    List<int> it = new List<int>();
                    it.Add(value);
                    result.Add(it);
                    return result;
                }
                int sum = maxCount * value;
                List<int> item = new List<int>();
                Dictionary<int, int> valueIndex = new Dictionary<int, int>();
                valueIndex[value] = i;
                for (int j = 0; j < maxCount; j++)
                {
                    item.Add(value);
                }
                int index = i;
                while (item.Count > 0)
                {
                    //atttention
                    if (index > len - 1)
                    {
                        int tv = candidates[index - 1];
                        while (item.Remove(tv))
                        {
                            if (item.Count == 0)
                            {
                                break;
                            }
                            sum -= tv;
                        }
                        if (item.Count == 0)
                        {
                            break;
                        }
                        int vt = item[item.Count - 1];
                        item.Remove(vt);
                        sum -= vt;
                        index = valueIndex[vt] + 1;
                        continue;
                    }
                    if (sum < target)
                    {
                        sum += candidates[index];
                        item.Add(candidates[index]);
                        valueIndex[candidates[index]] = index;
                    }
                    else if (sum >= target)
                    {
                        if (sum == target)
                        {
                            List<int> it = new List<int>();
                            foreach (var ii in item)
                            {
                                it.Add(ii);
                            }
                            result.Add(it);
                        }

                        int v1 = item[item.Count - 1];
                        int v2 = item[item.Count - 2];
                        item.RemoveAt(item.Count - 1);
                        item.RemoveAt(item.Count - 1);
                        sum -= (v1 + v2);
                        if (v1 == v2)
                        {
                            index++;
                        }
                        else
                        {
                            index = valueIndex[v2] + 1;
                        }
                    }
                }
            }
            return result;
        }
    }
}

```

## 回溯法

```java
public class Solution {
    public int longestValidParentheses(String s) {
        int maxans = 0;
        Deque<Integer> stack = new LinkedList<Integer>();
        stack.push(-1);
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == '(') {
                stack.push(i);
            } else {
                stack.pop();
                if (stack.empty()) {
                    stack.push(i);
                } else {
                    maxans = Math.max(maxans, i - stack.peek());
                }
            }
        }
        return maxans;
    }
}
```

