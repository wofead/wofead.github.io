# c#中ref和out的区别浅析

这篇文章主要介绍c#ref和out的区别浅析，当一个方法需要多个返回值就需要使用ref和out，这两个方法有什么区别呢？

**MSDN：
**    ref 关键字使参数按引用传递。其效果是，当控制权传递回调用方法时，在方法中对参数所做的任何更改都将反映在该变量中。若要使用 ref 参数，则方法定义和调用方法都必须显式使用 ref 关键字。

 out 关键字会导致参数通过引用来传递。这与 ref 关键字类似，不同之处在于 ref 要求变量必须在传递之前进行初始化。若要使用 out 参数，方法定义和调用方法都必须显式使用 out 关键字。

```c#
static int GetIntResult(int[] arr, ref float avg, ref int max, ref int min)
{
    int sum = 0;
   	max = arr[0];
    min = arr[0];
    for (int i = 0; i < arry.Length; i++)
    {
        sum += arry[i];

        if (max < arry[i])
        {
            max = arry[i];
        }
        if (min > arry[i])
        {
            min = arry[i];
        }
    }
    avg = sum / arry.Length;
    return sum;
}

static void Main(string[] args)
{
    int[] arr = { 1,2,3,4,5,6,7,8,9};
    //float avg;//会报错
    //int max;//会报错
    //int min;//会报错
    //必须初始化
    float avg = 0;
    int max = 0;
    int min = 0;
    int sum = GetIntResult(arr, ref avg, ref max, ref min);
}
```

 ref这个关键字告诉c#编译器被传递的参数值指向与调用代码中变量相同的内存。这样，如果被调用的方法修改了这些值然后返回的话，调用代码的变量也就被修改了。

 ref 关键字使参数按引用传递。其效果是，当控制权传递回调用方法时，在方法中对参数所做的任何更改都将反映在该变量中（avg，max，min的初始值为0，调用方法后值改变）。若要使用 ref 参数，则方法定义和调用方法都必须显式使用 ref 关键字。

**Out**

换成out之后，上面的方法不再适用，报错，错误 ： 控制离开当前方法之前必须对 out 参数“min”和"max"赋值。你会发现这里max和min在循环外并未初始化。所以才会出错。

 out 关键字会导致参数通过引用来传递。这与 ref 关键字类似，不同之处在于 ref 要求变量必须在传递之前进行初始化。若要使用 out 参数，方法定义和调用方法都必须显式使用 out 关键字。
