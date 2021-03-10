# 删除排序链表中的重复元素 II

> 给定一个排序链表，删除所有含有重复数字的节点，只保留原始链表中 *没有重复出现* 的数字。
>

## 结题思路

使用快慢指针，和前置指针来更新链表的节点，再通过标志位来判断，这个慢指针指向的节点是否可以添加到结果链表中。



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
            ListNode guard = new ListNode(0, null);
            ListNode preNode = guard;
            ListNode slow = head;
            if (head == null || head.next == null)
            {
                return head;
            }
            ListNode quick = head.next;
            bool hasSamed = false;
            while (quick != null)
            {
                if (quick.val == slow.val)
                {
                    hasSamed = true;
                }
                else
                {
                    if (!hasSamed)
                    {
                        preNode.next = slow;
                        preNode = slow;
                    }
                    else
                    {
                        hasSamed = false;
                    }
                }
                slow = quick;
                quick = quick.next;
            }
            if (!hasSamed)
            {
                preNode.next = slow;
                preNode = slow;
            }
            preNode.next = null;
            return guard.next;
        }
    }
}

```

## 

