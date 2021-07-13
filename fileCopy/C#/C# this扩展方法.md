# C# this扩展方法

扩展方法被定义为静态方法，但他们是通过实例方法语法进行调用的。它们的第一个参数指定该方法作用于那个类型，并且改参数以this修饰符作为前缀。扩展方法当然不能破坏面向封装的概念，所以只能是访问所扩展类的public成员。

## 给string类型添加一个Add方法，该方法的作用是给字符串增加一个字母a

```c#
 public static  string  Add(this string stringName)
 {
     return stringName+"a";
 }
```

**定义：**

1. 扩展方法能使你能够向现有类型添加“添加”方法，而无需创建新的派生类型,重新编译或以其他方式修改原始类型。
2. 扩展方法是一种特殊的静态方法，但可以像扩展类型上的实例方法一样调用。

**注意：**

1. 扩展方法的第一个参数指定该方法作用于那个类型，并且此参数用this为前缀修饰。仅当你使用using指令将命名空间显示导入到源码之中后，扩展方法才位于范围中。
2. 在代码中，可以使用实例方法语法调用改扩展方法，但是，编译器生成的中间语言（IL）会将代码转换为对静态方法的调用。因此，并未真正 违反封装原则，实际上，扩展方法无法访问到他们所扩展类型中的私有变量。
3. 不能重写扩展方法
4. 与接口或者类具有相同名称和签名的扩展方法永远不会被调用（编译时优先级：实例方法 > 扩展方法）

**使用是示例：**

**定义扩展方法**，`namespace`为`PipelineExtensions`。

```c#
namespace PipelineExtensions
{
    public static class StringExtensions
    {
        // 扩展方法---计算字符串长度
        public static int WordCount(this string str)
        {
            return str.Length;
        }
    }
}
```

****

**使用此扩展方法**

1. 先通过using把namespace引入到使用文件中。
2. 使用扩展方法 WordCount(this string str)中的第一个参数类型string为类型的参数调用此扩展方法

```c#
using PipelineExtension;
using system;
namespace pipeline
{
	public plass Program
    {
        public static void Main(string[] args)
        {
            string ex = "yyjs";
            int cout = ex.WordCount();
            Console.WriteLine(count);
            Console.ReadKey();
        }
    }
}
```

