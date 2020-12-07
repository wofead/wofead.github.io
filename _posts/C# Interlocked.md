# C# Interlocked

无锁代码下，在读写字段时使用内存屏障往往是不够的。在64位字段上进行加、减操作需要使用Interlocked工具类这样更加重型的方式。Interlocked也提供了Exchange和CompareExchange方法，后者能够进行无锁的读改写(read-modify-write)操作，只需要额外增加一点代码。

如果一条语句在底层处理器上被当做一个独立不可分割的指令，那么它本质上是原子的(aomic)。严格的原子性可以阻止任何抢占的可能。对于32位（或更低）的字段简单读写总是原子的。而操作64位字段仅在64位运行时环境下是原子的，并且结合了多个读写操作的语句必然不是原子的：

```c#
class Atomicity
{
    static int _x, _y;
    static long _z;
    static void Test()
    {
        long myLocal;
        _x = 3; // 原子的
        _z = 3;// 32位环境下不是原子的（_z 是64位的）
        myLocal = _z;// 32位环境下不是原子的（_z 是64位的）
        _y += _x;//不是原子的 (结合了读和写操作)
        _x++; //不是原子的(结合了读写操作)
    }
}
```

在32位环境下读写64位字段不是原子的，因为它需要两条独立的指令：每条用于对应的32位内存地址。所以，如果线程X在读一个64位的值，同时线程Y更新它，那么线程X最终可能的到新旧两个值按位组合后的结果(一个撕裂读(Torn read))。

编译器实现x++这种一元运算，是通过先读一个变量，然后计算，最后写回去的方式。考虑如下类：

```c#
class ThreadUnsafe
{
	static int _x = 1000;
    static void Go() {
        for(int i=0; i < 100;i++)
            _x--;
    }
}
```

抛开内存屏障的事情，你可能会认为如果 10 个线程并发运行`Go`，最终`_x`会为`0`。然而，这并不一定，因为可能存在竞态条件（race condition），在一个线程完成读取`x`的当前值，减少值，把值写回这个过程之间，被另一个线程抢占（导致一个过期的值被写回）。

当然，可以通过用`lock`语句封装非原子的操作来解决这些问题。实际上，锁如果一致的使用，可以模拟原子性。然而，`Interlocked`类为这样简单的操作提供了一个更方便更快的方案：

```c#
class Program
{
  static long _sum;

  static void Main()
  {                                                             // _sum
    // 简单的自增/自减操作:
    Interlocked.Increment (ref _sum);                              // 1
    Interlocked.Decrement (ref _sum);                              // 0

    // 加/减一个值:
    Interlocked.Add (ref _sum, 3);                                 // 3

    // 读取64位字段:
    Console.WriteLine (Interlocked.Read (ref _sum));               // 3

    // 读取当前值并且写64位字段
    // (打印 "3"，并且将 _sum 更新为 10 )
    Console.WriteLine (Interlocked.Exchange (ref _sum, 10));       // 10

    // 仅当字段的当前值匹配特定的值（10）时才更新它：
    Console.WriteLine (Interlocked.CompareExchange (ref _sum,
                                                    123, 10);      // 123
  }
}
```

`Interlocked`上的所有方法都使用全栅栏。因此，通过`Interlocked`访问字段不需要额外的栅栏，除非它们在程序其它地方没有通过`Interlocked`或`lock`来访问。

`Interlocked`的数学运算操作仅限于`Increment`、`Decrement`以及`Add`。如果你希望进行乘法或其它计算，在无锁方式下可以使用`CompareExchange`方法（通常与自旋等待一起使用）。

`Interlocked`类通过将原子性的需求传达给操作系统和虚拟机来进行实现其功能。

`Interlocked`类的方法通常产生 10ns 的开销，是无竞争锁的一半。此外，因为它们不会导致阻塞，所以不会带来上下文切换的开销。然而，如果在循环中多次迭代使用`Interlocked`，就可能比在循环外使用一个锁的效率低（不过`Interlocked`可以实现更高的并发度）。