# 	Unity Task 多线程

现在Unity开发一般都使用的是C#，所以Unity中的Task和C#中的Task一样么？

答案是不一样！！！！！！

[toc]

## Unity中的Task和C#中的Task区别

1. 它们属于不同的程序集：Unity属于netstandard.dll程序集，而c# task属于mscorlib.dll程序集。
2. Unity主线程调用async方法时，不论是await之前还是之后代码都在主线程中运行。c#主线程调用async方法时，await之前由主线程运行，await之后由子线程运行。Unity这样做的好处是，不用考虑线程安全问题，可以替代协程。但相比协程更加灵活，不需要MonoBehavior来开启（CustomYieldInstruction也可以），可以进行错误捕捉。

## Unity Task使用要注意的地方

### 在主线程中使用Task.wait方法或者Task.Result会导致死锁。

Task的副作用中，影响大的一个是deadlock了。通常的Task.Result、Task.Wait、Task.WaitAny、Task.WaitAll等都会阻塞当前线程，当配合上await后，则很容易造成context争用死锁。

```c#
using System.Threading.Tasks;
using UnityEngine;

public class TaskTest : MonoBehaviour
{
    // My "top-level" method.
    void Start()
    {
        Debug.Log("ThreadId 11111 : " + Thread.CurrentThread.ManagedThreadId);
        //var task = Task.Run<int>(()=> Task1());
        var task = Task1();
        Debug.Log("ThreadId 22222 : " + Thread.CurrentThread.ManagedThreadId);
        Debug.Log("Result: " +task.Result);
        Debug.Log("ThreadId 33333 : " + Thread.CurrentThread.ManagedThreadId);
    }

    // My Task method.
    async Task<int> Task1()
    {
        Debug.Log("ThreadId 44444 : " + Thread.CurrentThread.ManagedThreadId);
        // block.
        await Task.Delay(1);
        //await Task.Delay(1000).ConfigureAwait(false);
        Debug.Log("ThreadId 55555 : " + Thread.CurrentThread.ManagedThreadId);
        return 1;
    }
}
//ThreadId 11111 : 1
//ThreadId 22222 : 1
//ThreadId 44444 : 32
//ThreadId 55555 : 37
//ThreadId 33333 : 1
```

**造成死锁的原因**

1. 顶级方法调用Task1方法(处于start上下文中)；
2. Task1通过调用Task.Delay(1)；(仍在上下文中)来启东倒计时。
3. Task1返回一个未完成的任务，暗示倒计时还没有完成。
4. Task1等待Task1完成并返回结果，上下文被捕获，以后用于继续运行Task1方法，Task1返回一个未完成的Task，表明Task1方法未完成。
5. 顶级方法同步阻止Task1返回。这将阻塞上下文线程。
6. ...最终,倒计时完成，完成了Task1返回的任务。
7. Task1的延续现在可以运行了，他等待上下文可用，以便可以在上下文中执行。
8. 死锁。顶级方法正在阻止上下文线程，等待Task1完成，而Task1正在等待山下文空闲以便可以完成。

从线程ID的角度来分析，如果不开辟新的线程(不使用Task.Run\<int>())，默认第一个Task使用的是主线程，即线程Id为1的线程，然后`Task.Delay(100);`作为Task1的返回值，返回给Start上限文，即主线程内，但是由于倒计时并没有完成，这个时候调用task.Result这个函数，发现Task1没有完成，锁定上下文，等待Task1完成。过一段时间，倒计时这个任务完成了，完成了Task1返回的任务，他等待Start上下文可用，但是上下文已经被顶级函数锁定，已经用不了了，导致死锁。

`task.Result`把Unity的主线程block住了，等待`await Task.Delay(1)`异步操作完成，但是`await`执行完成之后，准备在captured context上执行"剩下"的代码逻辑（`return 1`），由于captured context依然由blocked的unity主线程hold住，所以此处会等待Unity主线程释放掉对context的独占。相互等待，产生死锁。

其实还是存在解决方法的，在异步操作执行完毕之后，不在寻求在Unity主线程所持用的context上继续执行剩下的代码，就不存在这个问题了，所以可以使用`var task = Task.Run<int>(()=> Task1());`的方式，在新的线程中执行Task，而不在主线程中执行Task。

还有一种方式就是不回到主线程中，`await Task.Delay(1000).ConfigureAwait(false);`。

上面两种方式`ConfigureAwait(false)`，这种方式是通过避免回到Start上下文的方式来避免死锁，而`Task.run()`,是让Task1运行的时候不处于主线程，两种方式都可以避免死锁。

### Task和struct搭配由于struct值拷贝会出现问题

```c#
public struct Test
{
    public async Task Inc()
    {
        ++iRef;
        Console.WriteLine(iRef);
        await Task.Yield();
    }
    
    public void Dec()
    {
        Console.WriteLine(iRef);
    }
    
    int iRef;
}

Test asset = new Tesk();
await asset.Inc();
asset.Dec();
//（输出结果依次为1和0）
```

Task方法出人意料之外细想后又在情理之中地copy了个新的struct对象实例来操作，本机制会引发的问题也是隐蔽。当然对与要求struct只能用来存数据的团队，应该不会踩到。

Task虽说比Coroutine对错误传递相对友好，但是也不是完美的错误传递，稍有不慎也会出现吞异常的情况，让错误定位增添一丝难度。例子如下：

```c#
static async Task Start()
{
	TestError(true);
}

static async Task TestError(bool state)
{
    await Task.Delay(1);
    if(state)
    {
        throw new Exception("aaa");
    }
}
```



### 避免使用 Async Void

