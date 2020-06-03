# C#委托

[toc]

自己之前还是弄明白过委托的，但是好久没有见到委托了，就又忘记了，自己又不愿意回顾，今天就强迫自己回顾一下，然后再次总结一下，用blog记录一下。

C#的委托和Lua的handler的作用应该是一样的，触发发了某一件事情，然后调用一个函数，只不过lua条用的函数随便写，而c#需要使用delegate声明的那个函数声明。

这样回顾下来我们会发现delegate需要三个部分：

1. delegate一个函数声明，这个函数声明是用来规定，以后回调的函数就行是什么样子了，有名称和参数，没有函数体。
2. 触发回调函数的委托函数。
3. 具体的回调函数,也可以是Lambda函数。

似乎认真回忆一下，都可以回忆出来呀，看来还是自己逃避了，接下来我来分析一篇博文学习一下。

来自： https://cnblogs.com/jixiaosa/p/10687068.html

## 定义一个委托

```c#
internal delegate void FeedBack(int value);//需要一个int类型的参数，返回为void
```

## 定义回调方法

回调方法一般有两种形式，他们还是存在区别的，一个是静态的，一个是实例。在这里我们需要注意，回调方法的签名和委托对象必须保持一致。

```c#
private static void FeedbackToConsole(int value)
{
	Console.WriteLine("Item="+value);
}
private void InstanceFeedbackToConsole(int value)
{
	Console.WriteLine("item=" + value);
}
```

## 编写一个方法来触发回调函数

这个方法的参数一定存在类型为委托的参数，否者调用不到回调函数。

```c#
private static void Counter(int  from, int to, FeedBack fb)
{
	for(int val = from; val <= to;val ++)
	{
		if(fb != null)
		{
			fb(val);
		}
	}
}
```

## 定义Counter的方法调用

```c#
private static void StaticDelegateDemo()
{
	Counter(1,10,null);
	Counter(1,10,new FeedBack(FeedbackToConsole));
}
private static void InstanceDelegateDemo()
{
	Program p = new Program();
    Counter(1, 10, null);
    Counter(1, 5, new Feedback(p.InstanceFeedbackToConsole));
}
```

## 委托链

委托链是委托对象的集合。可以利用委托链调用集合中的委托所绑定的全部方法。继续在原有的基础上添加委托链的方法。

新添加的两个方法本质上没有区别，都是对委托链的实现，不同的是方法，明显是第二个方法更加的精简一些。这是因为c#编译器重载了+=和-=操作符，这连个操作符分别调用了Combine和Remove方法。

委托链在执行的时候，每次执行链的时候，会把里面的函数全部执行一遍。

```c#
public void ChainTest()
    {
        FeedBack fb1 = new FeedBack(StaticFeedBack);
        FeedBack fb2 = new FeedBack(InstanceFeedBack);
        FeedBack fbChain = null;
        fbChain += fb1;
        fbChain += fb2;
        ExcuteFunc(fbChain);
        fbChain -= fb2;
        ExcuteFunc(fbChain);
    }
```



```C#
using System;

public class DelegateTest
{
    internal delegate void FeedBack(int value);
    public DelegateTest()
    {
    }

    private static void StaticFeedBack(int value)
    {
        Console.WriteLine("StaticFeedBack:" + value);
    }

    private void InstanceFeedBack(int value)
    {
        Console.WriteLine("InstanceFeedBack:" + value);
    }

    private static void ExcuteFunc(FeedBack feedBack)
    {
        for (int i = 0; i < 5; i++)
        {
            feedBack(i);
        }
    }

    public static void StaticTest()
    {
        ExcuteFunc(new FeedBack(StaticFeedBack));
    }

    public void InstanceTest()
    {
        ExcuteFunc(new FeedBack(InstanceFeedBack));
    }

    public void ChainTest()
    {
        FeedBack fb1 = new FeedBack(StaticFeedBack);
        FeedBack fb2 = new FeedBack(InstanceFeedBack);
        FeedBack fbChain = null;
        fbChain += fb1;
        fbChain += fb2;
        ExcuteFunc(fbChain);
        fbChain -= fb2;
        ExcuteFunc(fbChain);
    }
    
    public void LambdaTest()
    {
        ExcuteFunc(value =>
        {
            Console.WriteLine("Lambda:" + value);
        });
    }
}

```

```c#
static void Main(string[] args)
{
    DelegateTest.StaticTest();
    DelegateTest delegateTest = new DelegateTest();
    delegateTest.InstanceTest();
    delegateTest.ChainTest();
    delegateTest.LambdaTest();
}
```



## C#为委托提供了简化

1. 不需要构造委托对象

   ```c#
   //Counter(1, 10, new Feedback(FeedbackToConsole));
   Counter(1, 10, FeedbackToConsole);
   ```

2. 可以不需要回调方法（以lambda表达式来实现），即使用lambda表达式来代替函数体。

   ```c#
   private static void FeedbackToConsole(int value)
   {
       // 依次打印数字
       Console.WriteLine("Item=" + value);
   }
   private static void StaticDelegateDemo()
   {
       Console.WriteLine("---------委托调用静态方法------------");
       Counter(1, 10, null);
       //Counter(1, 10, new Feedback(FeedbackToConsole));
       //Counter(1, 10, FeedbackToConsole);
       Counter(1, 10, value => Console.WriteLine(value));
   
   }
   ```

   