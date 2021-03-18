# 两两交换链表中的节点

> 给定一个链表，两两交换其中相邻的节点，并返回交换后的链表。
>
> **你不能只是单纯的改变节点内部的值**，而是需要实际的进行节点交换。
>
> - `链表中节点的数目在范围 [0, 100] 内`
> - `0 <= Node.val <= 100`
> 

## 结题思路

1. 接着使用哨兵
2. 两个指针指向需要交换的两个节点
3. 交换和更新

## 解：

```c#
namespace CodeExercise
{
    public class ListSwapPairs
    {
        public static ListNode Solve(ListNode head)
        {
            if (head == null)
            {
                return null;
            }
            ListNode guard = new ListNode(0);
            ListNode curNode = guard;
            ListNode fNode = head;
            ListNode bNode = head.next;
            while (fNode != null)
            {
                if (bNode != null)
                {
                    curNode.next = bNode;
                    curNode = bNode;
                    bNode = bNode.next;
                    curNode.next = fNode;
                    curNode = fNode;
                    fNode = bNode;
                    if (bNode != null)
                    {
                        bNode = bNode.next;
                    }
                }
                else
                {
                    curNode.next = fNode;
                    curNode = fNode;
                    fNode = bNode;
                }
            }
            curNode.next = null;
            return guard.next;
        }
    }
}

```
