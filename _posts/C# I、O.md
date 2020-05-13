# C# I、O

1. Read只能读取一个字符，ReadLine可以读取一个字符串
2. Console.readKey()
3. Console.WriteLine()

```C#
using System;

namespace Study
{
    class Program
    {
        static void Main(string[] args)
        {
            int a = 1;
            int b = 2;
            int c = 3;
            Console.WriteLine("{0} {1} {2}", a, b, c);
            string name = "hou";
            int age = 25;
            string rs2 = string.Format("name is {0}, age is {1}.", name, age);
            string rs3 = $"name is {name},age is {age}";
            Console.WriteLine("name is {0}, age is {1}.", name, age);
            Console.WriteLine(rs2);
            Console.WriteLine(rs3);
            Console.ReadKey();
        }
    }
}

```

