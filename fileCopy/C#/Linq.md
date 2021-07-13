# Linq

[toc]

## 为什么使用Linq

要理解为什么使用LINQ，先来看下面一个例子。假设有一个整数类型的数组，找到里面的偶数并进行降序排序。

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace LinqOfSelectOperation
{
    class Program
    {
        static void Main(string[] args)
        {
            // 查询出数组中的偶数并排序
            int[] ints = { 5, 2, 0, 66, 4, 32, 7, 1 };
            // 定义一个整数类型的集合，用来存放数组中的偶数
            List<int> list = new List<int>();
            // 遍历数组查询出偶数放到集合中
            foreach (int i in ints)
            {
                // 如果是偶数，把偶数加入到集合中
                if (i % 2 == 0)
                {
                    list.Add(i);
                }
            }

            // 正序排序
            list.Sort();
            // 反转
            list.Reverse();
            // 输出
            Console.WriteLine(string.Join(",",list));

            Console.ReadKey();
        }
    }
}
```

使用for循环很麻烦，而且不可维护和可读。C#2.0引入了delegate，可以使用委托来处理这种场景，代码如下图所示：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace LinqOfSelectOperation
{
    // 定义委托
    delegate bool FindEven(int item);

    class IntExtension
    {
        public static int[] where(int[] array, FindEven dele)
        {
            int[] result=new int[5];
            int i = 0;
            foreach (int item in array)
            {
                if (dele(item))
                {
                   result[i]=item;
                    i++;
                }
            }

            return result;
        }
    }
    class Program
    {
        static void Main(string[] args)
        {
            // 查询出数组中的偶数并排序
            int[] ints = { 5, 2, 0, 66, 4, 32, 7, 1 };

            //delegate(int item){return item % 2 == 0;}表示委托的实现
            List<int> list = IntExtension.where(ints, delegate(int item)
            {
                return item % 2 == 0;
            }).ToList();
            // 正序排序
            list.Sort();
            // 反转
            list.Reverse();
            // 输出
            Console.WriteLine(string.Join(",", list));

            Console.ReadKey();
        }
    }
}
```

 所以，有了C#2.0，通过使用委托有了代理的优势，不必使用for循环来查询不同条件的数组。例如你可以使用相同的委托来查找数组中的奇数，并降序排序输出，代码如下图所示：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace LinqOfSelectOperation
{
    // 定义委托
    delegate bool FindEven(int item);

    class IntExtension
    {
        public static int[] where(int[] array, FindEven dele)
        {
            int[] result=new int[3];
            int i = 0;
            foreach (int item in array)
            {
                if (dele(item))
                {
                   result[i]=item;
                    i++;
                }
            }

            return result;
        }
    }
    class Program
    {
        static void Main(string[] args)
        {
            // 查询出数组中的奇数并排序
            int[] ints = { 5, 2, 0, 66, 4, 32, 7, 1 };

            //delegate(int item){return item % 2 != 0;}表示委托的实现
            List<int> list = IntExtension.where(ints, delegate(int item)
            {
                return item % 2 != 0;
            }).ToList();
            // 正序排序
            list.Sort();
            // 反转
            list.Reverse();
            // 输出
            Console.WriteLine(string.Join(",", list));

            Console.ReadKey();
        }
    }
}
```

 虽然使用delegate可以使程序的可读性增加了，但是C#团队认为他们仍然需要使代码更加紧凑和可读，所以他们在C#3.0中引入了扩展方法、Lambda表达式、匿名类型等新特性，你可以使用C#3.0的这些新特性，这些新特性的使用LINQ的前提，可以用来查询不同类型的集合，并返回需要的结果。

下面的示例演示了如何使用LINQ和Lambda表达式根据特定条件来查询数组，示例代码如下：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace LinqOfSelectOperation
{
    class Program
    {
        static void Main(string[] args)
        {
            // 查询出数组中的奇数并排序
            int[] ints = { 5, 2, 0, 66, 4, 32, 7, 1 };

            // 使用LINQ和Lambda表达式查询数组中的偶数
            int[] intEvens= ints.Where(p => p % 2 == 0).ToArray();
            // 使用LINQ和Lambda表达式查询数组中的奇数
            int[] intOdds = ints.Where(p => p % 2 != 0).ToArray();

            // 输出
            Console.WriteLine("偶数：" + string.Join(",", intEvens));
            Console.WriteLine("奇数：" + string.Join(",", intOdds));

            Console.ReadKey();
        }
    }
}
```

