# C#协程和Unity多线程

[toc]

## 开启协程

```c#
StartCoroutine(string methodName);
StartCoroutine(IEnumerator method);
```

开启协程有两种参数：

1. 函数名字符串
2. 返回类型为迭代器(IEumerator)的函数

这里有关于迭代器注意点：

1. 方法中不能含有ref或者out类型的参数，但可以含有被传递的引用
2. 形参方法必须有返回值，且返回值类型为IEnumrator,返回值使用（yield retuen +表达式或者值，或者 yield break）语句



## 结束协程

1. stopAllCoroutine：暂停当前脚本的所有协程，不建议使用，很有很能会暂停其他的协程脚本，但建议在场景推出的使用使用，这样避免由于场景已经销毁，而协程又回来调用场景中的元素。
2. stopCoroutine(string methodName):暂停指定的协程
3. gameObject.active = false;停止这个对象身上的协程。

## 协程结束的标志

即yield return的IEnumerator已经迭代到最后一个，即这个函数已经运行到了函数体的末尾，IEnumerator这个迭代器的MoveNext就会返回false。这个Unity就会将这个IEnumerator从coroutine list中移除。

只有当这个对象的MoveNext()返回false时，即该Enumerator的Current已经迭代到最后一个元素了，才会执行yield return后面的语句。

## 中断函数类型

1. null or 1:在下一帧所有的Update()函数调用之后执行。
2. WaitForSeconds():等待指定秒数，在该帧(延迟过后的那一帧)所有的update()函数调用完后执行。及等待给定时间周期,受Time.timeScale影响，当Time.timeScale = 0f;yield return new WaitForSeconds();将不会满足。
3. WaitForFixedUpdate:等待一个固定帧。
4. WaitForEndOfFrame：等待帧结束，即等待渲染周期循环结束后执行。
5. StartCoroutine:等待一个新协程暂停，子协程中断后，会返回父协程，父协程暂停，返回父协程的上级函数。决定父协程结束的标志是子协程是否结束，当子协程结束后返回父协程执行其后的代码才算结束。
6. WWW等待一个加载完成，等待www的网络请求完成后，isDone = true后执行。
7. yield break:导致协程的执行条件不被满足，不会从当前的位置继续执行程序，而是直接从当前位置跳出函数体，回到函数的根部.

## 协程的执行顺序

1. 开始协程
2. 执行协程
3. 遇到中断指令中断协程
4. 返回上层函数继续执行上层函数的写一行代码
5. 中断指令结束后，继续执行中断指令之后的代码
6. 协程结束

# 协程的执行原理

协程函数的返回值时IEnumerator,它是一个迭代器，可以把它当成执行一个序列的某个节点的指针。

它提供了两个重要的接口，分别是Current(返回当前指向的元素)和MoveNext()(将指针向后移动一个单位，如果移动成功，则返回true)。

yield关键词用来声明序列中的下一个值或者是一个无意义的值。

如果使用yield return x(x是指一个具体的对象或者数值)的话，那么MoveNext返回为true并且Current被赋值为x,如果使用yield break使得MoveNext()返回为false。

如果MoveNext函数返回为true意味着协程的执行条件被满足，则能够从当前的位置继续往下执行。否则不能从当前位置继续往下执行。



## **IEnumerator**

```c#
public interface IEnumerator
{
	bool MoveNext();
	void Reset();
	Object Current{get;}
}
```

一个迭代器，三个基本的操作：Current/MoveNext/Reset， 这儿简单说一下其操作的过程。在常见的集合中，我们使用foreach这样的枚举操作的时候，最开始，枚举数被定为在集合的第一个元素前面，Reset操作就是将枚举数返回到此位置。

迭代器在执行迭代的时候，首先会执行一个 MoveNext， 如果返回true，说明下一个位置有对象，然后此时将Current设置为下一个对象，这时候的Current就指向了下一个对象。



## **Yield**

c#中的yield关键词，后面有两种基本的表达式：

```c#
yield return <expresion>
yield break
```

yield break就是跳出协程的操作，一般用在报错或者需要退出协程的地方。

yield return是用的比较多的表达式，具体的expresion可以以下几个常见的示例:

