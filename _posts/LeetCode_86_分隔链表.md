# 分割链表

> 给你一个链表的头节点 head 和一个特定值 x ，请你对链表进行分隔，使得所有 小于 x 的节点都出现在 大于或等于 x 的节点之前。
>
> 你应当 保留 两个分区中每个节点的初始相对位置。
>

## 结题思路

三个节点指针进行遍历链表，一个负责指向小的，一个负责指向大的开始，一个指向大的结尾。

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace TaskExercise
{


    public class ListNode
    {
        public int val;
        public ListNode next;
        public ListNode(int val = 0, ListNode next = null)
        {
            this.val = val;
            this.next = next;
        }
    }

    public class LinkListPartition
    {
        public ListNode Solve(ListNode head, int x)
        {
            ListNode guard = new ListNode(0, null);
            ListNode cur = head;
            ListNode small = guard;
            ListNode bigStart = null;
            ListNode bigEnd = null;
            while (cur != null)
            {
                if (cur.val >= x)
                {
                    if (bigStart != null)
                    {
                        bigEnd.next = cur;
                        bigEnd = cur;
                    }
                    else
                    {
                        bigStart = cur;
                        bigEnd = cur;
                    }
                }
                else
                {
                    small.next = cur;
                    small = cur;
                }
                cur = cur.next;
            }
            if (bigEnd != null)
            {
                bigEnd.next = null;
            }
            small.next = bigStart;
            return guard.next;
        }
    }
}

```