在上面的例子中可以看到，我们在单个语句中使用LINQ和Lambda表达式指定不同的查询条件，因此，LINQ使代码更加紧凑和可读，并且它也可以用于查询不同的数据源。看到这里的时候，你可能会问：究竟什么是LINQ呢？下面将会具体讲解什么是LINQ。

## 什么是LINQ

长期以来，开发社区形成以下的格局：

1. 面向对象与数据访问两个领域长期分裂，各自为政。

2. 编程语言中的数据类型与数据库中的数据类型形成两套不同的体系，例如：
   C#中字符串用string数据类型表示。

   SQL中字符串用NVarchar/Varchar/Char数据类型表示。

3. SQL编码体验落后没有智能感知效果。没有严格意义上的强类型和类型检查。

4. SQL和XML都有各自的查询语言，而对象没有自己的查询语言。

上面描述的问题，都可以使用LINQ解决，那么究竟什么是LINQ呢？

LINQ（Language Integrated Query）即语言集成查询。

LINQ是一组语言特性和API，使得你可以使用统一的方式编写各种查询。用于保存和检索来自不同数据源的数据，从而消除了编程语言和数据库之间的不匹配，以及为不同类型的数据源提供单个查询接口。

LINQ总是使用对象，因此你可以使用相同的查询语法来查询和转换XML、对象集合、SQL数据库、ADO.NET数据集以及任何其他可用的LINQ提供程序格式的数据。

LINQ主要包含以下三部分：

1、LINQ to Objects   主要负责对象的查询。

2、LINQ to XML      主要负责XML的查询。

3、LINQ to ADO.NET  主要负责数据库的查询。

　　LINQ to SQL

　　LINQ to DataSet

　　LINQ to Entities

![img](../image/Linq/1033738-20180113120043535-1853832308.png)

## LINQ的优势

1、熟悉的语言：开发人员不必为每种类型的数据源或数据格式学习新的语言。

2、更少的编码：相比较传统的方式，LINQ减少了要编写的代码量。

3、可读性强：LINQ增加了代码的可读性，因此其他开发人员可以很轻松地理解和维护。

4、标准化的查询方式：可以使用相同的LINQ语法查询多个数据源。

5、类型检查：程序会在编译的时候提供类型检查。

6、智能感知提示：LINQ为通用集合提供智能感知提示。

7、整形数据：LINQ可以检索不同形状的数据。

## LINQ操作语法

LINQ查询时有两种语法可供选择：查询表达式语法（Query Expression）和方法语法（Fluent Syntax）。

### 查询表达式语法

查询表达式语法是一种更接近SQL语法的查询方式。

LINQ查询表达式语法如下：

```c#
from<range variable> in <IEnumerable<T> or IQueryable<T> Collection>
<Standard Query  Operators> <lambda expression>
<select or groupBy operator> <result   formation>
```

LINQ查询表达式

| 约束 | LINQ查询表达式必须以from子句开头，以select或group子句介绍 |
| ---- | --------------------------------------------------------- |

| 关键字              | 功能                                                         |
| ------------------- | ------------------------------------------------------------ |
| from....in...       | 指定要查询的数据源以及范围变量，多个from子句则表示从多个数据源查找数据。注意：C#编译器会把“复合from子句”的查询表达式转换为SelectMany()扩展方法。 |
| join…in…on…equals…  | 指定多个数据源的关联方式                                     |
| let                 | 引入用于存储查询表达式中子表达式结果的范围变量。通常能达到层次感会更好，使代码更易于阅读。 |
| orderby、descending | 指定元素的排序字段和排序方式。当有多个排序字段时，由字段顺序确定主次关系，可指定升序和降序两种排序方式 |
| where               | 指定元素的筛选条件。多个where子句则表示了并列条件，必须全部都满足才能入选。每个where子句可以使用谓词&&、\|\|连接多个条件表达式。 |
| group               | 指定元素的分组字段。                                         |
| select              | 指定查询要返回的目标数据，可以指定任何类型，甚至是匿名类型。（目前通常被指定为匿名类型） |
| into                | 提供一个临时的标识符。该标识可以引用join、group和select子句的结果。1)    直接出现在join子句之后的into关键字会被翻译为GroupJoin。（into之前的查询变量可以继续使用）2)    select或group子句之后的into它会重新开始一个查询，让我们可以继续引入where, orderby和select子句，它是对分步构建查询表达式的一种简写方式。（into之前的查询变量都不可再使用） |