1. www:常见的web操作，在每帧末调用，会检查isDone/isError，如果true，则 call MoveNext.
2. WaitForSeconds: 检测间隔时间是否到了，返回true， 则call MoveNext
3. null: 直接 call MoveNext
4. WaitForEndOfFrame: 在渲染之后调用， call MoveNext



## Example

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Profiling;
 
public class QuotaCoroutine : MonoBehaviour
{
    // 每帧的额度时间，全局共享
    static float frameQuotaSec = 0.001f;
 
    static LinkedList<IEnumerator> s_tasks = new LinkedList<IEnumerator>();
 
    // Use this for initialization
    void Start()
    {
        StartQuotaCoroutine(Task(1, 100));
    }
 
    // Update is called once per frame
    void Update()
    {
        ScheduleTask();
    }
 
    void StartQuotaCoroutine(IEnumerator task)
    {
        s_tasks.AddLast(task);
    }
 
    static void ScheduleTask()
    {
        float timeStart = Time.realtimeSinceStartup;
        while (s_tasks.Count > 0)
        {
            var t = s_tasks.First.Value;
            bool taskFinish = false;
            while (Time.realtimeSinceStartup - timeStart < frameQuotaSec)
            {
                // 执行任务的一步, 后续没步骤就是任务完成
                Profiler.BeginSample(string.Format("QuotaTaskStep, f:{0}", Time.frameCount));
                taskFinish = !t.MoveNext();
                Profiler.EndSample();
 
                if (taskFinish)
                {
                    s_tasks.RemoveFirst();
                    break;
                }
            }
 
            // 任务没结束执行到这里就是没时间额度了
            if (!taskFinish)
                return;
        }
    }
 
    IEnumerator Task(int taskId, int stepCount)
    {
        int i = 0;
        while (i < stepCount)
        {
            Debug.LogFormat("{0}.{1}, frame:{2}", taskId, i, Time.frameCount);
            i++;
            yield return null;
        }
    }
}
```

## TaskAsync

```c#
using System.Collections;
using System.Diagnostics;
using System.Runtime.CompilerServices;
using System.Threading;
using System.Threading.Tasks;
using UnityEngine;
using TMPro;

public class CoroutineTest : MonoBehaviour
{
    Stopwatch stopwatch;
    static bool flag = false;
    static int count = 0;
    public TextMeshProUGUI text;
    static double result = 0;
    static double numPro = 0;
    // Start is called before the first frame update
    void Start()
    {
        stopwatch = new System.Diagnostics.Stopwatch();
        stopwatch.Start();
        //StartCoroutine(CoroutineCountDownSeq(3, "BasicCoCallA", "BasicCoCallB"));
        //var _ = TaskAsyncCountDown(3, "BasicAsyncCall");
        //保证不在主线程中进行计算,ConfigureAwait(false)
        //var _ = Task.Run(() => TaskAsyncConutDownSeq(3, "BasicAsyncCallA", "BasicAsyncCallB"));
        //CPUBindHead(120);
        //var t = TaskAwaitACoroutine();
        StartCoroutine(CoroutineAwaitATask());
        //var _ = CPUBigHeadAsync(120);
    }

    void Update()
    {
        text.text = numPro + "";
    }

    void LogToTUnityConsole(object content, string flag, [CallerMemberName] string callerName = null)
    {
        UnityEngine.Debug.Log($"{callerName}：\t{flag} {content}\t at \t {stopwatch.ElapsedMilliseconds} \t\t in thread {Thread.CurrentThread.ManagedThreadId}");
    }

    public IEnumerator CoroutineCountDown(int count, string flag = "")
    {
        for (int i = count; i >= 0; i--)
        {
            LogToTUnityConsole(i, flag);
            yield return new WaitForSeconds(1);
        }
    }

    public IEnumerator CoroutineCountDownSeq(int countDown, params string[] flags)
    {
        foreach (var flag in flags)
        {
            yield return StartCoroutine(CoroutineCountDown(countDown, flag));
        }
    }

    public async Task TaskAsyncCountDown(int count, string flag = "")
    {
        for (int i = count; i >= 0; i--)
        {
            LogToTUnityConsole(i, flag);
            //async函数中await部分使用了默认的Task编排器，将每次Task执行完成后的线程上下文转换回到Unity的主线程。
            //await Task.Delay(1000);
            //ConfigureAwait(false),这时await 将使用线程池调度器，不再将每个步骤强行回归到Unity主线程。
            await Task.Delay(1000).ConfigureAwait(false);
        }
    }

