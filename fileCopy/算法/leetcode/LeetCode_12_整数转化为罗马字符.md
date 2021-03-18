#### 整数转化为罗马字符

> 罗马数字包含以下七种字符： `I`， `V`， `X`， `L`，`C`，`D` 和 `M`。
>
> 返回布尔值列表 answer，只有当 N_i 可以被 5 整除时，答案 answer[i] 为 true，否则为 false。
>
> 字符          数值
> I             1
> V             5
> X             10
> L             50
> C             100
> D             500
> M             1000
>
> 例如， 罗马数字 2 写做 II ，即为两个并列的 1。12 写做 XII ，即为 X + II 。 27 写做  XXVII, 即为 XX + V + II 。
>
> 通常情况下，罗马数字中小的数字在大的数字的右边。但也存在特例，例如 4 不写做 IIII，而是 IV。数字 1 在数字 5 的左边，所表示的数等于大数 5 减小数 1 得到的数值 4 。同样地，数字 9 表示为 IX。这个特殊的规则只适用于以下六种情况：
>
> - `I` 可以放在 `V` (5) 和 `X` (10) 的左边，来表示 4 和 9。
> - `X` 可以放在 `L` (50) 和 `C` (100) 的左边，来表示 40 和 90。 
> - `C` 可以放在 `D` (500) 和 `M` (1000) 的左边，来表示 400 和 900。
>
> 给定一个整数，将其转为罗马数字。输入确保在 1 到 3999 的范围内。

## 解题思路

不断的除以10来判断取那个字符，然后分情况来插入字符。

## 解：

```c#
using System.Collections.Generic;
using System.Text;

namespace CodeExercise
{
    public class IntToRoman
    {
        public static string Solve(int num)
        {
            Dictionary<int, char> romanChar = new Dictionary<int, char>();
            romanChar[1] = 'I';
            romanChar[5] = 'V';
            romanChar[10] = 'X';
            romanChar[50] = 'L';
            romanChar[100] = 'C';
            romanChar[500] = 'D';
            romanChar[1000] = 'M';
            int one = 1;
            int five = 5;
            StringBuilder result = new StringBuilder();
            while (num != 0)
            {
                int mod = num % 10;
                num = num / 10;
                if (0 < mod && mod < 4)
                {
                    for (int i = 0; i < mod; i++)
                    {
                        result.Insert(0, romanChar[one]);
                    }
                }
                else if (mod == 4)
                {
                    result.Insert(0, romanChar[five]);
                    result.Insert(0, romanChar[one]);
                }
                else if (mod == 5) result.Insert(0, romanChar[five]);
                else if (mod == 9)
                {
                    result.Insert(0, romanChar[one * 10]);
                    result.Insert(0, romanChar[one]);
                }
                else if (5 < mod && mod < 9)
                {
                    for (int i = 0; i < mod % 5; i++)
                    {
                        result.Insert(0, romanChar[one]);
                    }
                    result.Insert(0, romanChar[five]);
                }
                one *= 10;
                five *= 10;
            }
            return result.ToString();
        }
    }
}


```

## 官方解法

```c#
public static string OfficialSolve(int num)
{
    int[] values = { 1000, 900, 500, 400, 100, 90, 50, 40, 10, 9, 5, 4, 1 };
    string[] symbols = { "M", "CM", "D", "CD", "C", "XC", "L", "XL", "X", "IX", "V", "IV", "I" };
    StringBuilder result = new StringBuilder();
    for (int i = 0; i < values.Length && num >= 0; i++)
    {
        while (values[i] <= num)
        {
            num -= values[i];
            result.Append(symbols[i]);
        }
    }
    return result.ToString();
}
```

