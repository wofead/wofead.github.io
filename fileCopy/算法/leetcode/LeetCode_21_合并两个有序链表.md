# 合并两个有序链表

> 将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 
>
> - 两个链表的节点数目范围是 `[0, 50]`
>- `-100 <= Node.val <= 100`
> - `l1` 和 `l2` 均按 **非递减顺序** 排列
> 

## 结题思路

1. 判断大小就ok了

## 解：

```c#
namespace CodeExercise
{
    public class MergeTwoListsLink
    {
        public static ListNode Solve(ListNode l1, ListNode l2)
        {
            ListNode guard = new ListNode(0, null);
            ListNode index = guard;
            while (l1 != null || l2 != null)
            {
                if (l1 == null)
                {
                    index.next = l2;
                    break;
                }
                else if (l2 == null)
                {
                    index.next = l1;
                    break;
                }
                else if (l1.val <= l2.val)
                {
                    index.next = l1;
                    l1 = l1.next;
                }
                else
                {
                    index.next = l2;
                    l2 = l2.next;
                }
                index = index.next;
            }
            return guard.next;
        }
    }
}

```
