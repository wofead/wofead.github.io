# 最长回文子串

> 给你一个字符串 `s`，找到 `s` 中最长的回文子串。
>
> **提示：**
>
> * `1 <= s.length <= 1000`
>*  `s` 仅由数字和英文字母（大写和/或小写）组成

## 解题思路
- 中间扩展法，以i位置的字符为中心进行判断


## 解：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CodeExercise
{
    public class LongestPalindrome
    {
        public static string SolveError(string s)
        {
            int front = 0, length = 1;
            bool isPalinDromeState = false;
            int palinDromeFlag = 0;
            int currentIndex = 0;
            //有可能是多个字母，而不仅仅是一个字母重复，所以这里存在问题
            int sameIndex = -1;
            bool sameFlag = false;
            for (int i = 1; i < s.Length; i++)
            {
                if (s[i] == s[i - 1])
                {
                    if (!sameFlag)
                    {
                        sameIndex = i - 1;
                        sameFlag = true;
                    }
                }
                else
                {
                    sameFlag = false;
                    sameIndex = -1;
                }
                //非回文状态
                if (!isPalinDromeState)
                {
                    palinDromeFlag = PalinDromeState(s, i);
                    if (palinDromeFlag != 0)
                    {
                        isPalinDromeState = true;
                        if (palinDromeFlag == 1)
                        {
                            currentIndex = i;
                        }
                        else if (palinDromeFlag == 2)
                        {
                            currentIndex = i - 1;
                        }
                        else
                        {
                            currentIndex = i - 2;
                        }
                    }
                }
                else
                {
                    if (currentIndex - 1 >= 0 && s[i] == s[currentIndex - 1])
                    {
                        currentIndex--;
                    }
                    else
                    {
                        isPalinDromeState = false;
                        if (i - currentIndex > length)
                        {
                            front = currentIndex;
                            length = i - currentIndex;
                        }
                        palinDromeFlag = PalinDromeState(s, i);
                        if (palinDromeFlag != 0)
                        {
                            isPalinDromeState = true;
                            if (palinDromeFlag == 1)
                            {
                                currentIndex = sameIndex;
                            }
                            else if (palinDromeFlag == 2)
                            {
                                currentIndex = i - 1;
                            }
                            else if (palinDromeFlag == 3)
                            {
                                currentIndex = i - 2;
                            }
                        }
                    }
                }

            }
            if (isPalinDromeState && s.Length - currentIndex > length)
            {
                front = currentIndex;
                length = s.Length - currentIndex;
            }
            return s.Substring(front, length);
        }

        public static int PalinDromeState(string s, int i)
        {
            int palinDromeFlag = 0;
            if (s[i] == s[i - 1] && i > 1 && s[i] == s[i - 2])
            {
                palinDromeFlag = 1;
            }
            //连着两个相同的,并且不是中间隔一个的回文"aaa"
            else if (s[i] == s[i - 1])
            {
                palinDromeFlag = 2;
            }
            //中间隔了一个相同 "aaaa"
            else if (i > 1 && s[i] == s[i - 2])
            {
                palinDromeFlag = 3;
            }
            return palinDromeFlag;
        }

        public static string Solve(string s)
        {
            int start = 0, end = 0;
            for (int i = 0; i < s.Length; i++)
            {
                int len1 = ExpandAroundCenter(s, i, i);
                int len2 = ExpandAroundCenter(s, i, i + 1);
                int len = Math.Max(len1, len2);
                if (len > end - start)
                {
                    start = i - (len - 1) / 2;
                    end = i + len / 2;
                }
            }
            return s.Substring(start, end + 1 - start);
        }

        public static int ExpandAroundCenter(string s, int left, int right)
        {
            while (left >= 0 && right < s.Length && s[left] == s[right])
            {
                left--;
                right++;
            }
            return right - left - 1;
        }
    }
}

```

## 官方解题思路

#### Manacher 算法。

为了表述方便，我们定义一个新概念臂长，表示中心扩展算法向外扩展的长度。如果一个位置的最大回文字符串长度为 2 * length + 1 ，其臂长为 length。

下面的讨论只涉及长度为奇数的回文字符串。长度为偶数的回文字符串我们将会在最后与长度为奇数的情况统一起来。



在中心扩展算法的过程中，我们能够得出每个位置的臂长。那么当我们要得出以下一个位置 i 的臂长时，能不能利用之前得到的信息呢？

答案是肯定的。具体来说，如果位置 j 的臂长为 length，并且有 j + length > i，如下图所示：

![fig1](https://assets.leetcode-cn.com/solution-static/5/5_fig1.png)

当在位置 i 开始进行中心拓展时，我们可以先找到 i 关于 j 的对称点 2 * j - i。那么如果点 2 * j - i 的臂长等于 n，我们就可以知道，点 i 的臂长至少为 min(j + length - i, n)。那么我们就可以直接跳过 i 到 i + min(j + length - i, n) 这部分，从 i + min(j + length - i, n) + 1 开始拓展。

我们只需要在中心扩展法的过程中记录右臂在最右边的回文字符串，将其中心作为 j，在计算过程中就能最大限度地避免重复计算。

那么现在还有一个问题：如何处理长度为偶数的回文字符串呢？

我们可以通过一个特别的操作将奇偶数的情况统一起来：我们向字符串的头尾以及每两个字符中间添加一个特殊字符 #，比如字符串 aaba 处理后会变成 #a#a#b#a#。那么原先长度为偶数的回文字符串 aa 会变成长度为奇数的回文字符串 #a#a#，而长度为奇数的回文字符串 aba 会变成长度仍然为奇数的回文字符串 #a#b#a#，我们就不需要再考虑长度为偶数的回文字符串了。

注意这里的特殊字符不需要是没有出现过的字母，我们可以使用任何一个字符来作为这个特殊字符。这是因为，当我们只考虑长度为奇数的回文字符串时，每次我们比较的两个字符奇偶性一定是相同的，所以原来字符串中的字符不会与插入的特殊字符互相比较，不会因此产生问题。

```java
class Solution {
    public String longestPalindrome(String s) {
        int start = 0, end = -1;
        StringBuffer t = new StringBuffer("#");
        for (int i = 0; i < s.length(); ++i) {
            t.append(s.charAt(i));
            t.append('#');
        }
        t.append('#');
        s = t.toString();

        List<Integer> arm_len = new ArrayList<Integer>();
        int right = -1, j = -1;
        for (int i = 0; i < s.length(); ++i) {
            int cur_arm_len;
            if (right >= i) {
                int i_sym = j * 2 - i;
                int min_arm_len = Math.min(arm_len.get(i_sym), right - i);
                cur_arm_len = expand(s, i - min_arm_len, i + min_arm_len);
            } else {
                cur_arm_len = expand(s, i, i);
            }
            arm_len.add(cur_arm_len);
            if (i + cur_arm_len > right) {
                j = i;
                right = i + cur_arm_len;
            }
            if (cur_arm_len * 2 + 1 > end - start) {
                start = i - cur_arm_len;
                end = i + cur_arm_len;
            }
        }

        StringBuffer ans = new StringBuffer();
        for (int i = start; i <= end; ++i) {
            if (s.charAt(i) != '#') {
                ans.append(s.charAt(i));
            }
        }
        return ans.toString();
    }

    public int expand(String s, int left, int right) {
        while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
            --left;
            ++right;
        }
        return (right - left - 2) / 2;
    }
}

```

