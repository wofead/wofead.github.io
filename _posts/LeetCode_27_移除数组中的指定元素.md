# 删除数组中的指定元素

> 给你一个数组 nums 和一个值 val，你需要 原地 移除所有数值等于 val 的元素，并返回移除后数组的新长度。
>
> 不要使用额外的数组空间，你必须仅使用 O(1) 额外空间并 原地 修改输入数组。
>
> 元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。
>

## 结题思路

1. 通过使用一个指针，来指向上次赋值的位置
2. 当不是目标值的时候就赋值，更新指针位置
3. 重新设置数组大小

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
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
            int index = 0;
            int preIndex = 0;
            bool init = true;
            for (int i = 0; i < haystackLen; i++)
            {
                if (haystack[i] == needle[index])
                {
                    index++;
                    if (index == needleLen)
                    {
                        return i - index + 1;
                    }
                }
                else
                {
                    index -= preIndex;
                    preIndex = 0;
                    init = true;
                }
                if (!init && haystack[i - 1] == needle[preIndex])
                {
                    preIndex++;
                }
                else
                {
                    init = false;
                    preIndex = 0;
                }
            }
            return result;
        }
    }
}

```
