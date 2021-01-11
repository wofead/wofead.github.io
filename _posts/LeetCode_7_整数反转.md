# 整数反转

> 给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。
>
> **注意：**假设我们的环境只能存储得下 32 位的有符号整数，则其数值范围为[−231, 231 − 1]。请根据这个假设，如果反转后整数溢出那么就返回 0。
>
> **提示：**
>
> * `-231 <= x <= 231 - 1`

## 解题思路
- 反转整数的方法可以与反转字符串进行类比。

- 重复“弹出” x*x* 的最后一位数字，并将它“推入”到 \text{rev}rev 的后面。最后，\text{rev}rev 将与 x*x* 相反。

- 没有辅助堆栈 / 数组的帮助下 “弹出” 和 “推入” 数字，我们可以使用数学方法。

- ```c#
  //pop operation:
  pop = x % 10;
  x /= 10;
  
  //push operation:
  temp = rev * 10 + pop;
  rev = temp;
  ```

## 注意：

这种方法很危险，`temp = rev * 10 + pop;`会导致溢出。

1. 如果`temp = rev * 10 + pop;`导致溢出，那么一定有 `rev > int.MaxValue / 10`.
2. 如果 `rev > int.MaxValue / 10`.那么`temp = rev * 10 + pop;`一定溢出。
3. 如果`rev == int.MaxValue / 10`,那么只要`pop>7`，`temp = rev * 10 + pop;`一定溢出。
4. 负数类似，只不过7变成-8.


## 解：

```c#
namespace CodeExercise
{
    public class IntReverse
    {
        public static int Solve(int x)
        {
            int rev = 0;
            while (x != 0)
            {
                int pop = x % 10;
                x /= 10;
                if (rev > int.MaxValue / 10 || (rev == int.MaxValue / 10 && pop > 7)) return 0;
                if (rev < int.MinValue / 10 || (rev == int.MinValue / 10 && pop < -8)) return 0;
                rev = rev * 10 + pop;
            }
            return rev;
        }
    }
}

```

