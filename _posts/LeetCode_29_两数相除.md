# 两数相除

> 给定两个整数，被除数 dividend 和除数 divisor。将两数相除，要求不使用乘法、除法和 mod 运算符。
>
> 返回被除数 dividend 除以除数 divisor 得到的商。
>
> 整数除法的结果应当截去（truncate）其小数部分，例如：truncate(8.345) = 8 以及 truncate(-2.7335) = -2
>
> 提示：
>
> 1. 被除数和除数均为 32 位有符号整数。
> 2. 除数不为 0。
> 3. 假设我们的环境只能存储 32 位有符号整数，其数值范围是 [−2^31, 2^31 − 1]。本题中，如果除法结果溢出，则返回 2^31 − 1。

## 结题思路

1. 开始想的是有多少个除数，一个一个的累加，结果超时即：`Solve`函数
2. 二分外加上倍乘，再扩一倍就好了，二分没有什么意义，还没有倍乘好
3. 倍乘加上负数处理方式

**算有多少个除数，可以增倍，第二种优化方案更好理解**

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CodeExercise
{
    public class Divide
    {
        //超时啦
        public static int Solve(int dividend, int divisor)
        {
            if (divisor == -1 && dividend == int.MinValue)
            {
                return int.MaxValue;
            }
            int result = 0;
            bool isNeg = true;
            if ((dividend > 0 && divisor > 0) || (dividend < 0 && divisor < 0))
            {
                isNeg = false;
            }
            dividend = Math.Abs(dividend);
            divisor = Math.Abs(divisor);
            while (divisor >= dividend)
            {
                result++;
                divisor -= dividend;
            }
            if (isNeg)
            {
                result -= (result + result);
            }
            return result;
        }

        public static int ImproveSolve(int dividend, int divisor)
        {
            bool isNeg = true;
            if ((dividend > 0 && divisor > 0) || (dividend < 0 && divisor < 0))
            {
                isNeg = false;
            }
            long x = dividend, y = divisor;
            if (dividend < 0) x = -x;
            if (divisor < 0) y = -y;
            long l = 0, r = x;
            while (l < r)
            {
                long mid = l + r + 1 >> 1;
                if (Mul(mid, y) <= x)
                {
                    l = mid;
                }
                else
                {
                    r = mid - 1;
                }
            }
            long ans = isNeg ? -l : l;
            if (ans > int.MaxValue || ans < int.MinValue) return int.MaxValue;
            return (int)ans;
        }

        public static long Mul(long a, long k)
        {
            long ans = 0;
            while (k > 0)
            {
                if ((k & 1) == 1) ans += a;
                k >>= 1;
                a += a;
            }
            return ans;
        }

        public static int ImproSolve(int dividend, int divisor)
        {
            bool sign = (dividend > 0) ^ (divisor > 0);
            int result = 0;
            if (dividend > 0)
            {
                dividend = -dividend;
            }
            if (divisor > 0)
            {
                divisor = -divisor;
            }
            while (dividend <= divisor)
            {
                int tempResult = -1;
                int tempDivisor = divisor;
                while (dividend <= (tempDivisor << 1))
                {
                    if (tempDivisor <= (int.MinValue >> 1))
                    {
                        break;
                    }
                    tempResult <<= 1;
                    tempDivisor <<= 1;
                }
                dividend -= tempDivisor;
                result += tempResult;
            }
            if (!sign)
            {
                if (result <= int.MinValue)
                {
                    return int.MaxValue;
                }
                result = -result;
            }
            return result;
        }
    }
}

```