查询语法从一个From子句开始，然后是一个Range变量。 From子句的结构类似于“From rangeVariableName in IEnumerablecollection”。 在英语中，这意味着，从集合中的每个对象。 它类似于foreach循环：foreach（student in studentList）。

在From子句之后，您可以使用不同的标准查询运算符来过滤，分组，连接集合的元素。 LINQ中有大约50个标准查询运算符。标准查询运算符后面通常跟一个条件，这个条件通常使用lambda表达式来表示。

LINQ查询语法总是以Select或Group子句结束。 Select子句用于对数据进行整形。 您可以选择整个对象，因为它是或只有它的一些属性。 在上面的例子中，我们选择了每个结果字符串元素。

例如：我们要从数组中查询出偶数，查询表达式示例代码如下：

```c#
var result = from p in ints where p % 2 == 0 select p;
```

查询表达式语法要点总结：

1. 查询表达式语法与SQL（结构查询语言）语法相同。
2. 查询语法必须以from子句开头，可以以Select或GroupBy子句结束 。
3. 使用各种其他操作，如过滤，连接，分组，排序运算符以构造所需的结果。
4. 隐式类型变量 - var可以用于保存LINQ查询的结果。

## 方法语法

方法语法（也称为流利语法）主要利用System.Linq.Enumerable类中定义的扩展方法和Lambda表达式方式进行查询，类似于如何调用任何类的扩展方法。

以下是一个示例LINQ方法语法的查询，返回数组中的偶数：

```c#
var result = ints.Where(p => p % 2 == 0).ToArray();
```

从上面的示例代码中可以看出：方法语法包括扩展方法和Lambda表达式。 扩展方法Where（）在Enumerable类中定义。

如果你检查Where扩展方法的签名，你会发现Where方法接受一个谓词委托，如`Func <Student，bool>`。 这意味着您可以传递任何接受Student对象作为输入参数的委托函数，并返回一个布尔值，如下图所示。 lambda表达式作为在Where子句中传递的委托传递。 在下一节中学习lambda表达式。

### 查询表达式语法VS方法语法

查询表达式语法与方法语法存在着紧密的关系

1. CLR本身并不理解查询表达式语法，它只理解方法语法。
2. 编译器负责在编译时将查询表达式语法翻译为方法语法。
3. 大部分方法语法都有对应的查询表达式语法形式：如Select()对应select、OrderBy()对应orderby
4. 部分查询方法目前在C#中还没有对应的查询语句：如Count()和Max()，这是只能采用以下替代方案：
   　　方法语法
   　　查询表达式语法+方法语法的混合方式

## Lambda表达式解剖

C#3.0（.NET3.5）中引入了Lambda表达式和LINQ。Lambda表达式是使用一些特殊语法表示匿名方法的较短方法。

最基本的Lambda表达式语法如下：

（参数列表）=>{方法体}

1. 参数列表中的参数类型可以是明确类型或者推断类型。
2. 如果是推断类型，则参数的数据类型将由编辑器根据上下文自动推断出来。

****

**让我们看看Lambda表达式是如何从匿名方法演变而来的。**

相关示例：

```c#
delegate(int item) { return item % 2 == 0; };
```

1. Lambda表达式从匿名方法演变，首先删除delegate关键字和参数类型并添加Lambda运算符=>，演变后的代码如下：

   ```c#
   (item)=>{return item % 2 == 0;};
   ```

2. 如果我们只有一个返回值的语句，那么我们不需要花括号、返回和分号，所以我们可以去掉这些符号，演变后的代码如下：

   ```c#
   (item)=>item %2 == 0;
   ```

3. 如果我们只有一个参数，我们也可以删除（），代码如下：

   ```c#
   item=>item %2 == 0;
   ```

   因此，我们得到如下所示的Lambda表达式：

   item => item % 2 == 0

   其中item是参数，=>是Lambda运算符，item % 2 == 0是正文表达式。



---

**具有多个参数的Lambda表达式**