异步方法有三种可能的返回类型:Task、Task\<T>和void，但是异步方法的自然返回类型只是Task和Task\<T>。当从同步代码转换为异步代码时，返回类型T的任何方法都将成为返回Task\<T>的异步方法，而返回void的任何方法都将成为返回Task的异步方法。下面的代码片段说明了一个同步的空返回方法及其等效的异步方法

```c#
void MyMethod()
{
  // Do synchronous work.
  Thread.Sleep(1000);
}
async Task MyMethodAsync()
{
  // Do asynchronous work.
  await Task.Delay(1000);
}
```

空返回异步方法有一个特定的目的:使异步事件处理程序成为可能。可以有一个返回实际类型的事件处理程序，但这在语言中不能很好地工作;调用返回类型的事件处理程序是非常笨拙的，而事件处理程序实际上返回某些东西的概念没有多大意义。事件处理程序自然会返回void，因此异步方法返回void，这样您就可以拥有一个异步事件处理程序。然而，异步void方法的一些语义与异步任务或asyn的语义略有不同.

异步void方法有不同的错误处理语义。当从Task或Task\<T>方法中抛出异常时，该异常将被捕获并放置在任务对象上。使用async void方法时，没有Task对象，因此从async void方法中抛出的任何异常都将直接在async void方法启动时处于活动状态的SynchronizationContext上引发。

```c#
private async void ThrowExceptionAsync()
{
  throw new InvalidOperationException();
}
public void AsyncVoidExceptions_CannotBeCaughtByCatch()
{
  try
  {
    ThrowExceptionAsync();
  }
  catch (Exception)
  {
    // The exception is never caught here!
    throw;
  }
}
```

可以使用AppDomain.UnhandledException或类似的针对GUI / ASP.NET应用程序的全部事件来观察这些异常，但是使用这些事件进行常规异常处理是无法维护的。

异步void方法具有不同的构成语义。可以使用await，Task.WhenAny，Task.WhenAll等轻松地组成返回Task或Task \<T>的异步方法。返回void的异步方法不能提供一种简单的方法来通知调用代码它们已完成。启动几个异步void方法很容易，但是确定它们何时完成并不容易。异步void方法在启动和结束时会通知其SynchronizationContext，但是自定义SynchronizationContext是常规应用程序代码的复杂解决方案。

异步无效方法很难测试。由于错误处理和编写方面的差异，编写调用异步void方法的单元测试很困难。MSTest异步测试支持仅适用于返回Task或Task <T>的异步方法。可以安装一个SynchronizationContext来检测所有异步void方法何时完成并收集任何异常，但是使异步void方法代替返回Task则容易得多。

显然，与异步Task方法相比，异步void方法有几个缺点，但是它们在一种特殊情况下非常有用：异步事件处理程序。语义上的差异对于异步事件处理程序是有意义的。它们直接在SynchronizationContext上引发异常，这与同步事件处理程序的行为类似。同步事件处理程序通常是私有的，因此无法组成或直接对其进行测试。我想采用的一种方法是最小化异步事件处理程序中的代码，例如，让它等待包含实际逻辑的异步Task方法。下面的代码说明了这种方法，将异步void方法用于事件处理程序，而不牺牲可测试性：

```c#
private async void button1_Click(object sender, EventArgs e)
{
  await Button1ClickAsync();
}
public async Task Button1ClickAsync()
{
  // Do asynchronous work.
  await Task.Delay(1000);
}
```

如果调用方不希望异步方法无效，则异步方法可能造成严重破坏。当返回类型为Task时，调用者知道它正在处理将来的操作；当返回类型为void时，调用者可能会认为方法在返回时已完成。这个问题可能以许多意想不到的方式出现。在接口（或基类）上提供void返回方法的异步实现（或重写）通常是错误的。一些事件还假定它们的处理程序返回时已完成。一个微妙的陷阱是将异步lambda传递给带有Action参数的方法。在这种情况下，异步lambda返回void并继承异步void方法的所有问题。通常，仅将异步lambda转换为返回Task的委托类型（例如Func \<Task>），才应使用它们。

总结第一条指南，您应该首选异步任务，而不是异步void。异步任务方法可简化错误处理，可组合性和可测试性。该准则的例外是异步事件处理程序，该事件处理程序必须返回void。此异常包括从逻辑上讲是事件处理程序的方法，即使它们实际上不是真正的事件处理程序（例如ICommand.Execute实现）。

“一路异步”意味着您应该在不仔细考虑后果的情况下不要混合使用同步和异步代码。特别是，通过调用Task.Wait或Task.Result阻止异步代码通常是一个坏主意。对于程序员来说，这是一个特别普遍的问题，他们将自己的脚步“浸入”异步编程中，只转换其应用程序的一小部分并将其包装在同步API中，从而使应用程序的其余部分与更改隔离。不幸的是，它们遇到了死锁问题。

```c#
public static class DeadlockDemo
{
  private static async Task DelayAsync()
  {
    await Task.Delay(1000);
  }
  // This method causes a deadlock when called in a GUI or ASP.NET context.
  public static void Test()
  {
    // Start the delay.
    var delayTask = DelayAsync();
    // Wait for the delay to complete.
    delayTask.Wait();
  }
}
```

该死锁的根本原因是等待方式处理上下文的方式。默认情况下，当等待不完整的任务时，将捕获当前的“上下文”，并在任务完成时将其用于恢复该方法。除非为空，否则此“上下文”是当前的SynchronizationContext，在这种情况下，它是当前的TaskScheduler。GUI和ASP.NET应用程序具有SynchronizationContext，该上下文一次只允许运行一块代码。等待完成后，它将尝试在捕获的上下文中执行异步方法的其余部分。但是该上下文中已经有一个线程，该线程（同步）在等待异步方法完成。他们每个人都在等待对方，从而导致僵局。

