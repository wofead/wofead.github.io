# 合并K个升序链表

> 给你一个链表数组，每个链表都已经按升序排列。
>
> 请你将所有链表合并到一个升序链表中，返回合并后的链表。
>
> - `k == lists.length`
> - `0 <= k <= 10^4`
> - `0 <= lists[i].length <= 500`
> - `-10^4 <= lists[i][j] <= 10^4`
> - `lists[i]` 按 **升序** 排列
> - `lists[i].length` 的总和不超过 `10^4`
>

## 结题思路

1. 遍历所有列表的第一个节点，不断更新
2. 直至遍历完所有的节点

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CodeExercise
{
    public class MergeLists
    {
        public static ListNode Solve(ListNode[] lists)
        {
            int len = lists.Length;
            if (len == 0)
            {
                return null;
            }
            ListNode guard = new ListNode(0);
            ListNode curNode = guard;
            List<ListNode> nodeList = new List<ListNode>();
            for (int i = 0; i < len; i++)
            {
                nodeList.Add(lists[i]);
            }
            bool flag = true;
            while (flag)
            {
                ListNode minNode = null;
                int listIndex = -1;
                flag = false;
                for (int i = 0; i < len; i++)
                {
                    if (nodeList[i] != null)
                    {
                        flag = true;
                        if (minNode == null)
                        {
                            minNode = nodeList[i];
                            listIndex = i;
                        }
                        else if (nodeList[i].val < minNode.val)
                        {
                            minNode = nodeList[i];
                            listIndex = i;
                        }
                    }
                }
                curNode.next = minNode;
                curNode = minNode;
                if (listIndex >= 0)
                {
                    nodeList[listIndex] = minNode.next;
                }
            }
            return guard.next;
        }
    }
}

```
