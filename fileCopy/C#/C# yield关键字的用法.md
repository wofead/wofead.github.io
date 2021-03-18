# C# yield关键字的用法

[toc]



## yield实现的功能

**yield return:**

先看下面的代码，通过yield return实现了类似于foreach遍历数组的功能，说明yield return也是用来实现迭代器的功能的。

```c#
//一个返回类型为IEnumerable<int>，其中包含三个yield return
public static IEnumerable<int> EnumerableFuc()
{
    yield return 1;
    yield return 2;
    yield return 3;
}

static void Main(string[] args)
{
    foreach (int item in EnumerableFuc())
    {
        Console.WriteLine(item);
    }
    Console.ReadKey();
}
//1
//2
//3
```

**yield break:**

再看下面的代码，只输出了1，2，没有输出3，说明这个迭代器被yield break停掉了，所以yield break是用来终止迭代的。

```c#
using static System.Console;
using System.Collections.Generic;
class Program
{
    //一个返回类型为IEnumerable<int>，其中包含三个yield return
    public static IEnumerable<int> enumerableFuc()
    {
        yield return 1;
        yield return 2;
        yield break;
        yield return 3;
    }

    static void Main(string[] args)
    {
        //通过foreach循环迭代此函数
        foreach(int item in enumerableFuc())
        {
            WriteLine(item);
        }
        ReadKey();
    }
}
//1
//2
```

## 可使用地方

**只能使用在返回类型必须为IEnumerable、IEnumerable\<T>、IEnumerator或IEnumerator\<T>的方法、运算符、get访问器中。**

## yield关键字的实现原理

我们用while循环代替foreach循环，发现我们虽然没有实现GetEnumerator()，也没有实现对应的IEnumerator的MoveNext()，和Current属性，但是我们仍然能正常使用这些函数。

通过使用ILSpy生成的exe进行反编译可以找到原因。

具体内容可以看[链接](https://www.cnblogs.com/blueberryzzz/p/8678700.html),可以发现在反编译的代码里面会声明一个新的类，这个类继承IEnumerable、Enumerable\<T>、IEnumerator 或 IEnumerator\<T>，这时我们应该已经能猜到这个新的类就是我们没有实现对应的IEnumerator的MoveNext()，和Current属性，这是我们仍然能正常使用这些函数的原因了。

**用enumberableFuc()来进行迭代的真实流程就是：**

1. 运行enumberableFuc()函数，获取代码自动生成的类的实例。
2. 接着调用GetEnumberator()函数，将获取的类自己作为迭代器开始迭代。
3. 每次运行MoveNext()，state增加1，通过switch语句可以让每次调用MoveNext()的时候执行不同部分的代码。
4. MoveNext()返回false，结束。

**这也能说明yield关键字其实是一种语法糖，最终还是通过实现IEnumberable<T>、IEnumberable、IEnumberator<T>和IEnumberator接口实现的迭代功能。**