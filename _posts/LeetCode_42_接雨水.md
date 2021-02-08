# 缺失的第一个正整数

> 给定 *n* 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。
>
> 1. `n == height.length`
>2. `0 <= n <= 3 * 104`
> 3. ` 0 <= height[i] <= 105`

## 结题思路

1. 找到max
2. 加上结果，然后回溯

## 解：

```c#
public int Solve(int[] height)
{
    int n = height.Length;
    if (n == 0 || n == 1)
    {
        return 0;
    }
    int result = 0, left = 0, right = 1;
    while (right < n)
    {
        int rightMax = -1;
        int rightMaxIndex = n;
        while (right < n)
        {
            if (height[right] < height[left])
            {
                if (height[right] > height[right - 1] && (right + 1 > n - 1 || height[right] >= height[right + 1]))
                {
                    if (height[right] > rightMax)
                    {
                        rightMax = height[right];
                        rightMaxIndex = right;
                    }
                }
                right++;
            }
            else
            {
                rightMax = height[right];
                rightMaxIndex = right;
                break;
            }
        }
        if (rightMax > 0)
        {
            int theValue = Math.Min(height[left], rightMax);
            for (int i = left + 1; i < rightMaxIndex; i++)
            {
                int offset = theValue - height[i];
                result += offset > 0 ? offset : 0;
            }
        }
        left = rightMaxIndex;
        right = left + 1;
    }
    return result;
}

```

## 解2

不用像上个方法那样存储最大高度，而是用栈来跟踪可能储水的最长的条形块。使用栈就可以在一次遍历内完成计算。

我们在遍历数组时维护一个栈。如果当前的条形块小于或等于栈顶的条形块，我们将条形块的索引入栈，意思是当前的条形块被栈中的前一个条形块界定。如果我们发现一个条形块长于栈顶，我们可以确定栈顶的条形块被当前条形块和栈的前一个条形块界定，因此我们可以弹出栈顶元素并且累加答案到 `ans` 。

```c#
public int StackSolve(int[] height)
{
    int ans = 0, current = 0;
    Stack<int> stack = new Stack<int>();
    while(current < height.Length)
    {
        while (stack.Count != 0 && height[current] > height[stack.Peek()])
        {
            int top = stack.Pop();
            if (stack.Count == 0)
            {
                break;
            }
            int distance = current - stack.Peek() - 1;
            int boundHeight = Math.Min(height[current], height[stack.Peek()]) - height[top];
            ans += distance * boundHeight;
        }
        stack.Push(current++);
    }
    return ans;
}
```

## 解3

不从左和从右分开计算，我们想办法一次完成遍历。

只要`rightMax[i]>leftMax[i]`,积水高度将由`leftMax`决定，类似的`leftMax[i] > rightMax[i]`.

所以我们可以认为如果一端有更高的条形块（例如右端），积水的高度依赖于当前方向的高度（从左到右）。当我们发现另一侧（右侧）的条形块高度不是最高的，我们则开始从相反的方向遍历（从右到左）。

我们必须在遍历时维护`leftMax`和`rightMax`，但是我们现在可以使用两个指针交替进行，实现 1 次遍历即可完成。

```c#
public int TwoIndexSolve(int[] height)
{
    int left = 0, right = height.Length - 1;
    int ans = 0;
    int leftMax = 0, rightMax = 0;
    while (left < right)
    {
        if (height[left] < height[right])
        {
            if (height[left] >= leftMax)
            {
                leftMax = height[left];
            }
            else
            {
                ans += (leftMax - height[left]);
            }
            ++left;
        }
        else
        {
            if (height[right] >= rightMax)
            {
                rightMax = height[right];
            }
            else
            {
                ans += (rightMax - height[right]);
            }
            --right;
        }
    }
    return ans;
}
```

