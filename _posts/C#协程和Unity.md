# C#协程和Unity

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
yiled break
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

