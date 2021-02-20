# 八皇后问题II

> n 皇后问题 研究的是如何将 n 个皇后放置在 n×n 的棋盘上，并且使皇后彼此之间不能相互攻击。
>
> 给你一个整数 n ，返回所有不同的 n 皇后问题 的解决方案。
>
> 每一种解法包含一个不同的 n 皇后问题 的棋子放置方案，该方案中 'Q' 和 '.' 分别代表了皇后和空位。

## 结题思路

1. 使用flag进行标志，意味着这个地方被几个皇后覆盖
2. 更新flag，只需要更新当前行一下的raw
3. 找到能放皇后的位置，每个能放的都尝试一下
4. 知道放下所有的皇后

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class SolveNQueens
    {
        public IList<IList<string>> Solve(int n)
        {
            IList<IList<string>> res = new List<IList<string>>();
            int[,] flags = new int[n, n];
            List<string> combine = new List<string>();
            PutQueen(res, 0, flags, n, combine);
            return res;
        }

        public void PutQueen(IList<IList<string>> res, int curRaw, int[,] flags, int n, List<string> combine)
        {
            if (curRaw == n)
            {
                res.Add(new List<string>(combine));
                return;
            }
            for (int i = 0; i < n; i++)
            {
                if (flags[curRaw, i] == 0)
                {
                    StringBuilder item = new StringBuilder();
                    item.Append('.', n);
                    item[i] = 'Q';
                    combine.Add(item.ToString());
                    int left = i - 1;
                    int right = i + 1;
                    for (int j = curRaw + 1; j < n; j++)
                    {
                        flags[j, i] += 1;
                        if (left >= 0) flags[j, left] += 1;
                        if (right < n) flags[j, right] += 1;
                        left--;
                        right++;
                    }
                    PutQueen(res, curRaw + 1, flags, n, combine);
                    combine.Remove(item.ToString());
                    left = i - 1;
                    right = i + 1;
                    for (int j = curRaw + 1; j < n; j++)
                    {
                        flags[j, i] -= 1;
                        if (left >= 0) flags[j, left] -= 1;
                        if (right < n) flags[j, right] -= 1;
                        left--;
                        right++;
                    }
                }
            }
        }
        public int AnotherSolve(int n)
        {
            IList<IList<string>> res = new List<IList<string>>();
            int[,] flags = new int[n, n];
            List<string> combine = new List<string>();
            PutQueen(res, 0, flags, n, combine);
            return res.Count;
        }

    }
}

```