如果需要传递多个参数，那么必须将参数括在括号内，如下所示：

```c#
(ints, item) => ints.Contains(item);
```

 如果不想使用推断类型，那么可以给出每个参数的类型，如下所示：

```c#
(int[] ints, int item) => ints.Contains(item)
```

---

**不带任何参数的Lambda表达式**

```c#
() => Console.WriteLine("这是一个不带任何参数的Lambda表达式");
```

---

**正文表达式中有多条语句**

在前面讲过，如果正文表达式有一个语句，那么可以去掉方法体外面的大括号。如果正文表达式中有多条语句，那么必须用大括号将正文表达式括起来，如下所示：

```c#
(ints, item) =>
{
        Console.WriteLine("这是包含多条语句的Lambda表达式");
        return ints.Contains(item);
}; 
```

---

**表达式中的局部变量**

你可以在表达式的主体中声明一个变量，以便在表达式主体的任何位置使用它，如下所示：

```c#
ints=>{
	int item = 10;
	return ints.Contains(item);
}
```

## Lambda表达式中的内置泛型委托

### 当你想从lambda表达式返回一些东西时，使用Func <> delegate。 

其中T是输入参数的类型，TResult是返回类型。

示例代码如下：

```c#
Func<int[], bool> isContains = p =>p.Equals(10);
int[] ints = {5,2,3,8,56,3};
bool isEquals = isContains(ints);
```

在上面的例子中，Func委托期望第一个输入参数是int[]类型，返回类型是boolean。Lambda表达式是p => p.Equals(10)。

### Action委托

与Func委托不同，Action委托只能有输入参数。 当不需要从lambda表达式返回任何值时，请使用Action委托类型。

示例代码如下：

```c#
Action<int[]> PrintItem = p=>{
    foreach(int item in p){
        Console.WriteLine(item);
    }
};
int[] ints = {5,2,3,8,56,76};
PrintItem(ints);

```

### 在LINQ中使用Lambda表达式 

通常情况下，Lambda表达式与LINQ查询一起使用。枚举静态类包括接受Func <TSource，bool>的IEnumerable <T>的Where扩展方法。IEnumerable <Int>集合的Where（）扩展方法需要传递Func <Student，bool>，如下所示：

![img](../image/Linq/1033738-20180113214753019-884792667.png)

现在，您可以将分配给Func委托的lambda表达式传递给方法语法中的Where（）扩展方法，如下所示：

```c#
Func<int, bool> isContains = p =>p.Equals (4);
int[] ints = { 5, 2, 0, 66, 4, 32, 7, 1 };
var result = ints.Where(isContains).ToList();
```

### Lambda表达式要点总结

1. Lambda表达式是一种表示匿名方法的更短的方法。 
2. Lambda表达式语法：parameters =>正文表达式
3. Lambda表达式可以在（）中具有零个或多个参数。 
4. Lambda表达式可以在大括号{}中的正文表达式中有一条或多条语句。 
5. Lambda表达式可以分配给Func，Action或Predicate委托。
6. Lambda表达式可以以类似的方式调用委托。

## Linq操作符

### Select

Select操作符对单个序列或集合中的值进行投影。所谓投影，比如有一个数据集，想用LINQ语法去操作数据集，会写一个LINQ的表达式，表达式会把数据集合中的数据简单的投影到一个变量中，并且可以通过这个变量去筛选数据。

在查询表达式中，select 子句可以指定将在执行查询时产生的值的类型。 该子句的结果将基于前面所有子句的计算结果以及 select 子句本身中的所有表达式。 查询表达式必须以 select 子句或 group 子句结束。

```c#
public class Employees
{
        public Guid Id { get; set; }
        public string Name { get; set; }
        public int Sex { get; set; }
        public string CompanyName { get; set; }
}
```