请注意，控制台应用程序不会导致此死锁。它们具有线程池SynchronizationContext，而不是一次一次的SynchronizationContext，因此，在等待完成时，它将在线程池线程上调度异步方法的其余部分。该方法能够完成，从而完成其返回的任务，并且没有死锁。当程序员编写测试控制台程序，观察部分异步的代码是否按预期工作，然后将相同的代码移入GUI或ASP.NET应用程序时，这种行为上的差异可能会令人困惑，从而导致死锁。

解决此问题的最佳方法是允许异步代码通过代码库自然增长。如果您遵循此解决方案，则会看到异步代码扩展到其入口点，通常是事件处理程序或控制器操作。控制台应用程序不能完全遵循此解决方案，因为Main方法不能是异步的。如果Main方法是异步的，则它可能在完成之前返回，从而导致程序结束。

### 配置上下文

```c#
async Task MyMethodAsync()
{
  // Code here runs in the original context.
  await Task.Delay(1000);
  // Code here runs in the original context.
  await Task.Delay(1000).ConfigureAwait(
    continueOnCapturedContext: false);
  // Code here runs without the original
  // context (in this case, on the thread pool).
}
```

通过使用ConfigureAwait，您可以启用少量并行处理：某些异步代码可以与GUI线程并行运行，而不必不断地给它添加工作标记。

除了性能以外，ConfigureAwait还有另一个重要方面：它可以避免死锁。

如果将“ ConfigureAwait（false）”添加到DelayAsync中的代码行，则可以避免死锁。这次，当等待完成时，它将尝试在线程池上下文中执行异步方法的其余部分。该方法能够完成，从而完成其返回的任务，并且没有死锁。如果您需要逐步将应用程序从同步转换为异步，则此技术特别有用。

如果您可以在某个方法中的某个时候使用ConfigureAwait，那么我建议您在该方法之后对该方法中的每个等待使用它。回想一下，只有在等待不完整的Task时才捕获上下文；如果任务已经完成，则不会捕获上下文。在不同的硬件和网络情况下，某些任务的完成速度可能比预期的要快，因此，您需要在返回之前等待完成的任务，亲切地处理它。

```c#
async Task MyMethodAsync()
{
  // Code here runs in the original context.
  await Task.FromResult(1);
  // Code here runs in the original context.
  await Task.FromResult(1).ConfigureAwait(continueOnCapturedContext: false);
  // Code here runs in the original context.
  var random = new Random();
  int delay = random.Next(2); // Delay is either 0 or 1
  await Task.Delay(delay).ConfigureAwait(continueOnCapturedContext: false);
  // Code here might or might not run in the original context.
  // The same is true when you await any Task
  // that might complete very quickly.
}
```

如果在需要上下文的方法中等待之后有代码，则不应使用ConfigureAwait。对于GUI应用程序，这包括操纵GUI元素，编写数据绑定属性或依赖于特定于GUI的类型（例如Dispatcher / CoreDispatcher）的任何代码。对于ASP.NET应用程序，这包括使用HttpContext.Current或构建ASP.NET响应的任何代码，包括控制器操作中的return语句。

```c#
private async void button1_Click(object sender, EventArgs e)
{
  button1.Enabled = false;
  try
  {
    // Can't use ConfigureAwait here ...
    await Task.Delay(1000);
  }
  finally
  {
    // Because we need the context here.
    button1.Enabled = true;
  }
}
```

每个异步方法都有其自己的上下文，因此，如果一个异步方法调用另一个异步方法，则它们的上下文是独立的。（Unity-->Task.Run）

```c#
private async Task HandleClickAsync()
{
  // Can use ConfigureAwait here.
  await Task.Delay(1000).ConfigureAwait(continueOnCapturedContext: false);
}
private async void button1_Click(object sender, EventArgs e)
{
  button1.Enabled = false;
  try
  {
    // Can't use ConfigureAwait here.
    await HandleClickAsync();
  }
  finally
  {
    // We are back on the original context for this method.
    button1.Enabled = true;
  }
}
```

## Task详解

ThreadPool相比Thread来说具备了很多优势，但是ThreadPool却又存在一些使用上的不方便。比如：

- ThreadPool不支持线程的取消、完成、失败通知等交互性操作；
- ThreadPool不支持线程执行的先后次序；

以往，如果开发者要实现上述功能，需要完成很多额外的工作，现在，FCL中提供了一个功能更强大的概念：Task。Task在线程池的基础上进行了优化，并提供了更多的API。在FCL4.0中，如果我们要编写多线程程序，Task显然已经优于传统的方式。

```c#
using System;
using System.Diagnostics;
using System.Threading;
using System.Threading.Tasks;

namespace TaskStudy
{
    static class TaskTest
    {
        static Stopwatch stopwatch;
        public static void StartTest()
        {
            stopwatch = new Stopwatch();
            stopwatch.Start();
            Print("程序开始");
            Task t = new Task(() =>
            {
                Print("任务开始工作");
                //模拟工作过程
                Thread.Sleep(2000);
            });
            t.Start();
            t.ContinueWith((task) =>
            {
                Print("任务完成");
                Console.WriteLine("IsCanceled={0}\tIsCompleted={1}\tIsFaulted={2}, Time is {3}.", task.IsCanceled, task.IsCompleted, task.IsFaulted, stopwatch.ElapsedMilliseconds);
            });
            //
            Print("任务状态" + t.Status);
            Console.ReadKey();
        }

        static void Print(string str)
        {
            Console.WriteLine("{0}，Time is:{1}, Current ThreadIs is {2}.", str, stopwatch.ElapsedMilliseconds, Thread.CurrentThread.ManagedThreadId);
        }

    }
}

static void Main(string[] args)
{
    TaskTest.StartTest();
}

//程序开始，Time is:0, Current ThreadIs is 1.
//任务状态WaitingToRun，Time is:17, Current ThreadIs is 1.
//任务开始工作，Time is:22, Current ThreadIs is 3.
//任务完成，Time is:2035, Current ThreadIs is 4.
//IsCanceled=False        IsCompleted=True        IsFaulted=False, Time is 2036.
```

