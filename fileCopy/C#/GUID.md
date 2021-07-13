# C#中`GUID`

`GUID`（全局统一标识符）是指在一台机器上生成的数字，它保证对在同一时空中的所有机器都是唯一的。通常平台会提供生成`GUID`的`API`。生成算法很有意思，用到了以太网卡地址、纳秒级时间、芯片ID码和许多可能的数字。`GUID`的唯一缺陷在于生成的结果串会比较大。

1. 一个`GUID`为一个128位的整数(16字节)，在使用唯一标识符的情况下，你可以在所有计算机和网络之间使用这一整数。

2. `GUID` 的格式为`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`，其中每个 x 是 0-9 或 a-f 范围内的一个十六进制的数字。例如：`337c7f2b-7a34-4f50-9141-bab9e6478cc8` 即为有效的 `GUID` 值。

3. 世界上（`Koffer`注：应该是地球上）的任何两台计算机都不会生成重复的 `GUID` 值。`GUID` 主要用于在拥有多个节点、多台计算机的网络或系统中，分配必须具有唯一性的标识符。

4. 在 Windows 平台上，`GUID` 应用非常广泛：注册表、类及接口标识、数据库、甚至自动生成的机器名、目录名等。

****

**.NET中使用GUID**

GUID 在 .NET 中使用非常广泛，而且 .NET Framework 提供了专门 GUID 基础结构。

GUID 结构的常用法包括：

1. Guid.NewGUID()：生成一个新的 GUID 唯一值
2. Guid.ToString()：将 GUID 值转换成字符串，便于处理
3. 构造函数 Guid(string)：由 string 生成 Guid 结构，其中string 可以为大写，也可以为小写，可以包含两端的定界符“{}”或“()”，甚至可以省略中间的“-”，Guid 结构的构造函数有很多，其它构造用法并不常用。

.NET Framework 中可以使用类 GuidConverter 提供将 Guid 结构与各种其他表示形式相互转换的类型转换器。

在C#中生成一个GUID

处理一个唯一标识符使得存储和获得信息变得更加容易。在处理一个数据库中这一功能变得尤其有用，因为一个GUID能够操作一个主键。

同样，SQL Server也很好地集成了GUID的用途。SQL Server数据类型uniqueidentifier能够存储一个GUID数值。你可以通过使用NEWID()函数在SQL Server中生成这一数值，或者可以在SQL Server之外生成GUID,然后再手动地插入这一数值。

在.NET中，后面一种方法显得更加直接。.NET Framework中的基本System类包括GUID数值类型。除此之外，这一数值类型包含了处理GUID数值的方法。特别地，NewGUID方法允许你很容易地生成一个新的GUID。

```c#
using System; 
namespace DisplayGUID
{
    class Program
    {
        static void Main(string[] args)
        {
            GenerateGUID();
        }
        static void GenerateGUID()
        {
            Console.WriteLine("GUID:" + System.Guid.NewGuid().ToString());
        }
    }
}
```

