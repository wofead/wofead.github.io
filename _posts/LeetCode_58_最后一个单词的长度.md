# 最后一个单词的长度

> 给你一个字符串 s，由若干单词组成，单词之间用空格隔开。返回字符串中最后一个单词的长度。如果不存在最后一个单词，请返回 0 。
>
> 单词 是指仅由字母组成、不包含任何空格字符的最大子字符串。
>

## 结题思路

1. 从后往前遍历
2. 找到开始的那个非空格的字符
3. 直到再次遇到空格字符
4. 或者最后加上

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class LengthOfLastWord
    {
        public int Solve(string s)
        {
            int length = s.Length;
            int startIndex = -1;
            for (int i = length - 1; i >= 0; i--)
            {
                if (s[i] == ' ')
                {
                    if (startIndex < 0)
                    {
                        continue;
                    }
                    else
                    {
                        return startIndex - i;
                    }
                }
                else if (startIndex < 0)
                {
                    startIndex = i;
                }
            }
            return startIndex + 1;
        }
    }
}

```