```c#
class Program
{
        static void Main(string[] args)
        {
            //使用集合初始化器给集合赋值
            List<Employees> emp = new List<Employees> 
            { 
               new Employees(){Id=Guid.NewGuid(),Name="张三",Sex=0,CompanyName="xx技术有限公司"},
               new Employees(){Id=Guid.NewGuid(),Name="李四",Sex=0,CompanyName="xx培训"},
               new Employees(){Id=Guid.NewGuid(),Name="王五",Sex=0,CompanyName="xx集团"}
            };

            //查询语法：不能省略最后的select
            var query = (from p in emp where p.Name.StartsWith("王") select p).FirstOrDefault();

            //查询方法:设计到Lambda表达式，全部返回 可以省略最后的select 延迟加载
            var query1 = emp.Where(p => p.Name.StartsWith("王")).Select(e => new { e.Name,e.CompanyName});

            //查询方法:返回匿名类
            var query2 = emp.Where(p => p.Name.StartsWith("王")).Select(p => p);
            foreach (var item in query1)
            {
                Console.WriteLine(item.Name);
            }
            Console.ReadKey();
        }
}
```

Select操作包括7种形式，分别为简单用法、匿名类型形式、条件形式、筛选形式、嵌套类型形式、本地方法调用形式、Distinct形式。下面分别用实例举例下：

```c#
class Student
{
    public string Name { get; set; }
    public int Score { get; set; }
}
List<Student> students = new List<Student>{
    new Student {Name="Terry", Score=50},
    new Student {Name="AI", Score=80},
    new Student {Name="AI", Score=70},
};
```



#### 简单用法

说明:当以select结尾时表示的只是一个声明或者一个描述，并没有真正把数据取出来，只有当你需要该数据的时候，它才会执行这个语句，这就是延迟加载(deferred loading)。

查询学生的姓名：

```c#
 var query = from student in students
                        select student.Name;
foreach (var student in query)
{
    Console.WriteLine("{0}", student);
    //Terry
    //AI
    //AI
}
```

#### **匿名类型形式**

说明：其实质是编译器根据我们自定义产生一个匿名的类来帮助我们实现临时变量的储存。例如 var ob = new {Name = "Harry"}，编译器自动产生一个有property叫做Name的匿名类，然后按这个类型分配内存，并初始化对象。

查询学生的姓名：

```c#
var query = from student in students
                        select new
                        {
                            newName = "学生姓名：" + student.Name
                        };
foreach (var student in query)
{
    Console.WriteLine(student.newName);
    //学生姓名：Terry
    //学生姓名：AI
    //学生姓名：AI
}
```

#### **条件形式**

说明：三元运算，类似于SQL语句case when condition then else的用法。

查询学生的分数等级：

```c#
var query = from student in students
                        select new
                        {
                            student.Name,
                            level = student.Score < 60 ? "不及格" : "合格"
                        };
foreach (var student in query)
{
    Console.WriteLine("{0}:{1}", student.Name, student.level);
    //Terry:不及格
    //AI:及格
    //AI:及格
}

```

#### **筛选形式**

说明：结合where用起到过滤的作用。

查询Terry的分数：

```c#
var query = from student in students
                        where student.Name == "Terry"
                        select student;
foreach (var student in query)
{
    Console.WriteLine("{0}:{1}",student.Name,student.Score);
    //Terry:50
}
```

#### **嵌套类型形式**

说明：如果一个数据源里面又包含了一个或多个集合列表，那么应该使用复合的select子句来进行查询。

查询大于80分的学生分数：

```c#
class Student
{
    public string Name { get; set; }
    public List<int> Scores { get; set; }
}
List<Student> students = new List<Student>{
    new Student {Name="Terry", Scores=new List<int> {97, 72, 81, 60}},
    new Student {Name="AI", Scores=new List<int> {75, 84, 91, 39}},
    new Student {Name="Wade", Scores=new List<int> {88, 94, 65, 85}},
    new Student {Name="Tracy", Scores=new List<int>{97, 89, 85, 82}},
    new Student {Name="Kobe", Scores=new List<int> {35, 72, 91, 70}} 
};
var query = from student in students
    select new
{
    student.Name,
    //生成新的集合对象
    highScore=from sc in student.Scores
        where sc>80
        select sc
};
foreach (var student in query)
{
    Console.Write("{0}:",student.Name);
    foreach (var scores in student.highScore)
    {
        Console.Write("{0},",scores);
    }
    Console.WriteLine();
    //Terry:97,81,
    //AI:84,91,
    //Wade:88,94,85,
    //Tracy:97,89,85,82,
    //Kobe:91,
}

```

#### **本地方法调用形式**

说明：调用自定义方法。

