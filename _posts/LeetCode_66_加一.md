# 加一

> 给定一个由 整数 组成的 非空 数组所表示的非负整数，在该数的基础上加一。
>
> 最高位数字存放在数组的首位， 数组中每个元素只存储单个数字。
>
> 你可以假设除了整数 0 之外，这个整数不会以零开头。
>

## 结题思路

1. 从后往前判断，直到index处的值不为9
2. 为9，赋值为0
3. 如果index<0,resize,0处变为1
4. 否则index处+1


## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class ArrayPlusOne
    {
        public int[] Solve(int[] digits)
        {
            int length = digits.Length;
            int index = length - 1;
            while (index >= 0 && digits[index] == 9)
            {
                digits[index] = 0;
                index--;
            }
            if (index < 0)
            {
                Array.Resize<int>(ref digits, length + 1);
                digits[0] = 1;
            }
            else
            {
                digits[index] += 1;
            }
            return digits;
        }
    }
}

```





