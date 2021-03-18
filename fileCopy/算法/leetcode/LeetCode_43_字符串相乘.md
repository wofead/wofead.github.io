# 字符串相乘

> 给定两个以字符串形式表示的非负整数 `num1` 和 `num2`，返回 `num1` 和 `num2` 的乘积，它们的乘积也表示为字符串形式。
>
> 1. `num1 和 num2 的长度小于110。`
> 2. `num1 和 num2 只包含数字 0-9。`
> 3. ` num1 和 num2 均不以零开头，除非是数字 0 本身。`
> 4. **不能使用任何标准库的大数类型（比如 BigInteger）**或**直接将输入转换为整数来处理**。

## 结题思路

1. 两个数相乘，最大位数是这两个数字长度之和

2. 一个一个位的计算

3. ```c#
   
   int index2 = n2 - j - 1;
   int preValue = i - j;
   if (preValue < 0 || preValue > n1 - 1)
   {
       continue;
   }
   else
   {
   
       int index1 = n1 - 1 - preValue;
       int temp = (m2[index2] - zero) * (m1[index1] - zero);
       temp += sum;
       sum = temp % 10;
       addNum += temp / 10;
   }
   ```

   `idnex2`是第二个乘数的index，这个时候计算第一个乘数的位置

4. 首先判断是否能找到`index1`,i-j要大于等于0和小于n1-1

5. 得到`index1`

6. 注意进制的判断

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class StringMultiply
    {
        public string Solve(string num1, string num2)
        {
            int len1 = num1.Length;
            int len2 = num2.Length;
            int maxLen = len1 + len2;
            StringBuilder ans = new StringBuilder();
            List<char> charList = new List<char>();
            string m1, m2;
            int n1, n2;
            if (len1 >= len2)
            {
                m1 = num1;
                m2 = num2;
                n1 = len1;
                n2 = len2;
            }
            else
            {
                m1 = num2;
                m2 = num1;
                n1 = len2;
                n2 = len1;
            }
            char zero = '0';
            int lastAddNum = 0;
            for (int i = 0; i < maxLen; i++)
            {
                int addNum = 0;
                int sum = 0;
                for (int j = 0; j < n2; j++)
                {
                    int index2 = n2 - j - 1;
                    int preValue = i - j;
                    if (preValue < 0 || preValue > n1 - 1)
                    {
                        continue;
                    }
                    else
                    {

                        int index1 = n1 - 1 - preValue;
                        int temp = (m2[index2] - zero) * (m1[index1] - zero);
                        temp += sum;
                        sum = temp % 10;
                        addNum += temp / 10;
                    }
                }
                int value = (sum + lastAddNum) % 10;
                addNum += (sum + lastAddNum) / 10;
                charList.Add((char)(value + '0'));
                lastAddNum = addNum;
            }
            int index = charList.Count - 1;
            for (int i = charList.Count - 1; i > 0; i--)
            {
                if (charList[i] == '0')
                {
                    index--;
                }
                else
                {
                    break;
                }
            }
            for (int i = index; i >=0; i--)
            {
                ans.Append(charList[i]);
            }
            return ans.ToString();
        }
    }
}

```