```c#
 var query = from student in students
                        select new
                        {
                            student.Name,
                            //调用GetLevel方法
                            level = GetLevel(student.Score)
                        };
foreach (var student in query)
{
    Console.WriteLine("{0}:{1}", student.Name, student.level);
    //Terry:不及格
    //AI:及格
    //AI:及格
}

protected static string GetLevel(int score)
{
    if (score > 60)
    {
        return "及格";
    }
    else
    {
        return "不及格";
    }
}

```

#### **Distinct形式**

说明：用于查询不重复的结果集。类似于SQL语句SELECT DISTINCT 。

查询不重复的学生姓名：

```c#
var query = (from student in students
                         select student.Name).Distinct();
foreach (var student in query)
{
    Console.WriteLine("{0}", student);
    //Terry:
    //AI
}
```

### SelectMany

SelectMany操作符提供了将多个from子句组合起来的功能，相当于数据库中的多表连接查询，它将每个对象的结果合并成单个序列。

示例：

student类：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace SelectMany操作符
{
    /// <summary>
    /// 学生类
    /// </summary>
    public class Student
    {
        //姓名
        public string Name { get; set; }
        //成绩
        public int Score { get; set; }
        //构造函数
        public Student(string name, int score)
        {
            this.Name = name;
            this.Score = score;
        }
    }
}
```

teacher类：

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace SelectMany操作符
{
    /// <summary>
    /// Teacher类
    /// </summary>
    public class Teacher
    {
        //姓名
        public string Name { get; set; }
        //学生集合
        public List<Student> Students { get; set; }

        public Teacher(string name, List<Student> students)
        {
            this.Name = name;
            this.Students = students;
        }
    }
}
```

Program类

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace SelectMany操作符
{
    class Program
    {
        static void Main(string[] args)
        {
            //使用集合初始化器初始化Teacher集合
            List<Teacher> teachers = new List<Teacher> {
               new Teacher("徐老师",
               new List<Student>(){
                 new Student("宋江",80),
                new Student("卢俊义",95),
                new Student("朱武",45)
               }
               ),
                new Teacher("姜老师",
               new List<Student>(){
                 new Student("林冲",90),
                new Student("花荣",85),
                new Student("柴进",58)
               }
               ),
                new Teacher("樊老师",
               new List<Student>(){
                 new Student("关胜",100),
                new Student("阮小七",70),
                new Student("时迁",30)
               }
               )
            };

            //问题：查询Score小于60的学生
            //方法1：循环遍历、会有性能的损失
            foreach (Teacher t in teachers)
            {
                foreach (Student s in t.Students)
                {
                    if (s.Score < 60)
                    {
                        Console.WriteLine("姓名:" + s.Name + ",成绩:"+s.Score);
                    }
                }
            }

            //查询表达式
            //方法2：使用SelectMany  延迟加载：在不需要数据的时候，就不执行调用数据，能减轻程序和数据库的交互，可以提供程序的性能，执行循环的时候才去访问数据库取数据
            //直接返回学生的数据
            var query = from t in teachers
                        from s in t.Students
                        where s.Score < 60
                        select s;
            foreach (var item in query)
            {
                Console.WriteLine("姓名:" + item.Name + ",成绩:"+item.Score);
            }
            //只返回老师的数据
            var query1 = from t in teachers
                         from s in t.Students
                         where s.Score < 60
                         select new {
                            t,
                            teacherName=t.Name,
                            student=t.Students.Where(p=>p.Score<60).ToList()
                         };
            foreach (var item in query1)
            {
                Console.WriteLine("老师姓名:" + item.teacherName + ",学生姓名:" +item.student.FirstOrDefault().Name+ ",成绩:" + item.student.FirstOrDefault().Score);
            }
            // 使用匿名类 返回老师和学生的数据
            var query2 = from t in teachers
                         from s in t.Students
                         where s.Score < 60
                         select new { teacherName=t.Name, studentName=s.Name,studentScore=s.Score };
            foreach (var item in query2)
            {
                Console.WriteLine("老师姓名:" + item.teacherName + ",学生姓名:" + item.studentName + ",成绩:" + item.studentScore);
            }

            //使用查询方法
            var query3 = teachers.SelectMany(p => p.Students.Where(t=>t.Score<60).ToList());
            foreach (var item in query3)
            {
                Console.WriteLine("姓名:" + item.Name + ",成绩:" + item.Score);
            }
            Console.ReadKey();

        }
    }
}
```

