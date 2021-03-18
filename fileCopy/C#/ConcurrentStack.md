# C# ConcurrentStack

在C#中，Queue和Stack是依靠数组实现的，那么关于ConcurrentStack的栈又是如何实现的呢？它是如何做到线程安全的呢？

```c#
public class ConcurrentStack<T> : IProducerConsumerCollection<T>, IReadOnlyCollection<T>
{
    private class Node
    {
        internal readonly T m_value; // Value of the node.
        internal Node m_next; // Next pointer.
        internal Node(T value)
        {
            m_value = value;
            m_next = null;
        }
    }
    private volatile Node m_head; 
    private const int BACKOFF_MAX_YIELDS = 8;

    public ConcurrentStack(){}
    public ConcurrentStack(IEnumerable<T> collection)
    {
        if (collection == null)
        {
            throw new ArgumentNullException("collection");
        }
        InitializeFromCollection(collection);
    }

    private void InitializeFromCollection(IEnumerable<T> collection)
    {
        // We just copy the contents of the collection to our stack.
        Node lastNode = null;
        foreach (T element in collection)
        {
            Node newNode = new Node(element);
            newNode.m_next = lastNode;
            lastNode = newNode;
        }

        m_head = lastNode;
    }

    public void Push(T item)
    {
        Node newNode = new Node(item);
        newNode.m_next = m_head;
        if (Interlocked.CompareExchange(ref m_head, newNode, newNode.m_next) == newNode.m_next)
        {
            return;
        }
        PushCore(newNode, newNode);
    }

    private void PushCore(Node head, Node tail)
    {
        SpinWait spin = new SpinWait();
        do
        {
            spin.SpinOnce();
            // Reread the head and link our new node.
            tail.m_next = m_head;
        }
        while (Interlocked.CompareExchange(ref m_head, head, tail.m_next) != tail.m_next);

    }

    public bool TryPop(out T result)
    {
        Node head = m_head;
        //stack is empty
        if (head == null)
        {
            result = default(T);
            return false;
        }
        if (Interlocked.CompareExchange(ref m_head, head.m_next, head) == head)
        {
            result = head.m_value;
            return true;
        }
        return TryPopCore(out result);
    }

    private bool TryPopCore(out T result)
    {
        Node poppedNode;

        if (TryPopCore(1, out poppedNode) == 1)
        {
            result = poppedNode.m_value;
            return true;
        }

        result = default(T);
        return false;

    }
    private int TryPopCore(int count, out Node poppedHead)
    {
        SpinWait spin = new SpinWait();
        Node head;
        Node next;
        int backoff = 1;
        Random r = new Random(Environment.TickCount & Int32.MaxValue); // avoid the case where TickCount could return Int32.MinValue
        while (true)
        {
            head = m_head;
            // Is the stack empty?
            if (head == null)
            {
                poppedHead = null;
                return 0;
            }
            next = head;
            int nodesCount = 1;
            for (; nodesCount < count && next.m_next != null; nodesCount++)
            {
                next = next.m_next;
            }

            // Try to swap the new head.  If we succeed, break out of the loop.
            if (Interlocked.CompareExchange(ref m_head, next.m_next, head) == head)
            {
                poppedHead = head;
                return nodesCount;
            }

            // We failed to CAS the new head.  Spin briefly and retry.
            for (int i = 0; i < backoff; i++)
            {
                spin.SpinOnce();
            }

            backoff = spin.NextSpinWillYield ? r.Next(1, BACKOFF_MAX_YIELDS) : backoff * 2;
        }
    }
}
```

**ConcurrentStack\<T>里面有一个内部类Node，看到这里我们就知道ConcurrentStack\<T>的栈是一开节点Node来做的一个链表**.

那么线程安全又是怎么做到的了？

首先我们来看看Push方法，**首先我们需要新实例一个Node，并且新Node的m_next指向现有m_head头节点【newNode.m_next = m_head】，然后在原子比较newNode.m_next 是否是m_head【Interlocked.CompareExchange(ref m_head, newNode, newNode.m_next) == newNode.m_next】，如果是那么把m_head改为newNode ，push操作完成。**

如果第一个线程newNode.m_next = m_head之后，有新的线程执行了Interlocked.CompareExchange(ref m_head, newNode, newNode.m_next) == newNode.m_next 那么push就需要执行 **PushCore方法；该方法先自旋一下，然后在Interlocked.CompareExchange(ref m_head, head, tail.m_next) != tail.m_next【这里head和tail是新节点，tail.m_next是指向m_head】，如果当前线程是最新最近的那个 ，那么这个Interlocked.CompareExchange(ref m_head, head, tail.m_next) == tail.m_next就为true，退出循环，否者自旋后再次比较赋值。**

TryPop的实现也是类似的。**TryPopCore方法的核心是  if (Interlocked.CompareExchange(ref m_head, next.m_next, head) == head)，如果成立则退出，否者自旋，至于自旋的次数来源于for (int i = 0; i < backoff; i++){ spin.SpinOnce(); }。**

**线程安全依靠SpinWait 的自旋和原子操作Interlocked.CompareExchange来实现的。**

