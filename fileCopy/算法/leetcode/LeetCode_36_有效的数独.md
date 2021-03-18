# 有效的数独

> 判断一个 9x9 的数独是否有效。只需要根据以下规则，验证已经填入的数字是否有效即可。
>
> 1. 数字 1-9 在每一行只能出现一次。
> 2. 数字 1-9 在每一列只能出现一次。
> 3. 数字 1-9 在每一个以粗实线分隔的 3x3 宫内只能出现一次。

## 结题思路

1. 遍历三次，行一次，列一次，九宫格一次，HashSet需要一个。
2. 遍历一次，HashSet需要27个。

## 解：

```c#
using System.Collections.Generic;

namespace CodeExercise
{
    public class ArrayIsValidSudoku
    {
        public static bool Solve(char[][] board)
        {
            bool result = true;
            HashSet<char> vs = new HashSet<char>();
            for (int i = 0; i < 9; i++)
            {
                vs.Clear();
                for (int j = 0; j < 9; j++)
                {
                    if (board[i][j].Equals('.')) continue;
                    if (vs.Contains(board[i][j]))
                    {
                        return false;
                    }
                    vs.Add(board[i][j]);
                }
                vs.Clear();
                for (int j = 0; j < 9; j++)
                {
                    if (board[j][i].Equals('.')) continue;
                    if (vs.Contains(board[j][i]))
                    {
                        return false;
                    }
                    vs.Add(board[j][i]);
                }
                vs.Clear();
                int offsetX = i / 3 * 3;
                int offsetY = i % 3 * 3;
                for (int j = 0; j < 9; j++)
                {
                    int x = j / 3 + offsetX;
                    int y = j % 3 + offsetY;
                    if (board[x][y].Equals('.')) continue;
                    if (vs.Contains(board[x][y]))
                    {
                        return false;
                    }
                    vs.Add(board[x][y]);
                }
            }
            return result;
        }
    }
}

```
