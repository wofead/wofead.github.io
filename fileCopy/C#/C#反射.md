# C#反射

[toc]

## 什么是反射

Reflection，反射。这是.Net回去运行时类型信息的方式，.Net的应用程序有几个部分构成：程序集（Assembly）、模块（Module）和类型（class）构成，而反射提供一种编程的方式，让程序员可以在程序运行期间获取这几个组成部分的相关信息。例如：

Assembly类可以获得正在运行的装配件信息，也可以动态的加载装配件，以及在装配件中查找类型信息，创建该类型的实例。

Type类可以获得对象的类型信息，此信息包含对象的所有要素：方法、构造器、属性等等，通过Type类可以得到这些要素的信息，并且调用。

MethodInfo包含方法的信息，通过这个类可以得到方法的名称、参数以及返回值等，并且可以调用这个方法。

诸如此类，还有FieldInfo、EventInfo等等，这些类都包含在System.Reflection命名空间下。

## 命名空间与装配件的关系

装配件是.Net应用程序执行的最小单位，编译出来的.dll、.exe都是装配件。

命名空间就是规定这个类属于那个族群，而装配件这是说明这个族群住在那里。这说明，如果要使用一个类，必须告诉编译器这个类在那个装配件中，然后编译器去加载这个装配件，而明明命名空间是指定一个类在那一块，其实这一块已经加载了，建立索引就ok了、

## 运行期得到类型信息有什么用

就和虚函数一样，运行的时候再决定究竟执行那个函数，反射也是这样的。

## 如何用反射获取类型

获得类型信息有两种方法，一种是得到实例对象，另外一种获取类型的方法是通过Type.GetType以及Assembly.GetType方法。

第一种，我们拿到的是这个实例对象，也许是一个对象的引用，也有可能是一个接口的引用，如果需要了解其确切类型，我们可以通过调用`System.Object`的`GetType`来获取实例对象的类型对象，

第二种，我们可以使用`Type t = Type.GetType("System.String");`

需要注意的是，前面我们讲到了命名空间和装配件的关系，要查找一个类，必须执行它所在的装配件，或者在已经获得的Assembly实例上调用GetType。本装配件中类型可以只写类型名称，另一个例外是mscorlib.dll，这个装配件中声明的类型也可以省略装配件名称（.Net装配件编译的时候，默认都引用了mscorlib.dll，除非在编译的时候明确指定不引用它）。

System.String是在mscorlib.dll中声明的，上面的Type t = Type.GetType(“System.String”)是正确的。

System.Data.DataTable是在System.Data.dll中声明的，那么：Type.GetType(“System.Data.DataTable”)就只能得到空引用。

必须：

```c#
Type  t  =  Type.GetType("System.Data.DataTable,System.Data,Version=1.0.3300.0,  Culture=neutral,  PublicKeyToken=b77a5c561934e089");
```

## 如何根据类型来动态创建对象

1.  System.Activator提供了方法来根据类型动态创建对象，比如创建一个DataTable

```c#
Type  t  =  Type.GetType("System.Data.DataTable,System.Data,Version=1.0.3300.0,  Culture=neutral,  PublicKeyToken=b77a5c561934e089");
DataTable  table  =  (DataTable)Activator.CreateInstance(t);
namespace  TestSpace  
{
  public  class  TestClass
      {
      private  string  _value;
      public  TestClass(string  value)  
    {
      _value=value;
      }
  }
}
Type  t  =  Type.GetType(“TestSpace.TestClass”);
Object[]  constructParms  =  new  object[]  {“hello”};  //构造器参数
TestClass  obj  =  (TestClass)Activator.CreateInstance(t,constructParms);

```

## 获取方法以及动态调用方法

```c#
namespace  TestSpace
{
      public  class  TestClass  {
          private  string  _value;
          public  TestClass()  {
          }
          public  TestClass(string  value)  {
                _value  =  value;
          }
          public  string  GetValue(  string  prefix  )  {
              if(  _value==null  )
              return  "NULL";
          else
            return  prefix+"  :  "+_value;
          }
          public  string  Value  {
              set  {
                  _value=value;
              }
              get  {
                  if(  _value==null  )
                      return  "NULL";
                  else
                      return  _value;
              }
          }
      }
}
```

上面是一个简单的类，包含一个有参数的构造器，一个GetValue方法，一个Value属性：

```c#
//获取类型信息
Type  t  =  Type.GetType("TestSpace.TestClass");
//构造器的参数
object[]  constuctParms  =  new  object[]{"timmy"};
//根据类型创建对象
object  dObj  =  Activator.CreateInstance(t,constuctParms);
//获取方法的信息
MethodInfo  method  =  t.GetMethod("GetValue");
//调用方法的一些标志位，这里的含义是Public并且是实例方法，这也是默认的值
BindingFlags  flag  =  BindingFlags.Public  |  BindingFlags.Instance;
//GetValue方法的参数
object[]  parameters  =  new  object[]{"Hello"};
//调用方法，用一个object接收返回值
object  returnValue  =  method.Invoke(dObj,flag,Type.DefaultBinder,parameters,null);
```

