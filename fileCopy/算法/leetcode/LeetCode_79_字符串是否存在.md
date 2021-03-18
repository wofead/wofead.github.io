# 字符串是否存在

> 给定一个二维网格和一个单词，找出该单词是否存在于网格中。
>
> 单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。
>

## 结题思路

动态规划，广度优先遍历的方式进行搜索判断。

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class DoubleArrayExit
    {
        public bool Solve(char[][] board, string word)
        {
            int rowLen = board.Length;
            int colLen = board[0].Length;
            bool[,] vis = new bool[rowLen, colLen];
            for (int i = 0; i < rowLen; i++)
            {
                for (int j = 0; j < colLen; j++)
                {
                    bool flag = Wfs(ref board, ref word, vis, 0, i, j);
                    if (flag) return true;
                }
            }
            return false;
        }

        public bool Wfs(ref char[][] board, ref string word, bool[,] vis, int index, int row, int col)
        {
            if (board[row][col] != word[index])
            {
                return false;
            }
            else if (index == word.Length - 1)
            {
                return true;
            }
            vis[row,col] = true;
            bool result = false;
            List<KeyValuePair<int, int>> directions = new List<KeyValuePair<int, int>>();
            directions.Add(new KeyValuePair<int, int>(0, 1));
            directions.Add(new KeyValuePair<int, int>(0, -1));
            directions.Add(new KeyValuePair<int, int>(1, 0));
            directions.Add(new KeyValuePair<int, int>(-1, 0));
            foreach (var dir in directions)
            {
                int newi = row + dir.Key, newj = col + dir.Value;
                if (newi >= 0 && newi < board.Length && newj >= 0 && newj < board[0].Length)
                {
                    if (!vis[newi,newj])
                    {
                        bool flag = Wfs(ref board, ref word, vis, index + 1, newi, newj);
                        if (flag)
                        {
                            result = true;
                            break;
                        }
                    }
                }
            }
            vis[row, col] = false;
            return result;
        }
    }
}

```

## 