从输出我们可以看到同步与异步的区分，任务是从第23毫秒开始的，ThreadId是3，任务完成是滴2035毫秒，ThreadId是4，这里为什么是4呢？原因是因为ContinueWith开启了另一个Task，你现在身处于等待模拟工作过程完成的那个Task里面，所以这里存在两个ThreadId。

还有我们可以发现主线程其实已经执行到ReadKey了，只不过异步的线程还在执行。

### Task用法

**无返回值的方式**

```c#
var t1 = new Task(() => TaskMethod("Task 1"));
t1.Start();
Task.WaitAll(t1);//等待所有任务结束 
//注:任务的状态:
//Start之前为:Created
//Start之后为:WaitingToRun 
```



```c#
Task.Run(() => TaskMethod("Task 2"));
```



```c#
Task.Factory.StartNew(() => TaskMethod("Task 3")); 直接异步的方法 
//或者
var t3=Task.Factory.StartNew(() => TaskMethod("Task 3"));
Task.WaitAll(t3);//等待所有任务结束
//任务的状态:
//Start之前为:Running
//Start之后为:Running
```

```c#
using System;
using System.Diagnostics;
using System.Threading;
using System.Threading.Tasks;

namespace TaskStudy
{
    static class WithoutReturn
    {
        static Stopwatch stopwatch;
        public static void StartTest()
        {
            stopwatch = new Stopwatch();
            stopwatch.Start();
            TaskMethod("程序开始");
            var t1 = new Task(() => TaskMethod("Task 1"));
            var t2 = new Task(() => TaskMethod("Task 2"));
            t2.Start();
            t1.Start();
            Task.WaitAll(t1, t2);
            TaskMethod("程序执行1111");
            Task.Run(() => TaskMethod("Task 3"));
            Task.Factory.StartNew(() => TaskMethod("Task 4"));
            //标记为长时间运行任务,则任务不会使用线程池,而在单独的线程中运行。
            Task.Factory.StartNew(() => TaskMethod("Task 5"),
                TaskCreationOptions.LongRunning);

            #region 常规的使用方式
            TaskMethod("主线程执行业务开始");
            //创建任务
            Task task = new Task(() =>
            {
                Console.WriteLine("使用System.Threading.Tasks.Task执行异步操作.");
                for (int i = 0; i < 10; i++)
                {
                    Console.WriteLine(i);
                }
            });
            //启动任务,并安排到当前任务队列线程中执行任务(System.Threading.Tasks.TaskScheduler)
            task.Start();
            Console.WriteLine("主线程执行其他处理");
            task.Wait();
            #endregion
            Thread.Sleep(TimeSpan.FromSeconds(1));
            Console.ReadKey();
        }

        static void TaskMethod(string str)
        {
            Thread.Sleep(500);
            Console.WriteLine("Task name is {0}，Time is:{1}, Current ThreadIs is {2}.", str, stopwatch.ElapsedMilliseconds, Thread.CurrentThread.ManagedThreadId);
        }
    }
}

using System;
using System.Diagnostics;
using System.Threading;
using System.Threading.Tasks;

namespace TaskStudy
{
    class Program
    {
        static void Main(string[] args)
        {
            WithoutReturn.StartTest();
        }
    }

}
//Task name is 程序开始，Time is:502, Current ThreadIs is 1.
//Task name is Task 1，Time is:1040, Current ThreadIs is 5.
//Task name is Task 2，Time is:1040, Current ThreadIs is 3.
//Task name is 程序执行1111，Time is:1541, Current ThreadIs is 1.
//Task name is 主线程执行业务开始，Time is:2052, Current ThreadIs is 1.
//主线程执行其他处理
//使用System.Threading.Tasks.Task执行异步操作.
//0
//1
//2
//Task name is Task 3，Time is:2052, Current ThreadIs is 5.
//3
//4
//5
//6
//7
//8
//9
//Task name is Task 4，Time is:2083, Current ThreadIs is 6.
//Task name is Task 5，Time is:2099, Current ThreadIs is 7.
```
根据上面的输出，我们可以发现Task.WaitAll会阻塞当前的上下文。

**async/await的实现方式:**