## 动态创建委托

委托是C#中实现事件的基础，有时候不可避免的要动态的创建委托，实际上委托也是一种类型：System.Delegate，所有的委托都是从这个类派生的
    System.Delegate提供了一些静态方法来动态创建一个委托，比如一个委托：

```c#
namespace  TestSpace  {
    delegate  string  TestDelegate(string  value);
    public  class  TestClass  {
        public TestClass(){}
        public  string  GetValue(string  value)  {
            return  value;
        }
    }
}

TestClass  obj  =  new  TestClass();
//获取类型，实际上这里也可以直接用typeof来获取类型
Type  t  =  Type.GetType(“TestSpace.TestDelegate”);
//创建代理，传入类型、创建代理的对象以及方法名称
TestDelegate  method  =  (TestDelegate)Delegate.CreateDelegate(t,obj,”GetValue”);
String  returnValue  =  method(“hello”);

```

## 反射（Reflection）的命名空间

- System.Reflection.Assembly 
- System.Reflection.MemberInfo
- System.Reflection.EventInfo
- System.Reflection.FieldInfo
- System.Reflection.MethodBase
- System.Reflection.ConstructorInfo
- System.Reflection.MethodInfo
- System.Reflection.PropertyInfo
- System.Type

一下是使用放方法：

**Assembly**定义和加载程序集，加载在程序集清单中列出模块，以及从此程序集中查找类型并创建该类型的实例。

**Module**包含模块的程序集以及模块中的类等，还可以获取在模块上定义的所有全局方法或其他特定的非全局方法。

**ConstructorInfo**了解构造函数的名称、参数、访问修饰符合实现的详细信息（eg：abstract或virtual）等。使用Type的GetMethods或GetMethod方法调用特定改的方法。

**MethodInfo**了解方法的名称、返回类型、参数、访问修饰符和实现详细信息等。使用Type的GetMethods或GetMethod方法调用特定改的方法。

**FieldInfo**了解字段名称、访问修饰符和实现详细信息等，并获取或设置字段值。

**EventInfo**来了解事件名称、事件处理程序数据类型、自定义属性、声明类型和反射类型等，添加或移除事件处理程序。

**PropertyInfo**来了解属性的名称、数据类型、声明类型、反射类型和只读或可写状态等，获取或设置属性值。

**ParameterInfo**来了解参数的名称、数据类型、是输入参数还是输出参数以及参数在方法签名中的位置等。

**反射的层次模型**

1. 程序集
2. 模块
3. 类型：可以是类、委托、枚举或者结构
4. 事件、字段、特性、方法

## 反射的作用

1. 可以使用反射动态地创建类型的实例，将类型绑定到现有对象，或从现有对象中获取类型。
2. 应用程序需要在运行时从某个特定的程序集中载入一个特定的类型，以便实现某个任务时可以用到反射。
3. 反射主要应用与类库，这些类库需要知道一个类型的定义，以便提供更多的功**能。**

**反射appDomain的程序集**

```c#
static void Main
{
       //通过GetAssemblies 调用appDomain的所有程序集
       foreach (Assembly assem in Appdomain.currentDomain.GetAssemblies())
      {
       //反射当前程序集的信息
            reflector.ReflectOnAssembly(assem)
      }
}
```



调用AppDomain对象的GetAssemblies方法将返回一个有System.Reflection.Assembly元素组成的数组。

**反射单个程序集**

我们可以使用ystem.Reflection.Assembly类型提供的三个方法：

1. Load方法：极力推荐的一种方法，Load 方法带有一个程序集标志并载入它，Load 将引起CLR把策略应用到程序集上，先后在全局程序集缓冲区，应用程序基目录和私有路径下面查找该程序集，如果找不到该程序集系统抛出异常。
2. LoadFrom方法：传递一个程序集文件的路径名（包括扩展名），CLR会载入您指定的这个程序集，传递的这个参数不能包含任何关于版本号的信息，区域性，和公钥信息，如果在指定路径找不到程序集抛出异常。
3. LoadWithPartialName：永远不要使用这个方法，因为应用程序不能确定再在载入的程序集的版本。该方法的唯一用途是帮助那些在.Net框架的测试环节使用.net 框架提供的某种行为的客户，这个方法将最终被抛弃不用。

**注意**：System.AppDomain也提供了一种Load方法，他和Assembly的静态Load方法不一样，它是一个实例方法，返回的是对一个程序集的引用，Assembly的静态Load方法将程序集按值封装发送给调用的AppDomain.应尽量避免使用AppDomain的load方法。

## 例子

接下来用一个不完整的例子，类型反射一个简单利用反射获取类型信息的例子：

