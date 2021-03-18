# 编辑距离

> 给你两个单词 word1 和 word2，请你计算出将 word1 转换成 word2 所使用的最少操作数 。
>
> 你可以对一个单词进行如下三种操作：
>
> 插入一个字符
> 删除一个字符
> 替换一个字符
>

## 结题思路

1. 题目给定了两个单词，设为 `A` 和 `B`，这样我们就能够六种操作方法。

2. 但我们可以发现，如果我们有单词 `A` 和单词 `B`：

   * 对单词 A 删除一个字符和对单词 B 插入一个字符是等价的。例如当单词 A 为 doge，单词 B 为 dog 时，我们既可以删除单词 A 的最后一个字符 e，得到相同的 dog，也可以在单词 B 末尾添加一个字符 e，得到相同的 doge；

   * 同理，对单词 B 删除一个字符和对单词 A 插入一个字符也是等价的；

   * 对单词 A 替换一个字符和对单词 B 替换一个字符是等价的。例如当单词 A 为 bat，单词 B 为 cat 时，我们修改单词 A 的第一个字母 b -> c，和修改单词 B 的第一个字母 c -> b 是等价的。

3. 这样以来，本质不同的操作实际上只有三种：

   - 在单词 `A` 中插入一个字符；
   - 在单词 `B` 中插入一个字符；
   - 修改单词 `A` 的一个字符。

4. 这样以来，我们就可以把原问题转化为规模较小的子问题。我们用 A = horse，B = ros 作为例子，来看一看是如何把这个问题转化为规模较小的若干子问题的。

   * 在单词 A 中插入一个字符：如果我们知道 horse 到 ro 的编辑距离为 a，那么显然 horse 到 ros 的编辑距离不会超过 a + 1。这是因为我们可以在 a 次操作后将 horse 和 ro 变为相同的字符串，只需要额外的 1 次操作，在单词 A 的末尾添加字符 s，就能在 a + 1 次操作后将 horse 和 ro 变为相同的字符串；

   * 在单词 B 中插入一个字符：如果我们知道 hors 到 ros 的编辑距离为 b，那么显然 horse 到 ros 的编辑距离不会超过 b + 1，原因同上；

   * 修改单词 A 的一个字符：如果我们知道 hors 到 ro 的编辑距离为 c，那么显然 horse 到 ros 的编辑距离不会超过 c + 1，原因同上。

5. 那么从 `horse` 变成 `ros` 的编辑距离应该为 `min(a + 1, b + 1, c + 1)`。

6. 注意：为什么我们总是在单词 A 和 B 的末尾插入或者修改字符，能不能在其它的地方进行操作呢？答案是可以的，但是我们知道，操作的顺序是不影响最终的结果的。例如对于单词 cat，我们希望在 c 和 a 之间添加字符 d 并且将字符 t 修改为字符 b，那么这两个操作无论为什么顺序，都会得到最终的结果 cdab。

7. 因此，我们就可以使用动态规划来解决这个问题了。我们用 `D[i][j]` 表示 `A` 的前 `i` 个字母和 `B` 的前 `j` 个字母之间的编辑距离。

8. 如上所述，当我们获得 `D[i][j-1]`，`D[i-1][j]` 和 `D[i-1][j-1]` 的值之后就可以计算出 `D[i][j]`。

   * D[i][j-1] 为 A 的前 i 个字符和 B 的前 j - 1 个字符编辑距离的子问题。即对于 B 的第 j 个字符，我们在 A 的末尾添加了一个相同的字符，那么 D[i][j] 最小可以为 D[i][j-1] + 1；

   * D[i-1][j] 为 A 的前 i - 1 个字符和 B 的前 j 个字符编辑距离的子问题。即对于 A 的第 i 个字符，我们在 B 的末尾添加了一个相同的字符，那么 D[i][j] 最小可以为 D[i-1][j] + 1；

   * D[i-1][j-1] 为 A 前 i - 1 个字符和 B 的前 j - 1 个字符编辑距离的子问题。即对于 B 的第 j 个字符，我们修改 A 的第 i 个字符使它们相同，那么 D[i][j] 最小可以为 D[i-1][j-1] + 1。特别地，如果 A 的第 i 个字符和 B 的第 j 个字符原本就相同，那么我们实际上不需要进行修改操作。在这种情况下，D[i][j] 最小可以为 D[i-1][j-1]。

9. 那么我们可以写出如下的状态转移方程：

   * 若 `A` 和 `B` 的最后一个字母相同：
     $$
     D[i][j] = min(D[i][j-1]+1,D[i-1][j]+1,D[i-1][j-1]) \\
     = 1+min(D[i][j-1],D[i-1][j],D[i-1][j-1]-1)
     $$
     

   * 若 `A` 和 `B` 的最后一个字母不同：
     $$
     D[i][j] = 1+min(D[i][j-1],D[i-1][j],D[i-1][j-1])
     $$
     

   

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class StringMinDistance
    {
        public int minTime;
        public int Solve(string word1, string word2)
        {
            if (word2.Length == 0) return word1.Length;
            if (word1.Length == 0) return word2.Length;
            minTime = word1.Length;
            Change(word1, word2, 0, 0, 0, true);
            return minTime;
        }

        public void Change(string word1, string word2, int index1, int index2, int time, bool compare)
        {
            if (index1 == word1.Length && index2 == word2.Length)
            {
                minTime = Math.Min(minTime, time);
                return;
            }
            if (index1 == word1.Length)
            {
                Change(word1, word2, index1, index2 + 1, time + 1, false);
                return;
            }
            if (index2 == word2.Length)
            {
                Change(word1, word2, index1 + 1, index2, time + 1, false);
                return;
            }
            if (word1[index1] == word2[index2])
            {
                Change(word1, word2, index1 + 1, index2 + 1, time, true);
            }
            else
            {
                //删
                Change(word1, word2, index1 + 1, index2, time + 1, true);
                //增加
                Change(word1, word2, index1, index2 + 1, time + 1, true);
                //修改
                Change(word1, word2, index1 + 1, index2 + 1, time + 1, true);
            }

        }

        public int OfficialSolve(string word1, string word2)
        {
            int n = word1.Length;
            int m = word2.Length;

            // 有一个字符串为空串
            if (n * m == 0)
            {
                return n + m;
            }

            // DP 数组
            int[,] D = new int[n + 1,m + 1];

            // 边界状态初始化
            for (int i = 0; i < n + 1; i++)
            {
                D[i,0] = i;
            }
            for (int j = 0; j < m + 1; j++)
            {
                D[0,j] = j;
            }

            // 计算所有 DP 值
            for (int i = 1; i < n + 1; i++)
            {
                for (int j = 1; j < m + 1; j++)
                {
                    int left = D[i - 1,j] + 1;
                    int down = D[i,j - 1] + 1;
                    int left_down = D[i - 1,j - 1];
                    if (word1[i - 1] != word2[j - 1])
                    {
                        left_down += 1;
                    }
                    D[i,j] = Math.Min(left, Math.Min(down, left_down));
                }
            }
            return D[n,m];
        }
    }
}

```





