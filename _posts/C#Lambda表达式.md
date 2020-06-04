# C#Lambda表达式

[toc]

Lambda表达式是一种可用于创建委托或表达式目录树的匿名函数。它是一个匿名函数，形如(参数)=>{函数体}

## Labmda表达式之于委托

```c#
internal delegate string DelLambda();
internal delegate void DelLambdaOne(string str);
internal delegate int DelLambdaTwo(int a, int b);

public static void NoParam()
{
    DelLambda delLambda = () => { return "1"; };
    Console.WriteLine("Lambda no param:" + delLambda());
}

public static void OneParam()
{
    DelLambdaOne delLambdaOne = (str) =>
    {
        Console.WriteLine("Lambda one param:" + str);
    };
    delLambdaOne("one param");
}

public static void TwoParam()
{
    DelLambdaTwo delLambdaTwo = (a, b) =>
    {
        return a * b;
    };
    Console.WriteLine("Lambda two param:" + delLambdaTwo(2,3));
}
```

也可以是回调函数作为参数，然后在调用的时候，那个参数的位置直接使用Lambda函数。

```c#
private static void LambdaExcute(DelLambdaOne delLambdaOne)
{
    delLambdaOne("test");
}

public static void LambdaC()
{
    LambdaExcute(str => { Console.WriteLine(str); });
}
```

## Func和Lambda的用法

Func是一个.Net内置的委托。

Func<Result\>,Func<T1,Result\>是一个.Net内置的泛型委托

* Func<TResult\>
* Func<T,TResult>
* Func<T1,T2,TResult>
* Func<T1,T2,T3,TResult>
* Func<T1,T2,T3,T4,TResult>

Func是至少有一个返回值，参数可有可无。

```c#
private delegate string Say();
public static string SayHello()
{
	return "Hello";
}
public static string SayHelloParam(str)
{
	return "Hello" + str;
}
static void Main(string[] args)
{
	Say say = SayHello();
    Func<string> say1 = SayHello;
    Func<string, string> say2 = SayHelloParam;
    Console.WriteLine(say());
    Console.WriteLine(say1());
    Console.WriteLine(SayHelloParam());
}
```

接下来我们使用Func来实现。

## Action and Lambda的用法

Action<T\>的用法与Func几乎一样，调用方法也类似。

- Action
- Action<T\>
- Action<T1,T2>
- Action<T1,T2,T3>
- Action<T1,T2,T3,T4>



```c#
public static void TestAction()
{
    Action<string, string> hello = (str1, str2) => { Console.WriteLine(str2 + str1); };
    hello("hello ", "jow");
    Action hello1 = () => { Console.WriteLine("hello"); };
    hello1();
}
```

## Action 和 Func的区别

它们两个几乎作用一样，Func的三角括号中最少有一个类型，这个类型就是返回值类型，意味着Func至少存在返回值，这个返回值类型在参数的最后一个。

Action不存在返回值，所以没有返回值类型参数。

## Func还可以作为参数

```c#
string Test(Func<string,string>);
public static int Sun<TSource>(this IEnumerable<TSource> source, Func<TSource, int> selector);
```