    public async Task TaskAsyncConutDownSeq(int countDown, params string[] flags)
    {
        foreach (string flag in flags)
        {
            await TaskAsyncCountDown(countDown, flag);
        }
    }

    //除了作为事件处理器这样的必需使用void函数的琴况下，这种没有返回Task无法进行外部调度编排的写法是强烈不建议使用的
    public async void VoidAsyncCountDown(int count, string flag = "")
    {
        for (int i = count; i >= 0; i--)
        {
            LogToTUnityConsole(i, flag);
            await Task.Delay(1000).ConfigureAwait(false);
        }
    }

    public void CPUBindHead(int n)
    {
        result = 1;
        LogToTUnityConsole(result, $"n!(n={n}) start");
        for (int i = 1; i <= n; i++)
        {
            result = result * i;
            numPro = result;
            Thread.Sleep(10);
        }
        LogToTUnityConsole(result, $"n!(n={n}) stop");
        Thread.Sleep(1000);

        //Texture change logic puts here

        LogToTUnityConsole(result, $"n!(n={n}) output");
    }

    public async Task CPUBigHeadAsync(int n)
    {
        result = 1;
        await Task.Run(() =>
        {
            LogToTUnityConsole(result, $"n!(n={n}) start");
            for (int i = 1; i <= n; i++)
            {
                result = result * i;
                numPro = result;
                Thread.Sleep(10);
            }
            LogToTUnityConsole(result, $"n!(n={n}) stop");
        });  //Scheduler will contiue execution in main thread here 

        await Task.Delay(1000);
        //Texture change logic puts here
        LogToTUnityConsole(result, $"n!(n={n}) output");
    }

    IEnumerator tempCoroutine(IEnumerator coro, System.Action afterExecutionCallback)
    {
        yield return coro;
        afterExecutionCallback();
    }

    //如果想要在async方法中调用 一个Coroutine 并对其进行控制，我们需要将Coroutine 的执行封装成一个简单的Task.

    //Unity 并没有对Coroutine的封装的完成callbak暴露出来，所以我们需要一个父级Coroutine调用来监视目标Coroutine的完成

    //这时我们可以使用 TaskCompeletionSource<T> 来将异步操作封装成Task
    public async Task TaskAwaitACoroutine()
    {
        await TaskAsyncCountDown(2, "precoro");
        var tcs = new System.Threading.Tasks.TaskCompletionSource<object>();

        StartCoroutine(tempCoroutine(CoroutineCountDown(3, "coro"), () => tcs.TrySetResult(null)));
        await tcs.Task;
        await TaskAsyncCountDown(2, "postcoro");
    }

    public IEnumerator CoroutineAwaitATask()
    {
        yield return CoroutineCountDown(2, "pretask");

        var task = TaskAsyncCountDown(3, "task");
        yield return new WaitUntil(() => task.IsCompleted || task.IsFaulted || task.IsCanceled);
        //Check task's return;
        if (task.IsCompleted)
        {
            LogToTUnityConsole("TDone", "task");
        }
        yield return CoroutineCountDown(2, "posttask");
    }

    public async Task TaskWaitCoroutine()
    {
        await TaskCountDown(2, "PreCo");
        await this.StartCoroutineAsync(CoroutineCountDown(3, "WaitCoroutine"));
        await TaskCountDown(2, "postcoro");
    }

    public IEnumerator CoroutineWaitTask()
    {
        yield return CoroutineCountDown(2, "PreTask");
        var co = TaskCountDown(2, "ExcuteTask").AsCoroutine();
        yield return co;
        LogToUnityConsole("TDone", "task");
        yield return CoroutineCountDown(2, "posttask");
    }
}

```

```c#
using System.Collections;
using System.Threading.Tasks;
using UnityEngine;

static class CoroutineTaskHelper
{
    public static async Task StartCoroutineAsync(this MonoBehaviour monoBehaviour, IEnumerator coroutine)
    {
        var tcs = new TaskCompletionSource<object>();
        monoBehaviour.StartCoroutine(emptyCoroutine(coroutine, tcs));
        await tcs.Task;
    }

