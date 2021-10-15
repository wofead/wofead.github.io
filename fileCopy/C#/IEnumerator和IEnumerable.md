# 	IEnumerator和IEnumerable

IEnumerator是枚举器的意思，IEnumerable是可枚举的意思。

[toc]

## IEnumerator和IEnumerable的定义

**IEnumerator**

```c#
public interface IEnumerator
{
    // Interfaces are not serializable
    // Advances the enumerator to the next element of the enumeration and
    // returns a boolean indicating whether an element is available. Upon
    // creation, an enumerator is conceptually positioned before the first
    // element of the enumeration, and the first call to MoveNext 
    // brings the first element of the enumeration into view.
    // 
    bool MoveNext();

    // Returns the current element of the enumeration. The returned value is
    // undefined before the first call to MoveNext and following a
    // call to MoveNext that returned false. Multiple calls to
    // GetCurrent with no intervening calls to MoveNext 
    // will return the same object.
    // 
    Object Current {
    get; 
    }

    // Resets the enumerator to the beginning of the enumeration, starting over.
    // The preferred behavior for Reset is to return the exact same enumeration.
    // This means if you modify the underlying collection then call Reset, your
    // IEnumerator will be invalid, just as it would have been if you had called
    // MoveNext or Current.
    //
    void Reset();
}
```

**IEnumerable**

```c#
public interface IEnumerable
{
    // Interfaces are not serializable
    // Returns an IEnumerator for this enumerable Object.  The enumerator provides
    // a simple way to access all the contents of a collection.
    [Pure]
    [DispId(-4)]
    IEnumerator GetEnumerator();
}
```

法线IEnumerable只有一个GetEnumerator函数，返回值是IEnumerator类型，从注释我们可以得知IEnumerable代表继承此接口的类可以获取一个IEnumerator来实现枚举这个类中包含的集合中的元素的功能（比如`List<T>,ArrayList,Dictionary`等继承了IEnumeratble接口的类）。

## 用foreach来了解IEnumerable，IEnumerator的工作原理

模仿ArrayList来实现一个简单的ConstArrayList，然后用foreach遍历。

```c#
public class ConstArrayList : IEnumerable
{
    public int[] constItems = new int[] { 1, 2, 3, 4, 5 };
    public IEnumerator GetEnumerator()
    {
        Console.WriteLine("GetIEnumerator");
        return new ConstArrayListEnumeratorSimple(this);
    }

}

class ConstArrayListEnumeratorSimple : IEnumerator
{
    ConstArrayList list;
    int index;
    int currentElement;
    public ConstArrayListEnumeratorSimple(ConstArrayList _list)
    {
        list = _list;
        index = -1;
    }

    public object Current
    {
        get
        {
            Console.WriteLine("Current");
            return currentElement;
        }
    }

    public bool MoveNext()
    {
        if (index < list.constItems.Length - 1)
        {
            Console.WriteLine("MoveNext true");
            currentElement = list.constItems[++index];
            return true;
        }
        else
        {
            Console.WriteLine("MoveNext false");
            currentElement = -1;
            return false;
        }
    }

    public void Reset()
    {
        Console.WriteLine("Reset");
        index = -1;
    }
}
 static void Main(string[] args)
 {
     ConstArrayList constArrayList = new ConstArrayList();
     foreach (int item in constArrayList)
     {
         Console.WriteLine(item);
     }
     Console.ReadKey();
 }
//GetIEnumerator
//MoveNext true
//Current
//1
//MoveNext true
//Current
//2
//MoveNext true
//Current
//3
//MoveNext true
//Current
//4
//MoveNext true
//Current
//5
//MoveNext false
```

通过输出结果，可以发现，foreach在运行时先调用ConstArrayList的GetIEnumerator函数，获取一个ConstArrayListEnumeratorSimple，之后调用其中的MoveNext函数，index后更新，更新Current属性，返回Current的值，直到MoveNext返回false。