```c#
using System;
using System.Diagnostics;
using System.Threading;
using System.Threading.Tasks;
namespace TaskStudy
{
    static class WithOutReturnAwait
    {
        static Stopwatch stopwatch;
        public static void StartTest()
        {
            stopwatch = new Stopwatch();
            stopwatch.Start();
            Print("程序开始：");
            var t = AsyncFunction();
            //Task.WaitAll(t);
            for (int i = 0; i < 10; i++)
            {
                Console.WriteLine(string.Format("Main:i={0}", i));
            }
            Console.ReadKey();
        }

        async static Task AsyncFunction()
        {
            Print("AsyncFunction开始");
            await Task.Delay(1000);
            Print("异步操作开始");
            int result = 1;
            for (int i = 0; i < 100; i++)
            {
                result *= i;
            }
            Print("Result:" + result);
            Print("异步操作完成");
        }

        static void Print(string str)
        {
            Console.WriteLine("{0}，Time is:{1}, Current ThreadIs is {2}.", str, stopwatch.ElapsedMilliseconds, Thread.CurrentThread.ManagedThreadId);
        }
    }
}
//程序开始：，Time is:0, Current ThreadIs is 1.
//AsyncFunction开始，Time is:1, Current ThreadIs is 1.
//异步操作开始，Time is:1033, Current ThreadIs is 4.
//Result:0，Time is:1033, Current ThreadIs is 4.
//异步操作完成，Time is:1034, Current ThreadIs is 4.
//Main:i=0
//Main:i=1
//Main:i=2
//Main:i=3
//Main:i=4
//Main:i=5
//Main:i=6
//Main:i=7
//Main:i=8
//Main:i=9
```
从日志里面可以看到，`await Task.Delay(1000);`之前线程还在主线程，之后就在子线程了。还有就是当注释掉` Task.WaitAll(t);`可以发现，主线程没有被阻塞，当加上WaitAll的时候，主线程和上面的例子一样，会被阻塞。

**带返回值的方式**

```c#
Task<int> task = CreateTask("Task 1");
task.Start(); 
int result = task.Result;
```

```c#
using System;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class Program
    {
        static Task<int> CreateTask(string name)
        {
            return new Task<int>(() => TaskMethod(name));
        }

        static void Main(string[] args)
        {
            TaskMethod("Main Thread Task");
            Task<int> task = CreateTask("Task 1");
            task.Start();
            int result = task.Result;
            Console.WriteLine("Task 1 Result is: {0}", result);

            task = CreateTask("Task 2");
            //该任务会运行在主线程中
            task.RunSynchronously();
            result = task.Result;
            Console.WriteLine("Task 2 Result is: {0}", result);

            task = CreateTask("Task 3");
            Console.WriteLine(task.Status);
            task.Start();

            while (!task.IsCompleted)
            {
                Console.WriteLine(task.Status);
                Thread.Sleep(TimeSpan.FromSeconds(0.5));
            }

            Console.WriteLine(task.Status);
            result = task.Result;
            Console.WriteLine("Task 3 Result is: {0}", result);

            #region 常规使用方式
            //创建任务
            Task<int> getsumtask = new Task<int>(() => Getsum());
            //启动任务,并安排到当前任务队列线程中执行任务(System.Threading.Tasks.TaskScheduler)
            getsumtask.Start();
            Console.WriteLine("主线程执行其他处理");
            //等待任务的完成执行过程。
            getsumtask.Wait();
            //获得任务的执行结果
            Console.WriteLine("任务执行结果：{0}", getsumtask.Result.ToString());
            #endregion
        }

        static int TaskMethod(string name)
        {
            Console.WriteLine("Task {0} is running on a thread id {1}. Is thread pool thread: {2}",
                name, Thread.CurrentThread.ManagedThreadId, Thread.CurrentThread.IsThreadPoolThread);
            Thread.Sleep(TimeSpan.FromSeconds(2));
            return 42;
        }

        static int Getsum()
        {
            int sum = 0;
            Console.WriteLine("使用Task执行异步操作.");
            for (int i = 0; i < 100; i++)
            {
                sum += i;
            }
            return sum;
        }
    }
}

using System;
using System.Diagnostics;
using System.Threading;
using System.Threading.Tasks;
namespace TaskStudy
{
    static class WithOutReturnAwait
    {
        static Stopwatch stopwatch;
        public static void StartTest()
        {
            stopwatch = new Stopwatch();
            stopwatch.Start();
            Print("程序开始：");
            var t = AsyncFunction();
            //t.Wait();
            Print(" Main Result:" + t.Result);
            //Task.WaitAll(t);
            for (int i = 0; i < 10; i++)
            {
                Console.WriteLine(string.Format("Main:i={0}", i));
            }
            Console.ReadKey();
        }

        async static Task<int> AsyncFunction()
        {
            Print("AsyncFunction开始");
            await Task.Delay(1000);
            Print("异步操作开始");
            int result = 1;
            for (int i = 0; i < 100; i++)
            {
                result *= i;
            }
            Print("Result:" + result);
            Print("异步操作完成");
            return result;
        }

        static void Print(string str)
        {
            Console.WriteLine("{0}，Time is:{1}, Current ThreadIs is {2}.", str, stopwatch.ElapsedMilliseconds, Thread.CurrentThread.ManagedThreadId);
        }
    }
}

```
带返回值的Task，使用Task的泛型类型，泛型就是你的返回值类型，带泛型的就是带返回值的。

在这里我们可以发现，t.Result是会锁定上下文的，和WaitAll作用一致，t.Wait也具有这个功能。

**async/await的实现:**

```c
using System;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class Program
    {
        public static void Main()
        {
            var ret1 = AsyncGetsum();
            Console.WriteLine("主线程执行其他处理");
            for (int i = 1; i <= 3; i++)
                Console.WriteLine("Call Main()");
            int result = ret1.Result;                  //阻塞主线程
            Console.WriteLine("任务执行结果：{0}", result);
        }

        async static Task<int> AsyncGetsum()
        {
            await Task.Delay(1);
            int sum = 0;
            Console.WriteLine("使用Task执行异步操作.");
            for (int i = 0; i < 100; i++)
            {
                sum += i;
            }
            return sum;
        }
    }
}
```

### 组合任务

