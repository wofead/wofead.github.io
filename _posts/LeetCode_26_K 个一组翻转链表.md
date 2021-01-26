# K 个一组翻转链表

> 给你一个链表，每 k 个节点一组进行翻转，请你返回翻转后的链表。
>
> k 是一个正整数，它的值小于或等于链表的长度。
>
> 如果节点总数不是 k 的整数倍，那么请将最后剩余的节点保持原有顺序。
>

## 结题思路

1. 快慢指针，快的比慢的快k
2. 获取k个node的数值
3. 更新慢指针之后的值

## 解：

```c#
namespace CodeExercise
{
    public class ListNodeReverseKGroup
    {
        public static ListNode Solve(ListNode head, int k)
        {
            ListNode guard = new ListNode(0, head);
            ListNode q = head;
            ListNode s = guard;
            int[] kValue = new int[k];
            int curIndex = 0;
            while (q != null || curIndex == k)
            {
                if (curIndex == k)
                {
                    for (int i = k - 1; i >= 0; i--)
                    {
                        s.next.val = kValue[i];
                        s = s.next;
                    }
                    curIndex = 0;
                }
                else
                {
                    kValue[curIndex] = q.val;
                    curIndex++;
                    q = q.next;
                }
            }
            return head;
        }
    }
}

```
