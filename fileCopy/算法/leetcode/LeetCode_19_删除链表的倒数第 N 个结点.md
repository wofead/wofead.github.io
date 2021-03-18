# 删除链表的倒数第 N 个结点

> 给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。
>
> - `链表中结点的数目为 sz`
>- `1 <= sz <= 30`
> - `0 <= Node.val <= 100`
> - `1 <= n <= sz`
>

## 结题思路

1. 遍历链表，然后算出正遍历需要遍历到第几个节点，最后移除它

## 解题思路（官方）

1. 快慢指针
2. 快慢相隔n
3. 慢的到结尾
4. 快的指向需要移除节点的前面那个节点
5. 然后移除

## 解：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace CodeExercise
{
    public class RemoveNthFromEnd
    {
        public static ListNode Solve(ListNode head, int n)
        {
            ListNode f = head;
            int len = 1;
            while (f.next != null)
            {
                len++;
                f = f.next;
            }
            ListNode lastNode = null;
            ListNode curNode = head;
            if (len == 1)
            {
                return lastNode;
            }
            int index = len - n + 1;
            int curIndex = 1;
            while (curIndex != index)
            {
                lastNode = curNode;
                curNode = curNode.next;
                curIndex++;
            }
            if (lastNode == null)
            {
                return curNode.next;
            }
            else
            {
                lastNode.next = curNode.next;
                return head;
            }
        }

        public static ListNode OfficialSolve(ListNode head, int n)
        {
            ListNode guard = new ListNode(0, head);
            ListNode quick = head, slow = guard;
            int curValue = 0;
            while (quick != null)
            {
                if (n == curValue)
                {
                    slow = slow.next;
                    quick = quick.next;
                }
                else
                {
                    quick = quick.next;
                    curValue++;
                }
            }
            if (slow.next == head)
            {
                return head.next;
            }
            else
            {
                slow.next = slow.next.next;
                return head;
            }
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