```c#
using System;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class Program
    {
        public static void Main()
        {
            //创建一个任务
            Task<int> task = new Task<int>(() =>
            {
                int sum = 0;
                Console.WriteLine("使用Task执行异步操作.");
                for (int i = 0; i < 100; i++)
                {
                    sum += i;
                }
                return sum;
            });
            //启动任务,并安排到当前任务队列线程中执行任务(System.Threading.Tasks.TaskScheduler)
            task.Start();
            Console.WriteLine("主线程执行其他处理");
            //任务完成时执行处理。
            Task cwt = task.ContinueWith(t =>
            {
                Console.WriteLine("任务完成后的执行结果：{0}", t.Result.ToString());
            });
            task.Wait();
            cwt.Wait();
        }
    }
}
```

### 任务串行以及子任务

```c#
using System;
using System.Collections.Concurrent;
using System.Diagnostics;
using System.Threading;
using System.Threading.Tasks;
namespace TaskStudy
{
    static class AcyncAndChild
    {
        static Stopwatch stopwatch;
        static ConcurrentStack<int> stack;
        public static void StartTest()
        {
            stopwatch = new Stopwatch();
            stack = new ConcurrentStack<int>();
            stopwatch.Start();
            //t1和t2串行
            var t1 = Task.Factory.StartNew(() => {
                Print("Task1");
                Thread.Sleep(100);
                stack.Push(1);
                stack.Push(2);
            });

            var t2 = t1.ContinueWith(t =>
            {
                stack.TryPop(out int result);
                Thread.Sleep(100);
                Print("Task 2 pop result:" + result);
            });
            //t2 和 t3并行
            var t3 = t1.ContinueWith(t =>
            {
                stack.TryPop(out int result);
                Thread.Sleep(50);
                Print("Task 3 pop result:" + result);
            });

            Task<string[]> parent = new Task<string[]>(state =>
            {
                Console.WriteLine(state);
                string[] result = new string[2];
                //创建并启动子任务
                new Task(() => { result[0] = "我是子任务1。"; Print("子任务1"); }, TaskCreationOptions.AttachedToParent).Start();
                new Task(() => { result[1] = "我是子任务2。"; Print("子任务2"); }, TaskCreationOptions.AttachedToParent).Start();
                return result;
            }, "我是父任务，并在我的处理过程中创建多个子任务，所有子任务完成以后我才会结束执行。");
            //任务处理完成后执行的操作
            parent.ContinueWith(t =>
            {
                Print("父任务完成！！！");
            });
            //启动父任务
            parent.Start();
            //等待任务结束 Wait只能等待父线程结束,没办法等到父线程的ContinueWith结束
            //parent.Wait();

            Print("Main");
            Console.ReadKey();
            
        }


        static void Print(string str)
        {
            Console.WriteLine("{0}，Time is:{1}, Current ThreadIs is {2}.", str, stopwatch.ElapsedMilliseconds, Thread.CurrentThread.ManagedThreadId);
        }
    }
}

```

### 



### 动态并行(TaskCreationOptions.AttachedToParent) 父任务等待所有子任务完成后 整个任务才算完成

```c#
using System;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class Node
    {
        public Node Left { get; set; }
        public Node Right { get; set; }
        public string Text { get; set; }
    }


    class Program
    {
        static Node GetNode()
        {
            Node root = new Node
            {
                Left = new Node
                {
                    Left = new Node
                    {
                        Text = "L-L"
                    },
                    Right = new Node
                    {
                        Text = "L-R"
                    },
                    Text = "L"
                },
                Right = new Node
                {
                    Left = new Node
                    {
                        Text = "R-L"
                    },
                    Right = new Node
                    {
                        Text = "R-R"
                    },
                    Text = "R"
                },
                Text = "Root"
            };
            return root;
        }

        static void Main(string[] args)
        {
            Node root = GetNode();
            DisplayTree(root);
        }

        static void DisplayTree(Node root)
        {
            var task = Task.Factory.StartNew(() => DisplayNode(root),
                                            CancellationToken.None,
                                            TaskCreationOptions.None,
                                            TaskScheduler.Default);
            task.Wait();
        }

        static void DisplayNode(Node current)
        {

            if (current.Left != null)
                Task.Factory.StartNew(() => DisplayNode(current.Left),
                                            CancellationToken.None,
                                            TaskCreationOptions.AttachedToParent,
                                            TaskScheduler.Default);
            if (current.Right != null)
                Task.Factory.StartNew(() => DisplayNode(current.Right),
                                            CancellationToken.None,
                                            TaskCreationOptions.AttachedToParent,
                                            TaskScheduler.Default);
            Console.WriteLine("当前节点的值为{0};处理的ThreadId={1}", current.Text, Thread.CurrentThread.ManagedThreadId);
        }
    }
}
```

### 取消任务 CancellationTokenSource

```c#
using System;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class Program
    {
        private static int TaskMethod(string name, int seconds, CancellationToken token)
        {
            Console.WriteLine("Task {0} is running on a thread id {1}. Is thread pool thread: {2}",
                name, Thread.CurrentThread.ManagedThreadId, Thread.CurrentThread.IsThreadPoolThread);
            for (int i = 0; i < seconds; i++)
            {
                Thread.Sleep(TimeSpan.FromSeconds(1));
                if (token.IsCancellationRequested) return -1;
            }
            return 42 * seconds;
        }

        private static void Main(string[] args)
        {
            var cts = new CancellationTokenSource();
            var longTask = new Task<int>(() => TaskMethod("Task 1", 10, cts.Token), cts.Token);
            Console.WriteLine(longTask.Status);
            cts.Cancel();
            Console.WriteLine(longTask.Status);
            Console.WriteLine("First task has been cancelled before execution");
            cts = new CancellationTokenSource();
            longTask = new Task<int>(() => TaskMethod("Task 2", 10, cts.Token), cts.Token);
            longTask.Start();
            for (int i = 0; i < 5; i++)
            {
                Thread.Sleep(TimeSpan.FromSeconds(0.5));
                Console.WriteLine(longTask.Status);
            }
            cts.Cancel();
            for (int i = 0; i < 5; i++)
            {
                Thread.Sleep(TimeSpan.FromSeconds(0.5));
                Console.WriteLine(longTask.Status);
            }

            Console.WriteLine("A task has been completed with result {0}.", longTask.Result);
        }
    }
}
```