    /// <summary>
    /// Translate a YieldInstruction to Task and run. It needs a Mono Behavior to locate Unity thread context.
    /// </summary>
    /// <param name="monoBehavior">The Mono Behavior that managing coroutines</param>
    /// <param name="yieldInstruction"></param>
    /// <returns>Task that can be await</returns>
    public static async Task StartCoroutineAsync(this MonoBehaviour monoBehavior, YieldInstruction yieldInstruction)
    {
        var tcs = new TaskCompletionSource<object>();
        monoBehavior
            .StartCoroutine(
                emptyCoroutine(
                    yieldInstruction,
                    tcs));
        await tcs.Task;
    }

    private static IEnumerator emptyCoroutine(YieldInstruction coro, TaskCompletionSource<object> completion)
    {
        yield return coro;
        completion.TrySetResult(null);
    }


    public static IEnumerator emptyCoroutine(IEnumerator coro, TaskCompletionSource<object> tsk)
    {
        yield return coro;
        tsk.TrySetResult(null);
    }

    /// <summary>
    /// Wrap a Task as a Coroutine.
    /// </summary>
    /// <param name="task">The target task.</param>
    /// <returns>Wrapped Coroutine</returns>
    public static CoroutineWithTask<System.Object> AsCoroutine(this Task task)
    {
        var coroutine = new WaitUntil(() => task.IsCompleted || task.IsFaulted || task.IsCanceled);
        return new CoroutineWithTask<object>(task);
    }

    /// <summary>
    /// Wrap a Task as a Coroutine.
    /// </summary>
    /// <param name="task">The target task.</param>
    /// <returns>Wrapped Coroutine</returns>
    public static CoroutineWithTask<T> AsCoroutine<T>(this Task<T> task)
    {

        return new CoroutineWithTask<T>(task);
    }

    /// <summary>
    /// Wrapped Task, behaves like a coroutine
    /// </summary>
    public struct CoroutineWithTask<T> : IEnumerator
    {

        private IEnumerator _coreCoroutine;

        /// <summary>
        /// Constructor for Task with a return value;
        /// </summary>
        /// <param name="coreTask">Task that need wrap</param>
        public CoroutineWithTask(Task<T> coreTask)
        {
            WrappedTask = coreTask;
            _coreCoroutine = new WaitUntil(() => coreTask.IsCompleted || coreTask.IsFaulted || coreTask.IsCanceled);
        }

        /// <summary>
        /// Constructor for Task without a return value;
        /// </summary>
        /// <param name="coreTask">Task that need wrap</param>
        public CoroutineWithTask(Task coreTask)
        {
            WrappedTask = Task.Run(async () =>
            {
                await coreTask;
                return default(T);
            });
            _coreCoroutine = new WaitUntil(() => coreTask.IsCompleted || coreTask.IsFaulted || coreTask.IsCanceled);
        }

        /// <summary>
        /// The task have wrapped in this coroutine.
        /// </summary>
        public Task<T> WrappedTask { get; private set; }


        /// <summary>
        /// Task result, if it have.  Calling this property will wait execution synchronously.
        /// </summary>
        public T Result { get { return WrappedTask.Result; } }


        /// <summary>
        /// Gets the current element in the collection.
        /// </summary>
        public object Current => _coreCoroutine.Current;

        /// <summary>
        /// Advances the enumerator to the next element of the collection.
        /// </summary>
        /// <returns>
        /// true if the enumerator was successfully advanced to the next element; false if
        /// the enumerator has passed the end of the collection.
        /// </returns>
        public bool MoveNext()
        {
            return _coreCoroutine.MoveNext();
        }

        /// <summary>
        /// Sets the enumerator to its initial position, which is before the first element
        /// in the collection.
        /// </summary>
        public void Reset()
        {
            _coreCoroutine.Reset();
        }
    }
}
```

在CPUBigHeadAsync和CPUBindHead的对比中，我们可以明显的发现，当使用CPUBigHeadAsync的时候，ui一直更新到结果，但是CPUBindHead直到结果出来才更新。

还有为了避免第一个程序还是使用主线程，我们可以使用Task.Run(() => func)来避免,**这样做可以避免第一个程序片段由于太耗时导致主线程卡顿**。

还有在使用task的时候，最好不要追求再次回到主线程，这样可以大大的加快运算时间。

