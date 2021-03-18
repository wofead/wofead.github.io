# 串联所有单词的子串

> 给定一个字符串 s 和一些长度相同的单词 words。找出 s 中恰好可以由 words 中所有单词串联形成的子串的起始位置。
>
> 注意子串要与 words 中的单词完全匹配，中间不能有其他字符，但不需要考虑 words 中单词串联的顺序。
>

## 结题思路

1. 暴力解法。
2. 滑动窗口
3. 子串长度大小的窗口进行滑动
4. 匹配与不匹配
5. 匹配是否超额度，超额则移动左指针，直到不超额。
6. 判断count是否等于子串个数。

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class ArrayFindSubstring
    {
        //暴力解法
        public static IList<int> Solve(string s, string[] words)
        {
            IList<int> result = new List<int>();
            List<string> wordList = new List<string>();
            int strArrLen = words.Length;
            int itemStrLen = words[0].Length;
            int sLen = s.Length;
            for (int i = 0; i < sLen; i++)
            {
                if (strArrLen * itemStrLen > sLen - i)
                {
                    break;
                }
                wordList.Clear();
                for (int j = 0; j < strArrLen; j++)
                {
                    wordList.Add(words[j]);
                }
                bool isMath = true;
                for (int j = 0; j < strArrLen; j++)
                {
                    string str = s.Substring(i + j * itemStrLen, itemStrLen);
                    int index = wordList.IndexOf(str);
                    if (index != -1)
                    {
                        wordList.RemoveAt(index);
                    }
                    else
                    {
                        isMath = false;
                        break;
                    }
                }
                if (isMath)
                {
                    result.Add(i);
                    isMath = false;
                }
            }

            return result;
        }

        public static IList<int> ImproveSolve(string s, string[] words)
        {
            int wordsLen = words.Length;
            int oneWorldLen = words[0].Length;
            int wordsStrLen = wordsLen * oneWorldLen;
            int sLen = s.Length;
            IList<int> result = new List<int>();
            if (sLen == 0 || wordsLen == 0 || wordsStrLen > sLen)
            {
                return result;
            }
            Dictionary<string, int> wordsDic = new Dictionary<string, int>();
            foreach (var item in words)
            {
                if (s.IndexOf(item) <0)
                {
                    return result;
                }
                wordsDic[item] = GetDicValueNum(wordsDic, item) + 1;
            }
            for (int i = 0; i < oneWorldLen; i++)
            {
                int left = i, right = i, count = 0;
                Dictionary<string, int> subDic = new Dictionary<string, int>();
                while (right + oneWorldLen <= sLen)
                {
                    string word = s.Substring(right, oneWorldLen);
                    right += oneWorldLen;
                    if (!wordsDic.ContainsKey(word))
                    {
                        left = right;
                        subDic.Clear();
                        count = 0;
                    }
                    else{
                        subDic[word] = GetDicValueNum(subDic, word) + 1;
                        ++count;
                        while (GetDicValueNum(subDic, word) > GetDicValueNum(wordsDic, word))
                        {
                            string w = s.Substring(left, oneWorldLen);
                            subDic[w] = GetDicValueNum(subDic, w) - 1;
                            --count;
                            left += oneWorldLen;
                        }
                        //这里不要重置count，subDic以及设置left和right，有可能下一个字符串也满足
                        if (count == wordsLen) result.Add(left);
                    }
                }
            }
            return result;
        }

        public static int GetDicValueNum(Dictionary<string,int> dic, string value)
        {
            int num;
            if (!dic.ContainsKey(value))
            {
                num = 0;
            }
            else
            {
                num = dic[value];
            }
            return num;
        }
    }
}
```
