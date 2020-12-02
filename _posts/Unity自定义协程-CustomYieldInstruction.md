# Unity自定义协程-CustomYieldInstruction

Unity的Coroutine非常好用。在很多需要延迟执行的地方，都可省掉大量功夫。

[toc]

## Coroutine的缺点

普通协程有以下这些缺点：

1. 嵌套协程依赖StartCoroutine，从而代码依赖MonoBehavior
2. 执行异步逻辑的时候，需要些while等待，代码臃肿难看
3. 协程运行结果无法直接返回，需要另外处理，特别费力。

eg:

```c#
public IEnumerator Coroutine()
{
    //-----------------------------嵌套协程-----------------------------//
    yield return StartCoroutine(InternalCoroutine());

    //-----------------------------协程等待异步逻辑执行完毕-----------------------------//
    var running = false;
    DoSomething(() =>
    {
        running = true;
    });

    while (running == false)
    {
        yield return 0;
    }

    //-----------------------------协程返回值-----------------------------//
    running = false;
    var rtn = "bbb";
    DoSomthing2(s =>
    {
        running = true;
        rtn = s;
    });

    while (running == false)
    {
        yield return 0;
    }

}
```

## CustomYieldInstruction

现在我们可以通过集成CustomYieldInstruction这个接口，很好的解决了上面的问题。

```c#
public class WWW : CustomYieldInstruction, IDisposable {...}
public class WaitForSecondsRealtime : CustomYieldInstruction {...}
public sealed class WaitUntil : CustomYieldInstruction {...}
public sealed class WaitWhile : CustomYieldInstruction {...}

public abstract class CustomYieldInstruction : IEnumerator
{
    /// <summary>
    ///   <para>Indicates if coroutine should be kept suspended.</para>
    /// </summary>
    public abstract bool keepWaiting { get; }

    public object Current
    {
        get
        {
            return (object) null;
        }
    }

    public bool MoveNext()
    {
        return this.keepWaiting;
    }

    public void Reset()
    {
    }
}
```

比如下边这个下载文件的协程逻辑，我们把返回值Code放在类中。然后当下载完成后。再设置Code的值使协程被打断。

```c#
var download = new DownloadCoroutine(...);
yield return download;

Debug.Log(download.Code);
```

```c#
public class DownloadCoroutine : CustomYieldInstruction
{
    public int Code = -1;

    public DownloadCoroutine(string fromPath, string toPath, OnDownloadProgressDelegate onProgress)
    {
        Debug.Log("下载文件 ： " + fromPath + " -> " + toPath);

        var httpRequest = new HTTPRequest(new Uri(fromPath), HTTPMethods.Get, (request, response) =>
        {
            if (response == null)
            {
                Code = 500;
                return;
            }

            Debug.Log("下载完成 ： " + response.StatusCode);
            File.WriteAllBytes(toPath, response.Data);
            Code = response.StatusCode;
        });

        httpRequest.OnProgress = onProgress;
        httpRequest.Send();
    }

    public override bool keepWaiting
    {
        get { return Code == -1; }
    }
}
```



```c#
class YieldInstructionTest : CustomYieldInstruction
{
    public bool Complete = false;
    public override bool keepWaiting { get { return !Complete; } }
}


public IEnumerator DoSomething()
{
    var waitForComplete = new WaitForComplete();
    func(waitForComplete);
    yield return waitForComplete;
}


```

这个示例展示了自定义协程的用法，DoSomething会被挂起，直到keepWaiting==false



# CustomYieldInstruction  unity文档

描述：**用于挂起协程的自定义生成指令的基类。**

[CustomYieldInstruction](https://docs.unity3d.com/ScriptReference/CustomYieldInstruction.html)，允许实现自定义yield指令，以挂起协同程序执行，直到事件发生。在内部，自定义生成指令只是另一个运行的协同程序。要实现它，继承CustomYieldInstruction类并重写keepWaiting属性。保持协程暂停返回true。为了让协同程序继续执行，返回false。`keepWaiting`属性,在`Update`和`LateUpdate`之间每一帧被查询。

可以这么理解，如果想要被挂起，就返回true，想要执行就返回false。

eg：

```c#
// Example showing how a CustomYieldInstruction script file
// can be used. This waits for the left button to go up and then
// waits for the right button to go down.
using System.Collections;
using UnityEngine;
public class ExampleScript:MonoBehaviour
{
	void Update()
	{
		if(Input.GetMouseButtonUp(0))
		{
			Debug.Log("Left mouse button up");
			StartCoroutine(waitForMouseDown());
		}
	}
	
	public IEnumerator waitForMouseDown()
	{
		yield return new WaitForMouseDown();
		Debug.Log("Right mouse button pressed");
	}
}
```

下面的脚本实现了keepWaiting的可重写版本。这个c#实现可以被JS使用。在这种情况下，确保这个c#脚本在一个插件文件夹中，这样它就会在上面的JS脚本示例之前被编译。

```c#
using UnityEngine;
public class aitForMouseDown : CustomYieldInstruction
{
    public override bool keepWaiting
    {
        get
        {
            return !Input.GetMouseButtonDown(1);
        }
    }
    public WaitForMouseDown()
    {
        Debug.Log("Waiting for Mouse right button down");
    }
}
```



```c#
using System.Collections;
using UnityEngine;
using System;

// Implementation of WaitWhile yield instruction. This can be later used as:
// yield return new WaitWhile(() => Princess.isInCastle);
class WaitWhile1 : CustomYieldInstruction
{
    Func<bool> m_Predicate;

    public override bool keepWaiting { get { return m_Predicate(); } }

    public WaitWhile1(Func<bool> predicate) { m_Predicate = predicate; }
}
```

要获得更多的控制并实现更复杂的生成指令，您可以直接继承System.Collections.IEnumerator类。在本例中，以与实现keepWaiting属性相同的方式实现MoveNext()方法。除此之外，你还可以在Current属性中返回一个对象，这个对象将在执行MoveNext()方法后被Unity的协同调度程序处理。例如，如果当前返回了从IEnumerator继承的另一个对象，那么当前枚举器将被挂起，直到返回的枚举器完成为止。

```c#
using System;
using System.Collections;

// Same WaitWhile implemented by inheriting from IEnumerator.
class WaitWhile2 : IEnumerator
{
    Func<bool> m_Predicate;

    public object Current { get { return null; } }

    public bool MoveNext() { return m_Predicate(); }

    public void Reset() {}

    public WaitWhile2(Func<bool> predicate) { m_Predicate = predicate; }
}
```

这种方式就是我们我们Unity中的`WaitUtil`类。

```c#
//
// 摘要:
//     Suspends the coroutine execution until the supplied delegate evaluates to true.
public sealed class WaitUntil : CustomYieldInstruction
{
    public WaitUntil(Func<bool> predicate);

    public override bool keepWaiting { get; }
}
```

就像下面那个例子，等待task完成。

`new WaitUntil(() => coreTask.IsCompleted || coreTask.IsFaulted || coreTask.IsCanceled)`

```c#
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
```



# YieldInstruction

是所有yield指令的基类。

WaitForSeconds、WaitForFixedUpdate、Coroutine和MonoBehaviour.StartCoroutine这些函数都有继承YieldInstruction。