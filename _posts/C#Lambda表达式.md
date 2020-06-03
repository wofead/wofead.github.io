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

