#### 盛最多水的容器

> 给你 n 个非负整数 a1，a2，...，an，每个数代表坐标中的一个点 (i, ai) 。在坐标内画 n 条垂直线，垂直线 i 的两个端点分别为 (i, ai) 和 (i, 0) 。找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。
>
> **提示：**
>
> * `n = height.length`
> * `2 <= n <= 3 * 104`
> * `0 <= height[i] <= 3 * 104`

## 解题思路

两种解法，一种是暴力解法，计算每个i到len之间的距离，然后输出最大面积。

还有一种是双指针，前置指针与后置指针，因为这个区间永远是连续的，并且是不可拆分的。

通过移动左指针和右指针就可以得到结果，在移动的过程中，移动那个矮的，知道左指针和右指针重合。

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CodeExercise
{
    public class MaxArea
    {
        public static int Solve(int[] height)
        {
            int result = 0;
            int len = height.Length;
            for (int i = 0; i < len; i++)
            {
                for (int j = i + 1; j < len; j++)
                {
                    int area = Math.Min(height[i], height[j]) * (j - i);
                    if (area > result)
                    {
                        result = area;
                    }
                }
            }
            return result;
        }
        //双指针，因为总是连续的i组成最大的面积，且这个最大的区域一定在中间
        public static int OfficialSolve(int[] height)
        {
            int len = height.Length;
            int l = 0, r = len - 1;
            int ans = 0;
            while (l < r)
            {
                int area = (r - l) * Math.Min(height[l], height[r]);
                ans = Math.Max(area, ans);
                if (height[l] <= height[r])
                {
                    ++l;
                }
                else
                {
                    --r;
                }
            }
            return ans;
        }
    }
}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CodeExercise
{
    public class IsMatch
    {
        public static bool ErrorSolve(string s, string p)
        {
            //1表示*状态，2表示.状态
            char k = ' ';
            int index = 0;
            int repeatNum = 0;
            int flag = 0;
            if (index == p.Length) flag = -1;
            else flag = ChangeFlag(p, ref index, ref k);
            for (int i = 0; i < s.Length; i++)
            {
                if (flag == -1) return false;
                if (flag == 1 && (k == '.' || s[i] == k))
                {
                    repeatNum++;
                    continue;
                }
                if (flag == 1 && s[i] != k)
                {
                    index++;
                    if (index == p.Length) flag = -1;
                    else flag = ChangeFlag(p, ref index, ref k);
                    repeatNum = 0;
                }
                if (flag == 1 && (k == '.' || s[i] == k))
                {
                    repeatNum++;
                    continue;
                }
                if (flag == 2)
                {
                    index++;
                    if (index == p.Length) flag = -1;
                    else flag = ChangeFlag(p, ref index, ref k);
                    continue;
                }
                else
                {
                    if (flag != -1 && s[i] == p[index])
                    {
                        index++;
                        if (index == p.Length) flag = -1;
                        else flag = ChangeFlag(p, ref index, ref k);
                    }
                    else
                    {
                        return false;
                    }
                }
            }
            if (flag == -1)
            {
                return true;
            }
            if (flag == 1)
            {
                for (int i = p.Length - 1, j = s.Length - 1; i > index; i--, j--)
                {
                    if (p[i] == '*')
                    {
                        repeatNum += 2;
                        j += 1;
                        i -= 1;
                        continue;
                    }
                    if (j < 0 || (p[i] != s[j] && p[i] != '.') || p.Length - i > repeatNum)
                    {
                        return false;
                    }
                }
                return true;

            }
            return false;
        }

        public static int ChangeFlag(string p, ref int index, ref char k)
        {
            int flag = 0;
            if (index < p.Length - 1)
            {
                if (p[index + 1] == '*')
                {
                    flag = 1;
                    k = p[index];
                    index++;
                    return flag;
                }
            }
            if (p[index] == '.')
            {
                flag = 2;
            }

            return flag;
        }

        public static bool Solve(string s, string p)
        {
            int m = s.Length;
            int n = p.Length;
            bool[,] f = new bool[m+1,n+1];
            //f[i,j]表示s的前i个字符与p的前j个字符是否能够匹配
            f[0,0] = true;
            for (int i = 0; i <= m; ++i)
            {
                for (int j = 1; j <= n; ++j)
                {
                    if (p[j - 1] == '*')
                    {
                        f[i,j] = f[i,j - 2];
                        if (matches(s, p, i, j - 1))
                        {
                            f[i,j] = f[i,j] || f[i - 1,j];
                        }
                    }
                    else
                    {
                        if (matches(s, p, i, j))
                        {
                            f[i,j] = f[i - 1,j - 1];
                        }
                    }
                }
            }
            return f[m,n];
        }

        public static bool matches(string s, string p, int i, int j)
        {
            if (i == 0)
            {
                return false;
            }
            if (p[j - 1] == '.')
            {
                return true;
            }
            return s[i - 1] == p[j - 1];
        }
    }
}



```

