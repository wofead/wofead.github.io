# 文本左右对齐

> 给定一个单词数组和一个长度 maxWidth，重新排版单词，使其成为每行恰好有 maxWidth 个字符，且左右两端对齐的文本。
>
> 你应该使用“贪心算法”来放置给定的单词；也就是说，尽可能多地往每行中放置单词。必要时可用空格 ' ' 填充，使得每行恰好有 maxWidth 个字符。
>
> 要求尽可能均匀分配单词间的空格数量。如果某一行单词间的空格不能均匀分配，则左侧放置的空格数要多于右侧的空格数。
>
> 文本的最后一行应为左对齐，且单词之间不插入额外的空格。
>
> **说明:**
>
> - 单词是指由非空格字符组成的字符序列。
> - 每个单词的长度大于 0，小于等于 *maxWidth*。
>   - 输入单词数组 `words` 至少包含一个单词。

## 结题思路

1. 遍历所有的字符串
2. 当加上这个字符串小于等于maxWidth的时候，累加
3. 否则分情况，count ==0，证明只有一个单词，直接拼接上空字符就可以了
4. 不等于0的时候，得到平均每个空格加塞几个空格，然后余数就是需要多加塞一个空格的数量
5. 最后加上最后一行


## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class ArrayFullJustify
    {
        public IList<string> Solve(string[] words, int maxWidth)
        {
            IList<string> res = new List<string>();
            int length = words.Length;
            StringBuilder item = new StringBuilder(maxWidth);
            item.Append(words[0]);
            int count = 0;
            for (int i = 1; i < length; i++)
            {
                if (item.Length + words[i].Length + 1 <= maxWidth)
                {
                    item.Append(' ');
                    item.Append(words[i]);
                    count++;
                }
                else
                {
                    int empty = maxWidth - item.Length;
                    if (count == 0)
                    {
                        item.Insert(item.Length, " ", empty);
                    }
                    else
                    {
                        int per = empty / count;
                        int onePlus = count - empty % count;
                        int index = item.Length;
                        for (int k = 0; k < count; k++)
                        {
                            string temp = words[i - k - 1];
                            index -= (temp.Length + 1);
                            if (k < onePlus)
                            {
                                item.Insert(index, " ", per);
                            }
                            else
                            {
                                item.Insert(index, " ", per + 1);
                            }
                        }
                    }
                    res.Add(item.ToString());
                    item.Clear();
                    item.Append(words[i]);
                    count = 0;
                }
            }
            if (item.Length > 0)
            {
                int empty = maxWidth - item.Length;
                item.Insert(item.Length, " ", empty);
                res.Add(item.ToString());
            }
            return res;
        }
    }
}

```





