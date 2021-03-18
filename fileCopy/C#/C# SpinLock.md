# C# SpinLock

提供一个相互排斥锁基元，在该基元中，尝试获取锁的线程将在重复检查的循环中等待，直至该锁变为可用为止。

```c#
[System.Runtime.InteropServices.ComVisible(false)]
public struct SpinLock
```

继承 [Object](https://docs.microsoft.com/zh-cn/dotnet/api/system.object?view=netframework-4.8) ->[ValueType](https://docs.microsoft.com/zh-cn/dotnet/api/system.valuetype?view=netframework-4.8)->SpinLock

属性 ComVisibleAttribute



```c#
using System;
using System.Linq;
using System.Text;
using System.Threading;
using System.Threading.Tasks;

class SpinLockDemo
{

    // Demonstrates:
    //      Default SpinLock construction ()
    //      SpinLock.Enter(ref bool)
    //      SpinLock.Exit()
    static void SpinLockSample1()
    {
        SpinLock sl = new SpinLock();

        StringBuilder sb = new StringBuilder();

        // Action taken by each parallel job.
        // Append to the StringBuilder 10000 times, protecting
        // access to sb with a SpinLock.
        Action action = () =>
        {
            bool gotLock = false;
            for (int i = 0; i < 10000; i++)
            {
                gotLock = false;
                try
                {
                    sl.Enter(ref gotLock);
                    sb.Append((i % 10).ToString());
                }
                finally
                {
                    // Only give up the lock if you actually acquired it
                    if (gotLock) sl.Exit();
                }
            }
        };

        // Invoke 3 concurrent instances of the action above
        Parallel.Invoke(action, action, action);

        // Check/Show the results
        Console.WriteLine("sb.Length = {0} (should be 30000)", sb.Length);
        Console.WriteLine("number of occurrences of '5' in sb: {0} (should be 3000)",
            sb.ToString().Where(c => (c == '5')).Count());
    }

    // Demonstrates:
    //      Default SpinLock constructor (tracking thread owner)
    //      SpinLock.Enter(ref bool)
    //      SpinLock.Exit() throwing exception
    //      SpinLock.IsHeld
    //      SpinLock.IsHeldByCurrentThread
    //      SpinLock.IsThreadOwnerTrackingEnabled
    static void SpinLockSample2()
    {
        // Instantiate a SpinLock
        SpinLock sl = new SpinLock();

        // These MRESs help to sequence the two jobs below
        ManualResetEventSlim mre1 = new ManualResetEventSlim(false);
        ManualResetEventSlim mre2 = new ManualResetEventSlim(false);
        bool lockTaken = false;

        Task taskA = Task.Factory.StartNew(() =>
        {
            try
            {
                sl.Enter(ref lockTaken);
                Console.WriteLine("Task A: entered SpinLock");
                mre1.Set(); // Signal Task B to commence with its logic

                // Wait for Task B to complete its logic
                // (Normally, you would not want to perform such a potentially
                // heavyweight operation while holding a SpinLock, but we do it
                // here to more effectively show off SpinLock properties in
                // taskB.)
                mre2.Wait();
            }
            finally
            {
                if (lockTaken) sl.Exit();
            }
        });

        Task taskB = Task.Factory.StartNew(() =>
        {
            mre1.Wait(); // wait for Task A to signal me
            Console.WriteLine("Task B: sl.IsHeld = {0} (should be true)", sl.IsHeld);
            Console.WriteLine("Task B: sl.IsHeldByCurrentThread = {0} (should be false)", sl.IsHeldByCurrentThread);
            Console.WriteLine("Task B: sl.IsThreadOwnerTrackingEnabled = {0} (should be true)", sl.IsThreadOwnerTrackingEnabled);

            try
            {
                sl.Exit();
                Console.WriteLine("Task B: Released sl, should not have been able to!");
            }
            catch (Exception e)
            {
                Console.WriteLine("Task B: sl.Exit resulted in exception, as expected: {0}", e.Message);
            }

            mre2.Set(); // Signal Task A to exit the SpinLock
        });

        // Wait for task completion and clean up
        Task.WaitAll(taskA, taskB);
        mre1.Dispose();
        mre2.Dispose();
    }

    // Demonstrates:
    //      SpinLock constructor(false) -- thread ownership not tracked
    static void SpinLockSample3()
    {
        // Create SpinLock that does not track ownership/threadIDs
        SpinLock sl = new SpinLock(false);

        // Used to synchronize with the Task below
        ManualResetEventSlim mres = new ManualResetEventSlim(false);

        // We will verify that the Task below runs on a separate thread
        Console.WriteLine("main thread id = {0}", Thread.CurrentThread.ManagedThreadId);

        // Now enter the SpinLock.  Ordinarily, you would not want to spend so
        // much time holding a SpinLock, but we do it here for the purpose of 
        // demonstrating that a non-ownership-tracking SpinLock can be exited 
        // by a different thread than that which was used to enter it.
        bool lockTaken = false;
        sl.Enter(ref lockTaken);

        // Create a separate Task from which to Exit() the SpinLock
        Task worker = Task.Factory.StartNew(() =>
        {
            Console.WriteLine("worker task thread id = {0} (should be different than main thread id)",
                Thread.CurrentThread.ManagedThreadId);

            // Now exit the SpinLock
            try
            {
                sl.Exit();
                Console.WriteLine("worker task: successfully exited SpinLock, as expected");
            }
            catch (Exception e)
            {
                Console.WriteLine("worker task: unexpected failure in exiting SpinLock: {0}", e.Message);
            }

            // Notify main thread to continue
            mres.Set();
        });

        // Do this instead of worker.Wait(), because worker.Wait() could inline the worker Task,
        // causing it to be run on the same thread.  The purpose of this example is to show that
        // a different thread can exit the SpinLock created (without thread tracking) on your thread.
        mres.Wait();

        // now Wait() on worker and clean up
        worker.Wait();
        mres.Dispose();
    }
}
```

自旋锁可用于叶级锁，在这种情况 [Monitor](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.monitor?view=netframework-4.8) 下，通过使用、大小或由于垃圾回收压力而隐含的对象分配的成本非常高。 旋转锁定有助于避免阻塞;但是，如果你预计会有大量的阻塞，则可能由于旋转过多而无法使用自旋锁。 当锁的粒度较大且数量较大时，旋转可能非常有利 (例如，链接列表中的每个节点的锁) ，以及锁保持时间始终极短。 通常，在持有自旋锁时，应避免使用以下任何操作：

- 堵塞
- 调用自身可能会阻止的任何内容
- 同时保留多个自旋锁
- (接口和虚方法)进行动态调度的调用
- 对任何代码进行静态调用
- 分配内存

[SpinLock](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock?view=netframework-4.8) 仅应在确定这样做后使用才能改善应用程序的性能。 出于性能方面的考虑，还必须注意， [SpinLock](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock?view=netframework-4.8) 是值类型。 出于此原因，必须注意不要意外复制 [SpinLock](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock?view=netframework-4.8) 实例，因为两个实例 (原始实例，并且副本) 将彼此完全独立，这可能会导致应用程序出现错误的行为。 如果 [SpinLock](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock?view=netframework-4.8) 必须传递实例，则它应按引用而不是按值传递。

不要 [SpinLock](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock?view=netframework-4.8) 在只读字段中存储实例。

## 属性和方法

| [SpinLock(Boolean)](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock.-ctor?view=netframework-4.8#System_Threading_SpinLock__ctor_System_Boolean_) | 使用用于跟踪线程 ID 以改善调试的选项初始化 [SpinLock](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock?view=netframework-4.8) 结构的新实例。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [IsHeld](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock.isheld?view=netframework-4.8#System_Threading_SpinLock_IsHeld) | 获取锁当前是否已由任何线程占用。                             |
| [IsHeldByCurrentThread](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock.isheldbycurrentthread?view=netframework-4.8#System_Threading_SpinLock_IsHeldByCurrentThread) | 获取锁是否已由当前线程占用。                                 |
| [IsThreadOwnerTrackingEnabled](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock.isthreadownertrackingenabled?view=netframework-4.8#System_Threading_SpinLock_IsThreadOwnerTrackingEnabled) | 获取是否已为此实例启用了线程所有权跟踪。                     |
| [Enter(Boolean)](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock.enter?view=netframework-4.8#System_Threading_SpinLock_Enter_System_Boolean__) | 采用可靠的方式获取锁，这样，即使在方法调用中发生异常的情况下，都能采用可靠的方式检查 `lockTaken` 以确定是否已获取锁。 |
| [Exit()](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock.exit?view=netframework-4.8#System_Threading_SpinLock_Exit) | 释放锁。                                                     |
| [Exit(Boolean)](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock.exit?view=netframework-4.8#System_Threading_SpinLock_Exit_System_Boolean_) | 释放锁。                                                     |
| [TryEnter(Boolean)](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock.tryenter?view=netframework-4.8#System_Threading_SpinLock_TryEnter_System_Boolean__) | 尝试采用可靠的方式获取锁，这样，即使在方法调用中发生异常的情况下，都能采用可靠的方式检查 `lockTaken` 以确定是否已获取锁。 |
| [TryEnter(Int32, Boolean)](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock.tryenter?view=netframework-4.8#System_Threading_SpinLock_TryEnter_System_Int32_System_Boolean__) | 尝试采用可靠的方式获取锁，这样，即使在方法调用中发生异常的情况下，都能采用可靠的方式检查 `lockTaken` 以确定是否已获取锁。 |
| [TryEnter(TimeSpan, Boolean)](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock.tryenter?view=netframework-4.8#System_Threading_SpinLock_TryEnter_System_TimeSpan_System_Boolean__) | 尝试采用可靠的方式获取锁，这样，即使在方法调用中发生异常的情况下，都能采用可靠的方式检查 `lockTaken` 以确定是否已获取锁。 |

```c#
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Text;
using System.Threading;
using System.Threading.Tasks;

namespace TaskStudy
{
    static class SpinLockTest
    {
        public static void StartTest()
        {
            int count = 0;
            Task[] taskList = new Task[10];
            Stopwatch sp = new Stopwatch();
            sp.Start();
            // 不要意外复制。每个实例都是独立的。 因为SpinLock是值类型
            SpinLock _spinLock = new SpinLock();
            for (int i = 0; i < taskList.Length; i++)
            {
                taskList[i] = Task.Run(() =>
                {

                    bool _lock = false;
                    for (int j = 0; j < 100000; j++)
                    {
                        _spinLock.Enter(ref _lock);
                        count++;
                        _spinLock.Exit();
                        _lock = false;
                    }
                });
            }

            sp.Stop();
            Task.WaitAll(taskList);
            Console.WriteLine($"完成! 耗时:{sp.ElapsedTicks}");
            Console.WriteLine($"结果:{count}");
            Console.ReadKey();
        }
    }
}

```