```c#
using system;
using sytem.reflection;
class reflecting 
{
       static void Main(string[]args)
       {
             reflecting reflect=new reflecting();//定义一个新的自身类
             //调用一个reflecting.exe程序集

             assembly myAssembly =assembly.loadfrom(“reflecting.exe”)
             reflect.getreflectioninfo(myAssembly);//获取反射信息
       }

       //定义一个获取反射内容的方法
       void getreflectioninfo(assembly myassembly)
       {
             type[] typearr=myassemby.Gettypes();//获取类型
             foreach (type type in typearr)//针对每个类型获取详细信息
            {
                   //获取类型的结构信息
                  constructorinfo[] myconstructors=type.GetConstructors;

                 //获取类型的字段信息
                 fieldinfo[] myfields=type.GetFiedls()

                 //获取方法信息
                 MethodInfo   myMethodInfo=type.GetMethods();

                 //获取属性信息
                 propertyInfo[] myproperties=type.GetProperties

                 //获取事件信息
                 EventInfo[] Myevents=type.GetEvents;
           }
      }
}
```

其它集中获取Type对象的方法：

1. System.type  参数为字符串类型，该字符串必须指定类型的完整名称（包括其命名空间）
2. System.type 提供了两个实例方法：GetNestedType,GetNestedTypes
3. Syetem.Reflection.Assembly 类型提供的实例方法是：GetType,GetTypes,GetExporedTypes
4. System.Reflection.Moudle 提供了这些实例方法：GetType,GetTypes,FindTypes

**设置反射类型的成员**

反射类型的成员就是反射层次模型中最下面的一层数据。我们可以通过type对象的GemMembers方法取得一个类型的成员。如果我们使用的是不带参数的GetMembers，它只返回该类型的公共定义的静态变量和实例成员，我们也可以通过使用带参数的GetMembers通过参数设置来返回指定的类型成员。

```c#
//设置需要返回的类型的成员内容
bindingFlags bf=bingdingFlags.DeclaredOnly|bingdingFlags.Nonpublic|BingdingFlags.Public;
foreach (MemberInfo mi int t.getmembers(bf))
{
       writeline(mi.membertype)    //输出指定的类型成员
}
```

**通过反射创建类型的实例**

通过反射可以获取程序集的类型，我们就可以根据获得的程序集类型来创建该类型新的实例，这也是前面提到的在运行时创建对象实现晚绑定的功能。

1. System.Activator的CreateInstance方法。该方法返回新对象的引用。
2. System.Activator的CreateInstanceFrom与上一个方法类似，不过需要指定类型及其程序集。
3. System.Appdomain的方法：CreateInstance，CreateInstanceAndUnWrap，CreateInstanceFrom和CreateInstanceFromAndUnwrap
4. System.type的InvokeMember实例方法：这个方法返回一个与传入参数相符的构造函数，并构造该类型。
5. System.Reflection.ConstructionInfo的Invoke实例方法。

**反射类型的接口**

如果你想要获得一个类型继承的所有接口集合，可以调用Type的FindInterfaces GetInterface或者GetInterfaces。所有这些方法只能返回该类型直接继承的接口，他们不会返回从一个接口继承下来的接口。要想返回接口的基础接口必须再次调用上述方法。

**反射的性能**

使用反射来调用类型或者触发方法，或者访问一个字段或者属性时clr 需要做更多的工作：校验参数，检查权限等等，所以速度是非常慢的。所以尽量不要使用反射进行编程，对于打算编写一个动态构造类型（晚绑定）的应用程序，可以采取以下的几种方式进行代替：

1. 通过类的继承关系，让该类型从一个编译时可知的基础类型派生出来，在运行时生成该类型的一个实例，将对其的引用放到基础类型的一个变量中，然后调用该基础类型的虚方法。
2. 通过接口实现。在运行时，构建该类型的一个实例，将对其的引用放到其接口类型的一个变量中，然后调用该接口顶一个虚方法。
3. 通过委托实现。让该类型实现一个方法，其名称和原型都与一个在编译时就已知的委托相符。在运行时先构造该类型的实例，然后在用该方法的对象及名称构造出该委托的实例，接着通过委托调用你想要的方法。

源DLL类：

```c#
using System;
using System.Collections.Generic;
using System.Text;
using System.Text.RegularExpressions;
using System.Web.UI;
using System.Collections;


namespace cn.SwordYang
{

    public class TextClass:System.Web.UI.Page
    {

		public static void RunJs(Page _page, string Source)
        {
            _page.ClientScript.RegisterStartupScript(_page.GetType(), "", "<script type=\"text/javascript\">" + Source + ";		</script>");

        }

	}

}
```

调用代码：

```c#
System.Reflection.Assembly ass = Assembly.LoadFrom(Server.MapPath("bin/swordyang.dll")); //加载DLL
System.Type t = ass.GetType("cn.SwordYang.TextClass");//获得类型
object o = System.Activator.CreateInstance(t);//创建实例

System.Reflection.MethodInfo mi = t.GetMethod("RunJs");//获得方法


mi.Invoke(o, new object[] { this.Page,"alert('测试反射机制')"});//调用方法
```

