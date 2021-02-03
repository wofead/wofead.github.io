# 解答的数独

> 编写一个程序，通过填充空格来解决数独问题。
>
> 一个数独的解法需**遵循如下规则**：
>
> 1. 数字 1-9 在每一行只能出现一次。
> 2. 数字 1-9 在每一列只能出现一次。
> 3. 数字 1-9 在每一个以粗实线分隔的 3x3 宫内只能出现一次。
>
> 空白格用 `'.'` 表示。

## 结题思路

1. 状态压缩，使用一个int来代表一个九宫格数据存储，转化为二进制
2. 回溯法
3. 找出所有需要填的位置
4. 找到一个可填选项最少的位置
5. 然后开始一个一个填写
6. 填写之后递归调用dfs
7. 成功就返回
8. 失败了，就重置为`.`,接着尝试下个数字。

## 解：

```c#
using System.Collections.Generic;

namespace Arithmatic
{
    public class ArraySolveSudoku
    {
        List<int> rows;
        List<int> cols;
        List<int> cells;
        /// <summary>
        /// 回溯加上状态压缩
        /// </summary>
        /// <param name="board"></param>
        public void Solve(char[][] board)
        {
            rows = new List<int>();
            cols = new List<int>();
            cells = new List<int>();
            for (int i = 0; i < 9; i++)
            {
                rows.Add(0);
                cols.Add(0);
                cells.Add(0);
            }
            int cnt = 0;
            for (int i = 0; i < board.Length; i++)
            {
                for (int j = 0; j < board[i].Length; j++)
                {
                    cnt += (board[i][j] == '.' ? 1 : 0);
                    if (board[i][j] == '.') continue;
                    int n = board[i][j] - '1';
                    int value = 1 << n;
                    rows[i] |= value;
                    cols[j] |= value;
                    cells[j / 3 * 3 + i / 3 % 3] |= value;
                }
            }
            Dfs(ref board, cnt);
        }

        int GetPositionStatus(int row, int col)
        {
            return rows[row] | cols[col] | cells[col / 3 * 3 + row / 3 % 3];
        }

        int[] GetNext(char[][] board)
        {
            int[] result = new int[2];
            int minCnt = 10;
            for (int i = 0; i < 9; i++)
            {
                for (int j = 0; j < 9; j++)
                {
                    if (board[i][j] != '.') continue;
                    int cur = GetPositionStatus(i, j);
                    int count = 9;
                    for (int k = 0; k < 9; k++)
                    {
                        int value = 1 << k;
                        if ((cur & value) == value) count--;
                    }
                    if (count >= minCnt) continue;
                    result[0] = i;
                    result[1] = j;
                    minCnt = count;
                }
            }
            return result;
        }

        void FillNum(int row, int col, int n, bool fillFlag)
        {
            if (fillFlag)
            {
                //return rows[row] | cols[col] | cells[col / 3 * 3 + row / 3 % 3];
                rows[row] |= (1 << n);
                cols[col] |= (1 << n);
                cells[col / 3 * 3 + row / 3 % 3] |= (1 << n);
            }
            else
            {
                rows[row] &= (~(1 << n));
                cols[col] &= (~(1 << n));
                cells[col / 3 * 3 + row / 3 % 3] &= (~(1 << n));
            }
        }

        bool Dfs(ref char[][] board, int cnt)
        {
            if (cnt == 0) return true;
            int[] next = GetNext(board);
            int bits = GetPositionStatus(next[0], next[1]);
            for (int i = 0; i < 9; i++)
            {
                int value = 1 << i;
                if ((bits & value) == value) continue;
                FillNum(next[0], next[1], i, true);
                board[next[0]][next[1]] = (char)(i + '1');
                if (Dfs(ref board, cnt - 1)) return true;
                {
                    board[next[0]][next[1]] = '.';
                    FillNum(next[0], next[1], i, false);
                }
            }
            return false;
        }
    }
}

```

## 省空间的算法

```java
public class Solution {
    public int longestValidParentheses(String s) {
        int maxans = 0;
        Deque<Integer> stack = new LinkedList<Integer>();
        stack.push(-1);
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == '(') {
                stack.push(i);
            } else {
                stack.pop();
                if (stack.empty()) {
                    stack.push(i);
                } else {
                    maxans = Math.max(maxans, i - stack.peek());
                }
            }
        }
        return maxans;
    }
}
```

## 不需要额外空间

在此方法中，我们利用两个计数器 `left` 和 `right` 。首先，我们从左到右遍历字符串，对于遇到的每个 `‘(’`，我们增加 `left` 计数器，对于遇到的每个 `‘)’` ，我们增加 `right` 计数器。每当 `left` 计数器与 `right` 计数器相等时，我们计算当前有效字符串的长度，并且记录目前为止找到的最长子字符串。当 `right` 计数器比 `left` 计数器大时，我们将 `left` 和 `right` 计数器同时变回 00。

这样的做法贪心地考虑了以当前字符下标结尾的有效括号长度，每次当右括号数量多于左括号数量的时候之前的字符我们都扔掉不再考虑，重新从下一个字符开始计算，但这样会漏掉一种情况，就是遍历的时候左括号的数量始终大于右括号的数量，即 (() ，这种时候最长有效括号是求不出来的。

解决的方法也很简单，我们只需要从右往左遍历用类似的方法计算即可，只是这个时候判断条件反了过来：

当 `left` 计数器比 `right` 计数器大时，我们将 `left` 和 `right` 计数器同时变回 00
当 `left` 计数器与 `right` 计数器相等时，我们计算当前有效字符串的长度，并且记录目前为止找到的最长子字符串

```java
class Solution {
public:
    int longestValidParentheses(string s) {
        int left = 0, right = 0, maxlength = 0;
        for (int i = 0; i < s.length(); i++) {
            if (s[i] == '(') {
                left++;
            } else {
                right++;
            }
            if (left == right) {
                maxlength = max(maxlength, 2 * right);
            } else if (right > left) {
                left = right = 0;
            }
        }
        left = right = 0;
        for (int i = (int)s.length() - 1; i >= 0; i--) {
            if (s[i] == '(') {
                left++;
            } else {
                right++;
            }
            if (left == right) {
                maxlength = max(maxlength, 2 * left);
            } else if (left > right) {
                left = right = 0;
            }
        }
        return maxlength;
    }
```

