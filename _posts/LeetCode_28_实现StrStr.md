# 实现StrStr

> 实现 strStr() 函数。
>
> 给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从0开始)。如果不存在，则返回  -1。
>
> 说明:
>
> 当 needle 是空字符串时，我们应当返回什么值呢？这是一个在面试中很好的问题。
>
> 对于本题而言，当 needle 是空字符串时我们应当返回 0 。这与C语言的 strstr() 以及 Java的 indexOf() 定义相符。
>

## 结题思路

1. 将每个字符作为开头进行逐一判断匹配（暴力解法）
2. 暴力解法的优化，前缀表
3. KMP的主要思想是当出现字符串不匹配时，可以知道一部分之前已经匹配的文本内容，可以利用这些信息避免从头再去做匹配了。
4. next数组就是一个前缀表（prefix table）。
5. 前缀表是用来回退的，它记录了模式串与主串(文本串)不匹配的时候，模式串应该从哪里开始重新匹配。

## KMP

1. 构造前缀表，即prefixArr表

2. 初始化前缀表：定义两个指针i和j，j指向前缀终止位置（严格来说是终止位置减一的位置），i指向后缀终止位置（与j同理）。

3. 处理前后缀不相同的情况：因为j初始化为-1，那么i就从1开始，进行s[i] 与 s[j+1]的比较。如果 s[i] 与 s[j+1]不相同，也就是遇到 前后缀末尾不相同的情况，就要向前回溯。next[j]就是记录着j（包括j）之前的子串的相同前后缀的长度。

   那么 s[i] 与 s[j+1] 不相同，就要找 j+1前一个元素在next数组里的值（就是next[j]）。

4. 处理前后缀相同的情况：如果s[i] 与 s[j + 1] 相同，那么就同时向后移动i 和j 说明找到了相同的前后缀，同时还要将j（前缀的长度）赋给next[i], 因为next[i]要记录相同前后缀的长度。

5. 定义两个下标j 指向模式串起始位置，i指向文本串其实位置。

6. 那么j初始值依然为-1，为什么呢？ **依然因为next数组里记录的起始位置为-1。**

7. i就从0开始，遍历文本串

8. 接下来就是 s[i] 与 t[j + 1] （因为j从-1开始的） 经行比较。

   如果 s[i] 与 t[j + 1] 不相同，j就要从next数组里寻找下一个匹配的位置。

9. 如何判断在文本串s里出现了模式串t呢，如果j指向了模式串t的末尾，那么就说明模式串t完全匹配文本串s里的某个子串了。

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace CodeExercise
{
    public class StrStr
    {
        public static int Solve(string haystack, string needle)
        {
            if (needle.Length == 0)
            {
                return 0;
            }
            int result = -1;
            int needleLen = needle.Length;
            int haystackLen = haystack.Length;
            for (int i = 0; i < haystackLen; i++)
            {
                if (needleLen > haystackLen - i)
                {
                    return -1;
                }
                bool flag = true;
                for (int j = 0; j < needleLen; j++)
                {
                    if (needle[j] != haystack[i + j])
                    {
                        flag = false;
                        break;
                    }
                }
                if (flag)
                {
                    return i;
                }
            }
            return result;
        }

        public static int KMPSovle(string haystack, string needle)
        {
            //KMP的主要思想是当出现字符串不匹配时，可以知道一部分之前已经匹配的文本内容，可以利用这些信息避免从头再去做匹配了。
            //next数组就是一个前缀表（prefix table）。
            //前缀表是用来回退的，它记录了模式串与主串(文本串)不匹配的时候，模式串应该从哪里开始重新匹配。
            if (needle.Length == 0)
            {
                return 0;
            }
            int result = -1;
            int needleLen = needle.Length;
            int haystackLen = haystack.Length;
            //构造前缀表
            int[] prefixArr = new int[needleLen];
            int j = -1;
            prefixArr[0] = j;
            for (int i = 1; i < needleLen; i++)
            {
                while (j >= 0 && needle[i] != needle[j+1])
                {
                    j = prefixArr[j];
                }
                if (needle[i] == needle[j + 1])
                {
                    j++;
                }
                prefixArr[i] = j;
            }
            j = -1;
            for (int i = 0; i < haystackLen; i++)
            {
                while (j >= 0 && haystack[i] != needle[j+1])
                {
                    j = prefixArr[j];
                }
                if (haystack[i] == needle[j + 1])
                {
                    j++;
                }
                if (j == needleLen - 1)
                {
                    return i - needleLen + 1;
                }
            }
            return result;
        }
    }
}

```
