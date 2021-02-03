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

