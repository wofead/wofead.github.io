# C# this扩展方法

扩展方法被定义为静态方法，但他们是通过实例方法语法进行调用的。它们的第一个参数指定该方法作用于那个类型，并且改参数以this修饰符作为前缀。扩展方法当然不能破坏面向封装的概念，所以只能是访问所扩展类的public成员。

## 给string类型添加一个Add方法，该方法的作用是给字符串增加一个字母a

```c#
 public static  string  Add(this string stringName)
 {
     return stringName+"a";
 }
```

