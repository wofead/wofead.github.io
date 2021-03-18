#### 盛最多水的容器

> 给定由若干 0 和 1 组成的数组 A。我们定义 N_i：从 A[0] 到 A[i] 的第 i 个子数组被解释为一个二进制数（从最高有效位到最低有效位）。
>
> 返回布尔值列表 answer，只有当 N_i 可以被 5 整除时，答案 answer[i] 为 true，否则为 false。
>
> **提示：**
>
> * `1 <= A.length <= 30000`
> * `A[i]` 为 `0` 或 `1`
> * 注意数组很长的时候会溢出

## 解题思路

这里存在一个问题，怎么证明余数M[i]可以代表N[i]\(到A[i]这里的二进制)这个值呢？这里使用归纳法。

当i=0是，由于N[0]=A[0]<5,所以M[0]=N[0]。

当i>0时，假设M[i-1]=N[i-1] mod 5成立，考虑N[i] mod 5 和 M[i]的值：
$$
N_i mod 5 = (N_{i-1} * 2+A[i]) mod 5 \\
	=(N_{i-1} * 2) mod 5 + A[i]) mod 5
\\
\\
M_i=(M_{i-1} * 2+A[i]) mod 5 \\
	=(N_{i-1} mod 5 * 2 + A[i]) mod 5 \\
	=(N_{i-1} mod 5 * 2 ) mod 5+ A[i]mod 5\\
	=(N_{i-1}* 2 ) mod 5+ A[i]mod 5
$$


## 解：

```c#
using System.Collections.Generic;

namespace CodeExercise
{
    public class PrefixesDivBy5
    {
        public static IList<bool> Solve(int[] A)
        {
            IList<bool> result = new List<bool>();
            int value = 0;
            for (int i = 0; i < A.Length; i++)
            {
                //数组很大的时候会导致value溢出，导致结果出错。
                //value = (value << 1) + A[i];
                //归纳法判断正确性
                value = ((value << 1) + A[i]) % 5;
                if (value == 0)
                {
                    result.Insert(i, true);
                }
                else
                {
                    result.Insert(i, false);
                }
            }
            return result;
        }
    }
}

```

