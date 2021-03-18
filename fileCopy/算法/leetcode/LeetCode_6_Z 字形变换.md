# Z 字形变换

> 将一个给定字符串 `s` 根据给定的行数 `numRows` ，以从上往下、从左到右进行 Z 字形排列。
>
> **提示：**
>
> * `1 <= s.length <= 1000`
> *  `s` 由英文字母（小写和大写）、`','` 和 `'.'` 组成
> *  `1 <= numRows <= 1000`

## 解题思路
- min(numRows,len(*s*)) 个列表来表示 Z 字形图案中的非空行
- 从左到右迭代 s*s*，将每个字符添加到合适的行。
- 使用当前行和当前方向这两个变量对合适的行进行跟踪。
- 向上移动到最上面的行或向下移动到最下面的行时，当前方向才会发生改变。


## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace CodeExercise
{
    public class ZConvert
    {
        public static string Solve(string s, int numRows)
        {
            if (numRows == 1) return s;
            int len = s.Length;
            int rowLen = Math.Min(numRows, len);
            List<StringBuilder> rows = new List<StringBuilder>();
            for (int i = 0; i < rowLen; i++)
            {
                rows.Add(new StringBuilder());
            }
            int curRow = 0;
            bool goingDown = false;
            for (int i = 0; i < len; i++)
            {
                rows[curRow].Append(s[i]);
                if (curRow == 0 || curRow == numRows - 1)
                {
                    goingDown = !goingDown;
                }
                curRow += goingDown ? 1 : -1;
            }
            StringBuilder ret = new StringBuilder();
            foreach (StringBuilder row in rows)
            {
                ret.Append(row);
            }
            return ret.ToString();
        }
    }
}

```

