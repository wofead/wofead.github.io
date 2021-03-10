# 删除排序链表中的重复元素 II

> 给定一个排序链表，删除所有含有重复数字的节点，只保留原始链表中 *没有重复出现* 的数字。
>

## 结题思路

使用快慢指针，判断是否需要删除快指针指向的那个节点。

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class DeleteDuplicates
    {
        public ListNode Solve(ListNode head)
        {
            if (head == null || head.next == null)
            {
                return head;
            }
            ListNode slow = head;
            ListNode quick = head.next;
            while (quick != null)
            {
                if (quick.val == slow.val)
                {
                    quick = quick.next;
                    slow.next = quick;
                }
                else
                {
                    slow = quick;
                    quick = quick.next;
                }
            }

            return head;
        }
    }
}

```

## 

