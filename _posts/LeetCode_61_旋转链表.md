# 旋转链表

> 给定一个链表，旋转链表，将链表每个节点向右移动 *k* 个位置，其中 *k* 是非负数。

## 结题思路1

1. 算出链表的长度
2. 找到index，即从哪个位置开始，后面的元素要放到前面
3. 开始从前面插入

## 结题思路2

1. 将链表成环
2. 找到index，断环，返回新的head


## 解：

```c#
using System;
using System.Collections.Generic;
using System.Text;

namespace Arithmatic
{
    public class LickListRotateRight
    {
        public ListNode Solve(ListNode head, int k)
        {
            ListNode guard = new ListNode(0, head);
            int length = 0;
            while (head != null)
            {
                length++;
                head = head.next;
            }
            if (length == 0 || length == 1)
            {
                return guard.next;
            }
            int index = k % length;
            head = guard.next;
            ListNode tail = head;
            for (int i = 0; i < length - index; i++)
            {
                tail = head;
                head = head.next;
            }
            tail.next = null;
            ListNode lastNode = guard;
            while (head != null)
            {
                ListNode temp = head.next;
                head.next = lastNode.next;
                lastNode.next = head;
                lastNode = head;
                head = temp;
            }
            return guard.next;
        }

        public ListNode OfficialSolve(ListNode head, int k)
        {
            // base cases
            if (head == null) return null;
            if (head.next == null) return head;

            // close the linked list into the ring
            ListNode old_tail = head;
            int n;
            for (n = 1; old_tail.next != null; n++)
                old_tail = old_tail.next;
            old_tail.next = head;

            // find new tail : (n - k % n - 1)th node
            // and new head : (n - k % n)th node
            ListNode new_tail = head;
            for (int i = 0; i < n - k % n - 1; i++)
                new_tail = new_tail.next;
            ListNode new_head = new_tail.next;

            // break the ring
            new_tail.next = null;

            return new_head;
        }
    }

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
}

```