### 处理任务中的异常

```c#
using System;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class Program
    {
        static int TaskMethod(string name, int seconds)
        {
            Console.WriteLine("Task {0} is running on a thread id {1}. Is thread pool thread: {2}",
                name, Thread.CurrentThread.ManagedThreadId, Thread.CurrentThread.IsThreadPoolThread);
            Thread.Sleep(TimeSpan.FromSeconds(seconds));
            throw new Exception("Boom!");
            return 42 * seconds;
        }

        static void Main(string[] args)
        {
            try
            {
                Task<int> task = Task.Run(() => TaskMethod("Task 2", 2));
                int result = task.GetAwaiter().GetResult();
                Console.WriteLine("Result: {0}", result);
            }
            catch (Exception ex)
            {
                Console.WriteLine("Task 2 Exception caught: {0}", ex.Message);
            }
            Console.WriteLine("----------------------------------------------");
            Console.WriteLine();
        }
    }
}
```

### 多个任务:

```c#
using System;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class Program
    {
        static int TaskMethod(string name, int seconds)
        {
            Console.WriteLine("Task {0} is running on a thread id {1}. Is thread pool thread: {2}",
                name, Thread.CurrentThread.ManagedThreadId, Thread.CurrentThread.IsThreadPoolThread);
            Thread.Sleep(TimeSpan.FromSeconds(seconds));
            throw new Exception(string.Format("Task {0} Boom!", name));
            return 42 * seconds;
        }


        public static void Main(string[] args)
        {
            try
            {
                var t1 = new Task<int>(() => TaskMethod("Task 3", 3));
                var t2 = new Task<int>(() => TaskMethod("Task 4", 2));
                var complexTask = Task.WhenAll(t1, t2);
                var exceptionHandler = complexTask.ContinueWith(t =>
                        Console.WriteLine("Result: {0}", t.Result),
                        TaskContinuationOptions.OnlyOnFaulted
                    );
                t1.Start();
                t2.Start();
                Task.WaitAll(t1, t2);
            }
            catch (AggregateException ex)
            {
                ex.Handle(exception =>
                {
                    Console.WriteLine(exception.Message);
                    return true;
                });
            }
        }
    }
}
```
 **async/await的方式:**
```c#
using System;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class Program
    {
        static async Task ThrowNotImplementedExceptionAsync()
        {
            throw new NotImplementedException();
        }

        static async Task ThrowInvalidOperationExceptionAsync()
        {
            throw new InvalidOperationException();
        }

        static async Task Normal()
        {
            await Fun();
        }

        static Task Fun()
        {
            return Task.Run(() =>
            {
                for (int i = 1; i <= 10; i++)
                {
                    Console.WriteLine("i={0}", i);
                    Thread.Sleep(200);
                }
            });
        }

        static async Task ObserveOneExceptionAsync()
        {
            var task1 = ThrowNotImplementedExceptionAsync();
            var task2 = ThrowInvalidOperationExceptionAsync();
            var task3 = Normal();


            try
            {
                //异步的方式
                Task allTasks = Task.WhenAll(task1, task2, task3);
                await allTasks;
                //同步的方式
                //Task.WaitAll(task1, task2, task3);
            }
            catch (NotImplementedException ex)
            {
                Console.WriteLine("task1 任务报错!");
            }
            catch (InvalidOperationException ex)
            {
                Console.WriteLine("task2 任务报错!");
            }
            catch (Exception ex)
            {
                Console.WriteLine("任务报错!");
            }

        }

        public static void Main()
        {
            Task task = ObserveOneExceptionAsync();
            Console.WriteLine("主线程继续运行........");
            task.Wait();
        }
    }
}
```

### Task.FromResult的应用

```c#
using System;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class Program
    {
        static IDictionary<string, string> cache = new Dictionary<string, string>()
        {
            {"0001","A"},
            {"0002","B"},
            {"0003","C"},
            {"0004","D"},
            {"0005","E"},
            {"0006","F"},
        };

        public static void Main()
        {
            Task<string> task = GetValueFromCache("0006");
            Console.WriteLine("主程序继续执行。。。。");
            string result = task.Result;
            Console.WriteLine("result={0}", result);

        }

        private static Task<string> GetValueFromCache(string key)
        {
            Console.WriteLine("GetValueFromCache开始执行。。。。");
            string result = string.Empty;
            //Task.Delay(5000);
            Thread.Sleep(5000);
            Console.WriteLine("GetValueFromCache继续执行。。。。");
            if (cache.TryGetValue(key, out result))
            {
                return Task.FromResult(result);
            }
            return Task.FromResult("");
        }

    }
}
```

### 使用IProgress实现异步编程的进程通知

IProgress<in T>只提供了一个方法void Report(T value)，通过Report方法把一个T类型的值报告给IProgress,然后IProgress<in T>的实现类Progress<in T>的构造函数接收类型为Action<T>的形参，通过这个委托让进度显示在UI界面中。

