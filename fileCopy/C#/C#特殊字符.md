# C# 特殊字符

特殊字符是预定义的上下文字符，用于修改最前面插入了此类字符的程序元素（文本字符串、标识符或属性名称）。 C# 支持以下特殊字符：

- [@](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/tokens/verbatim)：逐字字符串标识符字符。
- [$](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/tokens/interpolated)：内插的字符串字符。

`$` 特殊字符将字符串文本标识为内插字符串 。 内插字符串是可能包含内插表达式的字符串文本 。 将内插字符串解析为结果字符串时，带有内插表达式的项会替换为表达式结果的字符串表示形式。 从 C# 6 开始可以使用此功能。

与使用[字符串复合格式设置](https://docs.microsoft.com/zh-cn/dotnet/standard/base-types/composite-formatting)功能创建格式化字符串相比，字符串内插提供的语法更具可读性，且更加方便。 下面的示例使用了这两种功能生成同样的输出结果：

```c#
string name = "Mark";
var date = DateTime.Now;

// Composite formatting:
Console.WriteLine("Hello, {0}! Today is {1}, it's {2:HH:mm} now.", name, date.DayOfWeek, date);
// String interpolation:
Console.WriteLine($"Hello, {name}! Today is {date.DayOfWeek}, it's {date:HH:mm} now.");
// Both calls produce the same output that is similar to:
// Hello, Mark! Today is Wednesday, it's 19:40 now.
```

## 内插字符串的结构

若要将字符串标识为内插字符串，可在该字符串前面加上 `$` 符号。 字符串文本开头的 `$` 和 `"` 之间不能有任何空格。

具备内插表达式的项的结构如下所示：

```c#
{<interpolationExpression>[,<alignment>][:<formatString>]}
```

括号中的元素是可选的。 下表说明了每个元素：

| 元素                      | 描述                                                         |
| :------------------------ | :----------------------------------------------------------- |
| `interpolationExpression` | 生成需要设置格式的结果的表达式。 `null` 的字符串表示形式为 [String.Empty](https://docs.microsoft.com/zh-cn/dotnet/api/system.string.empty)。 |
| `alignment`               | 常数表达式，它的值定义表达式结果的字符串表示形式中的最小字符数。 如果值为正，则字符串表示形式为右对齐；如果值为负，则为左对齐。 有关详细信息，请参阅[对齐组件](https://docs.microsoft.com/zh-cn/dotnet/standard/base-types/composite-formatting#alignment-component)。 |
| `formatString`            | 受表达式结果类型支持的格式字符串。 有关更多信息，请参阅[格式字符串组件](https://docs.microsoft.com/zh-cn/dotnet/standard/base-types/composite-formatting#format-string-component)。 |

以下示例使用上述可选的格式设置组件：

```c#
Console.WriteLine($"|{"Left",-7}|{"Right",7}|");

const int FieldWidthRightAligned = 20;
Console.WriteLine($"{Math.PI,FieldWidthRightAligned} - default formatting of the pi number");
Console.WriteLine($"{Math.PI,FieldWidthRightAligned:F3} - display only three decimal digits of the pi number");
// Expected output is:
// |Left   |  Right|
//     3.14159265358979 - default formatting of the pi number
//                3.142 - display only three decimal digits of the pi number
```

从 C# 10 开始，当用于占位符的所有表达式也是常量字符串时，可以使用字符串内插来初始化常量字符串。 换言之，每个 interpolationexpression 都必须是一个字符串，并且必须是编译时常量。

## 特殊字符

要在内插字符串生成的文本中包含大括号 "{" 或 "}"，请使用两个大括号，即 "{{" 或 "}}"。 有关详细信息，请参阅[转义大括号](https://docs.microsoft.com/zh-cn/dotnet/standard/base-types/composite-formatting#escaping-braces)。

因为冒号（“:”）在内插表达式项中具有特殊含义，为了在内插表达式中使用[条件运算符](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/operators/conditional-operator)，请将表达式放在括号内。

以下示例演示如何将大括号含入结果字符串中，以及如何在内插表达式中使用条件运算符：

```c#
string name = "Horace";
int age = 34;
Console.WriteLine($"He asked, \"Is your name {name}?\", but didn't wait for a reply :-{{");
Console.WriteLine($"{name} is {age} year{(age == 1 ? "" : "s")} old.");
// Expected output is:
// He asked, "Is your name Horace?", but didn't wait for a reply :-{
// Horace is 34 years old.
```

内插逐字字符串以 `$` 字符开头，后跟 `@` 字符。 有关逐字字符串的详细信息，请参阅[字符串](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/builtin-types/reference-types)和[逐字标识符](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/tokens/verbatim)主题。

## 隐式转换和指定 `IFormatProvider` 实现的方式

内插字符串有 3 种隐式转换：

1. 将内插字符串转换为 [String](https://docs.microsoft.com/zh-cn/dotnet/api/system.string) 实例，该类例是内插字符串的解析结果，其中内插表达式项被替换为结果的格式设置正确的字符串表示形式。 此转换使用 [CurrentCulture](https://docs.microsoft.com/zh-cn/dotnet/api/system.globalization.cultureinfo.currentculture#System_Globalization_CultureInfo_CurrentCulture) 设置表达式结果的格式。

2. 将内插字符串转换为表示复合格式字符串的 [FormattableString](https://docs.microsoft.com/zh-cn/dotnet/api/system.formattablestring) 实例，同时也将表达式结果格式化。 这允许通过单个 [FormattableString](https://docs.microsoft.com/zh-cn/dotnet/api/system.formattablestring) 实例创建多个包含区域性特定内容的结果字符串。 要执行此操作，请调用以下方法之一：

   - [ToString()](https://docs.microsoft.com/zh-cn/dotnet/api/system.formattablestring.tostring#System_FormattableString_ToString) 重载，生成 [CurrentCulture](https://docs.microsoft.com/zh-cn/dotnet/api/system.globalization.cultureinfo.currentculture#System_Globalization_CultureInfo_CurrentCulture) 的结果字符串。
   - [Invariant](https://docs.microsoft.com/zh-cn/dotnet/api/system.formattablestring.invariant) 方法，生成 [InvariantCulture](https://docs.microsoft.com/zh-cn/dotnet/api/system.globalization.cultureinfo.invariantculture#System_Globalization_CultureInfo_InvariantCulture) 的结果字符串。
   - [ToString(IFormatProvider)](https://docs.microsoft.com/zh-cn/dotnet/api/system.formattablestring.tostring#System_FormattableString_ToString_System_IFormatProvider_) 方法，生成特定区域性的结果字符串。

   还可以使用 [ToString(IFormatProvider)](https://docs.microsoft.com/zh-cn/dotnet/api/system.formattablestring.tostring#System_FormattableString_ToString_System_IFormatProvider_) 方法，以提供支持自定义格式设置的 [IFormatProvider](https://docs.microsoft.com/zh-cn/dotnet/api/system.iformatprovider) 接口的用户定义实现。 有关详细信息，请参阅[在 .NET 中设置类型格式](https://docs.microsoft.com/zh-cn/dotnet/standard/base-types/formatting-types)一文中的[使用 ICustomFormatter 进行自定义格式设置](https://docs.microsoft.com/zh-cn/dotnet/standard/base-types/formatting-types#custom-formatting-with-icustomformatter)部分。

3. 将内插字符串转换为 [IFormattable](https://docs.microsoft.com/zh-cn/dotnet/api/system.iformattable) 实例，使用此实例也可通过单个 [IFormattable](https://docs.microsoft.com/zh-cn/dotnet/api/system.iformattable) 实例创建多个包含区域性特定内容的结果字符串。

以下示例通过隐式转换为 [FormattableString](https://docs.microsoft.com/zh-cn/dotnet/api/system.formattablestring) 来创建特定于区域性的结果字符串：

```c#
double speedOfLight = 299792.458;
FormattableString message = $"The speed of light is {speedOfLight:N3} km/s.";

System.Globalization.CultureInfo.CurrentCulture = System.Globalization.CultureInfo.GetCultureInfo("nl-NL");
string messageInCurrentCulture = message.ToString();

var specificCulture = System.Globalization.CultureInfo.GetCultureInfo("en-IN");
string messageInSpecificCulture = message.ToString(specificCulture);

string messageInInvariantCulture = FormattableString.Invariant(message);

Console.WriteLine($"{System.Globalization.CultureInfo.CurrentCulture,-10} {messageInCurrentCulture}");
Console.WriteLine($"{specificCulture,-10} {messageInSpecificCulture}");
Console.WriteLine($"{"Invariant",-10} {messageInInvariantCulture}");
// Expected output is:
// nl-NL      The speed of light is 299.792,458 km/s.
// en-IN      The speed of light is 2,99,792.458 km/s.
// Invariant  The speed of light is 299,792.458 km/s.
```

## 其他资源

如果你不熟悉字符串内插，请参阅 [C# 中的字符串内插](https://docs.microsoft.com/zh-cn/dotnet/csharp/tutorials/exploration/interpolated-strings)交互式教程。 还可查看另一个 [C# 中的字符串内插](https://docs.microsoft.com/zh-cn/dotnet/csharp/tutorials/string-interpolation)教程，该教程演示了如何使用内插字符串生成带格式的字符串。

## 内插字符串编译

如果内插字符串类型为 `string`，则通常将其转换为 [String.Format](https://docs.microsoft.com/zh-cn/dotnet/api/system.string.format) 方法调用。 如果分析的行为等同于串联，则编译器可将 [String.Format](https://docs.microsoft.com/zh-cn/dotnet/api/system.string.format) 替换为 [String.Concat](https://docs.microsoft.com/zh-cn/dotnet/api/system.string.concat)。

如果内插字符串类型为 [IFormattable](https://docs.microsoft.com/zh-cn/dotnet/api/system.iformattable) 或 [FormattableString](https://docs.microsoft.com/zh-cn/dotnet/api/system.formattablestring)，则编译器会生成对 [FormattableStringFactory.Create](https://docs.microsoft.com/zh-cn/dotnet/api/system.runtime.compilerservices.formattablestringfactory.create) 方法的调用。





## `@` 特殊字符用作原义标识符。 它具有以下用途：

1. 使 C# 关键字用作标识符。 `@` 字符可作为代码元素的前缀，编译器将把此代码元素解释为标识符而非 C# 关键字。 下面的示例使用 `@` 字符定义其在 `for` 循环中使用的名为 `for` 的标识符。

   C#复制

   ```csharp
   string[] @for = { "John", "James", "Joan", "Jamie" };
   for (int ctr = 0; ctr < @for.Length; ctr++)
   {
      Console.WriteLine($"Here is your gift, {@for[ctr]}!");
   }
   // The example displays the following output:
   //     Here is your gift, John!
   //     Here is your gift, James!
   //     Here is your gift, Joan!
   //     Here is your gift, Jamie!
   ```

2. 指示将原义解释字符串。 `@` 字符在此实例中定义原义标识符 。 简单转义序列（如代表反斜杠的 `"\\"`）、十六进制转义序列（如代表大写字母 A 的 `"\x0041"`）和 Unicode 转义序列（如代表大写字母 A 的 `"\u0041"`）都将按字面解释。 只有引号转义序列 (`""`) 不会按字面解释；因为它生成一个双引号。 此外，如果是逐字[内插字符串](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/tokens/interpolated)，大括号转义序列（`{{` 和 `}}`）不按字面解释；它们会生成单个大括号字符。 下面的示例分别使用常规字符串和原义字符串定义两个相同的文件路径。 这是原义字符串的较常见用法之一。

   C#复制

   ```csharp
   string filename1 = @"c:\documents\files\u0066.txt";
   string filename2 = "c:\\documents\\files\\u0066.txt";
   
   Console.WriteLine(filename1);
   Console.WriteLine(filename2);
   // The example displays the following output:
   //     c:\documents\files\u0066.txt
   //     c:\documents\files\u0066.txt
   ```

   下面的示例演示定义包含相同字符序列的常规字符串和原义字符串的效果。

   C#复制

   ```csharp
   string s1 = "He said, \"This is the last \u0063hance\x0021\"";
   string s2 = @"He said, ""This is the last \u0063hance\x0021""";
   
   Console.WriteLine(s1);
   Console.WriteLine(s2);
   // The example displays the following output:
   //     He said, "This is the last chance!"
   //     He said, "This is the last \u0063hance\x0021"
   ```

3. 使编译器在命名冲突的情况下区分两种属性。 属性是派生自 [Attribute](https://docs.microsoft.com/zh-cn/dotnet/api/system.attribute) 的类。 其类型名称通常包含后缀 **Attribute**，但编译器不会强制进行此转换。 随后可在代码中按其完整类型名称（例如 `[InfoAttribute]`）或短名称（例如 `[Info]`）引用此属性。 但是，如果两个短名称相同，并且一个类型名称包含 **Attribute** 后缀而另一类型名称不包含，则会出现命名冲突。 例如，由于编译器无法确定将 `Info` 还是 `InfoAttribute` 属性应用于 `Example` 类，因此下面的代码无法编译。 有关详细信息，请参阅 [CS1614](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/compiler-messages/cs1614)。

   C#复制

   ```csharp
   using System;
   
   [AttributeUsage(AttributeTargets.Class)]
   public class Info : Attribute
   {
      private string information;
   
      public Info(string info)
      {
         information = info;
      }
   }
   
   [AttributeUsage(AttributeTargets.Method)]
   public class InfoAttribute : Attribute
   {
      private string information;
   
      public InfoAttribute(string info)
      {
         information = info;
      }
   }
   
   [Info("A simple executable.")] // Generates compiler error CS1614. Ambiguous Info and InfoAttribute.
   // Prepend '@' to select 'Info' ([@Info("A simple executable.")]). Specify the full name 'InfoAttribute' to select it.
   public class Example
   {
      [InfoAttribute("The entry point.")]
      public static void Main()
      {
      }
   }
   ```
