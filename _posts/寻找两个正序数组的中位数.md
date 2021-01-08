# 寻找两个正序数组的中位数

> 给定两个大小为 m 和 n 的正序（从小到大）数组 nums1 和 nums2。请你找出并返回这两个正序数组的中位数。
>
> 进阶：你能设计一个时间复杂度为 O(log (m+n)) 的算法解决此问题吗？
>
> **提示：**
>
> * nums1.length == m
> *  nums2.length == n
> * 0 <= m <= 1000
> * 0 <= n <= 1000
> * 1 <= m + n <= 2000
> * -106 <= nums1[i], nums2[i] <= 106

## 解题思路
- 寻找第k小的数字
- 比较k/2-1位置上的两个数字的大小
- 更新删除指针，更新k的值
- 判断k是否为1
- 然后得到结果

## 注意事项
- k值的更新为index+1
- 两个数组的前置指针越界问题，等于和大于处理方式不一样
- 当total length为偶数的时候，注意比较小的值的后面那个值和另一个数组值得大小
- 在更新k值的时候，要注意使用的是ff -f而不是index的值


## 解：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CodeExercise
{
    public class FindMedianSortedArrays
    {
        public static double Solve(int[] nums1, int[] nums2)
        {
double result = 0;
            int n1 = nums1.Length, n2 = nums2.Length;
            int totalLength = n1 + n2;
            int k = totalLength % 2 == 0 ? totalLength / 2 : totalLength / 2 + 1;
            int f1 = 0, f2 = 0;
            int ff1 = 0, ff2 = 0;

            if (n1 == 0)
            {
                result = n2 % 2 == 0 ? (nums2[n2 / 2] + nums2[n2 / 2 - 1]) / 2.0 : nums2[n2 / 2];
                return result;
            }
            if (n2 == 0)
            {
                result = n1 % 2 == 0 ? (nums1[n1 / 2] + nums1[n1 / 2 - 1]) / 2.0 : nums1[n1 / 2];
                return result;
            }
            while (true)
            {
                if (f1 > n1)
                {
                    result = totalLength % 2 == 0 ? (nums2[k + f2] + nums2[k + f2 + 1]) / 2.0 : nums2[k + f2];
                    break;
                }
                else if (f2 > n2)
                {
                    result = totalLength % 2 == 0 ? (nums1[k + f1] + nums1[k + f1 + 1]) / 2.0 : nums1[k + f1];
                    break;
                }
                if (f1 == n1)
                {
                    result = totalLength % 2 == 0 ? (nums2[k + f2] + nums2[k + f2 - 1]) / 2.0 : nums2[k + f2 - 1];
                    break;
                }
                else if (f2 == n2)
                {
                    result = totalLength % 2 == 0 ? (nums1[k + f1] + nums1[k + f1 - 1]) / 2.0 : nums1[k + f1 - 1];
                    break;
                }
                if (k == 1)
                {
                    if (totalLength % 2 == 0)
                    {
                        if (nums1[f1] <= nums2[f2] && f1 + 1 < n1)
                        {
                            result = (nums1[f1] + Math.Min(nums1[f1 + 1], nums2[f2])) / 2.0;
                        }
                        else if (nums1[f1] > nums2[f2] && f2 + 1 < n2)
                        {
                            result = (nums2[f2] + Math.Min(nums2[f2 + 1], nums1[f1])) / 2.0;
                        }
                        else
                        {
                            result = (nums1[f1] + nums2[f2]) / 2.0;
                        }
                    }
                    else
                    {
                        result = Math.Min(nums1[f1], nums2[f2]);
                    }
                    break;
                }
                int index = k / 2 - 1;
                if (f1 + index > n1 - 1)
                {
                    ff1 = n1 - 1;
                }
                ff1 = f1 + index > n1 - 1 ? n1 - 1 : f1 + index;
                ff2 = f2 + index > n2 - 1 ? n2 - 1 : f2 + index;
                if (nums1[ff1] <= nums2[ff2])
                {
                    f1 += index + 1;
                    index = ff1 - f1;
                }
                else
                {
                    f2 += index + 1;
                    index = ff2 - f2;
                }
                k -= index + 1;

            }
            return result;
        }
    }
}

```

## 官方解题思路

其本质思想还是通过寻找第k小的数字，只不过在偶数和奇数情况下有所不同，偶数的时候会寻找两次。

```c#
public static double OfficialSolve(int[] nums1, int[] nums2)
        {
            int length1 = nums1.Length, length2 = nums2.Length;
            int totalLength = length1 + length2;
            if (totalLength % 2 == 1)
            {
                int midIdex = totalLength / 2;
                double median = GetKthElement(nums1, nums2, midIdex + 1);
                return median;
            }
            else
            {
                int midIndex1 = totalLength / 2 - 1;
                int midIndex2 = totalLength / 2;
                double median = (GetKthElement(nums1, nums2, midIndex1 + 1) + GetKthElement(nums1, nums2, midIndex2 + 1)) / 2.0;
                return median;
            }
        }

        public static int GetKthElement(int[] nums1, int[] nums2, int k)
        {
            int length1 = nums1.Length, length2 = nums2.Length;
            int f1 = 0, ff1 = 0, f2 = 0, ff2 = 0;
            while (true)
            {
                if (f1 >= length1)
                {
                    if (f1 == length1)
                    {
                        return nums2[k + f2 - 1];

                    }
                    else
                    {
                        return nums2[k + f2];
                    }
                }
                if (f2 >= length2)
                {
                    if (f2 == length2)
                    {
                        return nums1[k + f1 - 1];

                    }
                    else
                    {
                        return nums1[k + f1];
                    }
                }
                if (k == 1)
                {
                    return Math.Min(nums1[f1], nums2[f2]);
                }
                int index = k / 2 - 1;
                ff1 = f1 + index >= length1 ? length1 - 1 : f1 + index;
                ff2 = f2 + index >= length2 ? length2 - 1 : f2 + index;
                if (nums1[ff1] <= nums2[ff2])
                {
                    index = ff1 - f1;
                    f1 += index + 1;
                }
                else
                {
                    index = ff2 - f2;
                    f2 += index + 1;
                }
                k -= index + 1;
            }
            return k;
        }
```