```c#
using System;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class Program
    {
        static void DoProcessing(IProgress<int> progress)
        {
            for (int i = 0; i <= 100; ++i)
            {
                Thread.Sleep(100);
                if (progress != null)
                {
                    progress.Report(i);
                }
            }
        }

        static async Task Display()
        {
            //当前线程
            var progress = new Progress<int>(percent =>
            {
                Console.Clear();
                Console.Write("{0}%", percent);
            });
            //线程池线程
            await Task.Run(() => DoProcessing(progress));
            Console.WriteLine("");
            Console.WriteLine("结束");
        }

        public static void Main()
        {
            Task task = Display();
            task.Wait();
        }
    }
}
```

### Factory.FromAsync的应用 (简APM模式(委托)转换为任务)(BeginXXX和EndXXX)

```c#
using System;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class Program
    {
        private delegate string AsynchronousTask(string threadName);

        private static string Test(string threadName)
        {
            Console.WriteLine("Starting...");
            Console.WriteLine("Is thread pool thread: {0}", Thread.CurrentThread.IsThreadPoolThread);
            Thread.Sleep(TimeSpan.FromSeconds(2));
            Thread.CurrentThread.Name = threadName;
            return string.Format("Thread name: {0}", Thread.CurrentThread.Name);
        }

        private static void Callback(IAsyncResult ar)
        {
            Console.WriteLine("Starting a callback...");
            Console.WriteLine("State passed to a callbak: {0}", ar.AsyncState);
            Console.WriteLine("Is thread pool thread: {0}", Thread.CurrentThread.IsThreadPoolThread);
            Console.WriteLine("Thread pool worker thread id: {0}", Thread.CurrentThread.ManagedThreadId);
        }

        //执行的流程是 先执行Test--->Callback--->task.ContinueWith
        static void Main(string[] args)
        {
            AsynchronousTask d = Test;
            Console.WriteLine("Option 1");
            Task<string> task = Task<string>.Factory.FromAsync(
                d.BeginInvoke("AsyncTaskThread", Callback, "a delegate asynchronous call"), d.EndInvoke);

            task.ContinueWith(t => Console.WriteLine("Callback is finished, now running a continuation! Result: {0}",
                t.Result));

            while (!task.IsCompleted)
            {
                Console.WriteLine(task.Status);
                Thread.Sleep(TimeSpan.FromSeconds(0.5));
            }
            Console.WriteLine(task.Status);

        }
    }
}
```

**不带回调方式的**

```c#
using System;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class Program
    {
        private delegate string AsynchronousTask(string threadName);

        private static string Test(string threadName)
        {
            Console.WriteLine("Starting...");
            Console.WriteLine("Is thread pool thread: {0}", Thread.CurrentThread.IsThreadPoolThread);
            Thread.Sleep(TimeSpan.FromSeconds(2));
            Thread.CurrentThread.Name = threadName;
            return string.Format("Thread name: {0}", Thread.CurrentThread.Name);
        }

        //执行的流程是 先执行Test--->task.ContinueWith
        static void Main(string[] args)
        {
            AsynchronousTask d = Test;
            Task<string> task = Task<string>.Factory.FromAsync(
                d.BeginInvoke, d.EndInvoke, "AsyncTaskThread", "a delegate asynchronous call");
            task.ContinueWith(t => Console.WriteLine("Task is completed, now running a continuation! Result: {0}",
                t.Result));
            while (!task.IsCompleted)
            {
                Console.WriteLine(task.Status);
                Thread.Sleep(TimeSpan.FromSeconds(0.5));
            }
            Console.WriteLine(task.Status);

        }
    }
}
```

```c#
//Task启动带参数和返回值的函数任务
//下面的例子test2 是个带参数和返回值的函数。

private int test2(object i)
{
    this.Invoke(new Action(() =>
    {
        pictureBox1.Visible = true;
    }));
    System.Threading.Thread.Sleep(3000);
    MessageBox.Show("hello:" + i);
    this.Invoke(new Action(() =>
    {
        pictureBox1.Visible = false;
    }));
    return 0;
}

//测试调用
private void call()
{
    //Func<string, string> funcOne = delegate(string s){ return "fff"; };
    object i = 55;
    var t = Task<int>.Factory.StartNew(new Func<object, int>(test2), i);
}

//= 下载网站源文件例子 == == == == == == == == == == == ==
//HttpClient 引用System.Net.Http
private async Task< int> test2(object i)
{
    this.Invoke(new Action(() =>
    {
        pictureBox1.Visible = true;
    }));

    HttpClient client = new HttpClient();
    var a = await client.GetAsync("http://www.baidu.com");
    Task<string> s = a.Content.ReadAsStringAsync();
    MessageBox.Show (s.Result);

    //System.Threading.Thread.Sleep(3000);
    //MessageBox.Show("hello:"+ i);
    this.Invoke(new Action(() =>
    {
        pictureBox1.Visible = false;
    }));
    return 0;
}

async private void call()
{
    //Func<string, string> funcOne = delegate(string s){ return "fff"; };
    object i = 55;
    var t = Task<Task<int>>.Factory.StartNew(new Func<object, Task<int>>(test2), i);
}

//----------或者----------

private async void test2()
{
    this.Invoke(new Action(() =>
    {
        pictureBox1.Visible = true;
    }));
    HttpClient client = new HttpClient();
    var a = await client.GetAsync("http://www.baidu.com");
    Task<string> s = a.Content.ReadAsStringAsync();
    MessageBox.Show (s.Result);
    this.Invoke(new Action(() =>
    {
        pictureBox1.Visible = false;
    }));
}

private void call()
{
    var t = Task.Run(new Action(test2));
    //相当于
    //Thread th= new Thread(new ThreadStart(test2));
    //th.Start();
}
```

