# 组合

> 给定两个整数 *n* 和 *k*，返回 1 ... *n* 中所有可能的 *k* 个数的组合。
> 

## 结题思路1

使用动态规划的方式。

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class Combine
    {
        public IList<IList<int>> Solve(int n, int k)
        {
            IList<IList<int>> res = new List<IList<int>>();
            List<int> combine = new List<int>();
            ComibeArr(combine, res, n, k, 0, k);
            return res;
        }

        public void ComibeArr(List<int> combine, IList<IList<int>> res, int n, int k, int index, int remind)
        {
            if (remind == 0)
            {
                res.Add(new List<int>(combine));
                return;
            }
            for (int i = 1; i <= n - index; i++)
            {
                combine.Add(index + i);
                ComibeArr(combine, res, n, k, index + i, remind - 1);
                combine.Remove(index + i);
            }
        }
    }
}

```

## 结题思路2

#### 非递归（字典序法）实现组合型枚举

这里的非递归版不是简单的用栈模拟递归转化为非递归：我们希望通过合适的手段，消除递归栈带来的额外空间代价。

假设我们把原序列中被选中的位置记为 1，不被选中的位置记为 0，对于每个方案都可以构造出一个二进制数。

我们可以看出「对应的二进制数」一列包含了由 k 个 1 和 n - k 个 0 组成的所有二进制数，并且按照字典序排列。这给了我们一些启发，我们可以通过某种方法枚举，使得生成的序列是根据字典序递增的。我们可以考虑我们一个二进制数数字 x，它由 k 个 11 和 n - k 个 0 组成，如何找到它的字典序中的下一个数字next(x)，这里分两种情况：

* 规则一：x 的最低位为 1，这种情况下，如果末尾由 t 个连续的 1，我们直接将倒数第 t 位的 1 和倒数第 t + 1位的 0 替换，就可以得到 next(x)。如 0011 →0101，0101→0110，1001→1010，1001111→1010111。
* 规则二：x 的最低位为 0，这种情况下，末尾有 t 个连续的 0，而这 t 个连续的 0 之前有 m 个连续的 1，我们可以将倒数第t+m 位置的 1 和倒数第 t + m + 1位的 0 对换，然后把倒数第t+1 位到倒数第 t+m−1 位的 1 移动到最低位。如 0110→1001，11010→1100，1011100→1100011。

至此，我们可以写出一个朴素的程序，用一个长度为n 的 0/1 数组来表示选择方案对应的二进制数，初始状态下最低的 k 位全部为 1，其余位置全部为 0，然后不断通过上述方案求 next，就可以构造出所有的方案。

```java
class Solution {
    List<Integer> temp = new ArrayList<Integer>();
    List<List<Integer>> ans = new ArrayList<List<Integer>>();

    public List<List<Integer>> combine(int n, int k) {
        List<Integer> temp = new ArrayList<Integer>();
        List<List<Integer>> ans = new ArrayList<List<Integer>>();
        // 初始化
        // 将 temp 中 [0, k - 1] 每个位置 i 设置为 i + 1，即 [0, k - 1] 存 [1, k]
        // 末尾加一位 n + 1 作为哨兵
        for (int i = 1; i <= k; ++i) {
            temp.add(i);
        }
        temp.add(n + 1);
        
        int j = 0;
        while (j < k) {
            ans.add(new ArrayList<Integer>(temp.subList(0, k)));
            j = 0;
            // 寻找第一个 temp[j] + 1 != temp[j + 1] 的位置 t
            // 我们需要把 [0, t - 1] 区间内的每个位置重置成 [1, t]
            while (j < k && temp.get(j) + 1 == temp.get(j + 1)) {
                temp.set(j, j + 1);
                ++j;
            }
            // j 是第一个 temp[j] + 1 != temp[j + 1] 的位置
            temp.set(j, temp.get(j) + 1);
        }
        return ans;
    }
}
```

