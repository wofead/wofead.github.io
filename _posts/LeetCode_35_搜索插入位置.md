# 在排序数组中查找元素的第一个和最后一个位置

> 给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。
>
> 你可以假设数组中无重复元素。

## 结题思路

1. 二分法查找目标值
2. 当没有找到时，right大于left，返回right
3. left大于right，返回left。

## 解：

```c#
using System;

namespace CodeExercise
{
    public class ArraySearchInsert
    {
        public static int Solve(int[] nums, int target)
        {
            int len = nums.Length;
            int left = 0, right = len - 1;
            while (left <= right)
            {
                int mid = (left + right) / 2;
                int midValue = nums[mid];
                if (midValue == target)
                {
                    return mid;
                }
                else if (midValue < target)
                {
                    left = mid + 1;
                }
                else
                {
                    right = mid - 1;
                }
            }
            return Math.Max(left, right);
        }
    }
}

```

## 省空间的算法

```java
public class Solution {
    public int longestValidParentheses(String s) {
        int maxans = 0;
        Deque<Integer> stack = new LinkedList<Integer>();
        stack.push(-1);
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == '(') {
                stack.push(i);
            } else {
                stack.pop();
                if (stack.empty()) {
                    stack.push(i);
                } else {
                    maxans = Math.max(maxans, i - stack.peek());
                }
            }
        }
        return maxans;
    }
}
```

## 不需要额外空间

在此方法中，我们利用两个计数器 `left` 和 `right` 。首先，我们从左到右遍历字符串，对于遇到的每个 `‘(’`，我们增加 `left` 计数器，对于遇到的每个 `‘)’` ，我们增加 `right` 计数器。每当 `left` 计数器与 `right` 计数器相等时，我们计算当前有效字符串的长度，并且记录目前为止找到的最长子字符串。当 `right` 计数器比 `left` 计数器大时，我们将 `left` 和 `right` 计数器同时变回 00。

这样的做法贪心地考虑了以当前字符下标结尾的有效括号长度，每次当右括号数量多于左括号数量的时候之前的字符我们都扔掉不再考虑，重新从下一个字符开始计算，但这样会漏掉一种情况，就是遍历的时候左括号的数量始终大于右括号的数量，即 (() ，这种时候最长有效括号是求不出来的。

解决的方法也很简单，我们只需要从右往左遍历用类似的方法计算即可，只是这个时候判断条件反了过来：

当 `left` 计数器比 `right` 计数器大时，我们将 `left` 和 `right` 计数器同时变回 00
当 `left` 计数器与 `right` 计数器相等时，我们计算当前有效字符串的长度，并且记录目前为止找到的最长子字符串

```java
class Solution {
public:
    int longestValidParentheses(string s) {
        int left = 0, right = 0, maxlength = 0;
        for (int i = 0; i < s.length(); i++) {
            if (s[i] == '(') {
                left++;
            } else {
                right++;
            }
            if (left == right) {
                maxlength = max(maxlength, 2 * right);
            } else if (right > left) {
                left = right = 0;
            }
        }
        left = right = 0;
        for (int i = (int)s.length() - 1; i >= 0; i--) {
            if (s[i] == '(') {
                left++;
            } else {
                right++;
            }
            if (left == right) {
                maxlength = max(maxlength, 2 * left);
            } else if (left > right) {
                left = right = 0;
            }
        }
        return maxlength;
    }
```

