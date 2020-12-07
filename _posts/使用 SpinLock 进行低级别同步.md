# 使用 SpinLock 进行低级别同步.

下面的示例展示了如何使用 [SpinLock](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock)。 在此示例中，关键部分执行的工作量最少，因而非常适合执行 [SpinLock](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock)。 与标准锁相比，增加一点工作量即可提升 [SpinLock](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock) 的性能。 但是，超过某个点时 SpinLock 将比标准锁开销更大。 可以使用分析工具中的并发分析功能，查看哪种类型的锁可以在程序中提供更好的性能。 有关详细信息，请参阅[并发可视化工具](https://docs.microsoft.com/zh-cn/visualstudio/profiling/concurrency-visualizer)。

```c#
class SpinLockDemo2
{
    const int N = 100000;
    static Queue<Data> _queue = new Queue<Data>();
    static object _lock = new Object();
    static SpinLock _spinlock = new SpinLock();

    class Data
    {
        public string Name { get; set; }
        public double Number { get; set; }
    }
    static void Main(string[] args)
    {

        // First use a standard lock for comparison purposes.
        UseLock();
        _queue.Clear();
        UseSpinLock();

        Console.WriteLine("Press a key");
        Console.ReadKey();
    }

    private static void UpdateWithSpinLock(Data d, int i)
    {
        bool lockTaken = false;
        try
        {
            _spinlock.Enter(ref lockTaken);
            _queue.Enqueue( d );
        }
        finally
        {
            if (lockTaken) _spinlock.Exit(false);
        }
    }

    private static void UseSpinLock()
    {

          Stopwatch sw = Stopwatch.StartNew();

          Parallel.Invoke(
                  () => {
                      for (int i = 0; i < N; i++)
                      {
                          UpdateWithSpinLock(new Data() { Name = i.ToString(), Number = i }, i);
                      }
                  },
                  () => {
                      for (int i = 0; i < N; i++)
                      {
                          UpdateWithSpinLock(new Data() { Name = i.ToString(), Number = i }, i);
                      }
                  }
              );
          sw.Stop();
          Console.WriteLine("elapsed ms with spinlock: {0}", sw.ElapsedMilliseconds);
    }

    static void UpdateWithLock(Data d, int i)
    {
        lock (_lock)
        {
            _queue.Enqueue(d);
        }
    }

    private static void UseLock()
    {
        Stopwatch sw = Stopwatch.StartNew();

        Parallel.Invoke(
                () => {
                    for (int i = 0; i < N; i++)
                    {
                        UpdateWithLock(new Data() { Name = i.ToString(), Number = i }, i);
                    }
                },
                () => {
                    for (int i = 0; i < N; i++)
                    {
                        UpdateWithLock(new Data() { Name = i.ToString(), Number = i }, i);
                    }
                }
            );
        sw.Stop();
        Console.WriteLine("elapsed ms with lock: {0}", sw.ElapsedMilliseconds);
    }
}
```

如果共享资源上的锁不会保留太长时间，[SpinLock](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock) 可能会很有用。 在这种情况下，多核计算机上的阻止线程可高效旋转几个周期，直到锁被释放。 通过旋转，线程不会受到阻止，这是一个占用大量 CPU 资源的进程。 在某些情况下，[SpinLock](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock) 会停止旋转，以防出现逻辑处理器资源不足或超线程系统上优先级反转的情况。

此示例使用 [System.Collections.Generic.Queue](https://docs.microsoft.com/zh-cn/dotnet/api/system.collections.generic.queue-1) 类，要求必须有用户同步，才能执行多线程访问。 另一种方法是使用 [System.Collections.Concurrent.ConcurrentQueue](https://docs.microsoft.com/zh-cn/dotnet/api/system.collections.concurrent.concurrentqueue-1)，这不需要任何用户锁定。

请注意，在对 [SpinLock.Exit](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock.exit) 的调用中使用 `false`。 这可提供最佳性能。 在 IA64 架构上指定 `true` 可使用内存界定，这会刷新写入缓冲区以确保锁现在可用于其他线程进入。

