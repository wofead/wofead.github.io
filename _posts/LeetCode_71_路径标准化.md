# 路径标准化

> 假设你正在爬楼梯。需要 *n* 阶你才能到达楼顶。
>
> 每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？
>
> **注意：**给定 *n* 是一个正整数。

## 结题思路

1. 斐波那契数列类似。
2. 一阶为1，二阶为2，三阶是1和2之和。
3. 4阶是2与3之和。


## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class StringSimplifyPath
    {
        public enum Dir
        {
            Dot,
            Double,
            Char,
            Sprit
        }
        public string Solve(string path)
        {
            int length = path.Length;
            Stack<string> dirStack = new Stack<string>();
            dirStack.Push("/");
            StringBuilder item = new StringBuilder();
            Dir state = Dir.Sprit;
            for (int i = 1; i <= length; i++)
            {
                char value = i == length ? '/' : path[i];
                if (value == '/')
                {
                    if (state == Dir.Char)
                    {
                        item.Append('/');
                        dirStack.Push(item.ToString());
                    }
                    else if(state != Dir.Sprit && state != Dir.Dot)
                    {
                        if (dirStack.Count > 1)
                        {
                            dirStack.Pop();
                        }
                    }
                    state = Dir.Sprit;
                    item.Clear();
                }
                else if (value == '.')
                {
                    item.Append(path[i]);
                    if (state == Dir.Dot)
                    {
                        state = Dir.Double;
                    }
                    else if(state == Dir.Double)
                    {
                        state = Dir.Char;
                    }
                    else
                    {
                        state = Dir.Dot;
                    }
                }
                else
                {
                    state = Dir.Char;
                    item.Append(path[i]);
                }
            }
            string lastStr = dirStack.Pop();
            while (dirStack.Count != 0)
            {
                item.Insert(0, dirStack.Pop());
            }  
            return item.ToString();
        }
    }
}

```





