# 二进制求和

> 给你两个二进制字符串，返回它们的和（用二进制表示）。
>
> 输入为 **非空** 字符串且只包含数字 `1` 和 `0`。

## 结题思路

1. 从后往前一起相加
2. 亦或放到`stringbuilder`中


## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{
    public class StringAddBinary
    {
        public string Solve(string a, string b)
        {
            StringBuilder res = new StringBuilder();
            int len1 = a.Length - 1;
            int len2 = b.Length - 1;
            int carry = 0;
            while (len1 >= 0 || len2 >= 0)
            {
                int theA = len1 < 0 ? 0 : (int)(a[len1] - '0');
                int theB = len2 < 0 ? 0 : (int)(b[len2] - '0');
                res.Insert(0, (char)(((theA ^ theB) ^ carry) + '0'));
                if (theA + theB + carry >= 2)
                {
                    carry = 1;
                }
                else
                {
                    carry = 0;
                }
                len1--;
                len2--;
            }
            if (carry == 1)
            {
                res.Insert(0, (char)(carry + '0'));
            }
            return res.ToString();
        }
    }
}

```





