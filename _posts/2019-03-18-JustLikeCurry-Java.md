---
layout:     post
title:      Java从入门到精通学习
subtitle:  Just like curry.
date:       2019-3-14
author:     Jow
header-img: img/post-bg-2015.jpg
catalog: 	 true 
tags:
    - Java

---

### 目录
1. Object类
2. instanceod 的使用
3. final
4. 内部类
5. 异常的捕捉
6. Swing
7. 集合类
8. I/O
9. 反射
10. 枚举
11. 泛型

> 没有人会莫名其妙的喜欢你和讨厌你，只有你自己喜欢你自己，并且按照自己内心真正正确的道路走下去，这样你才能让自己和别人喜欢。

## Object类
所有的类默认继承Object类
Object包含 getClass()函数 返回对象执行时Class实例
toString()方法 讲对象返回为字符串行形式
equals() “==”比较引用值 即内存地址  equals()方法比较内容是否相同
但是在自定义类中 equals默认比较引用地址 除非重写equals方法让它们比较内容
## instanceod 的使用
判断一个对象是不是有某个类实例化的 特别适用于声明使用父类 实例使用子类
## final
即const变量 声明
**被定义为final的对象引用只能指向唯一一个对象，不可以将它再指向其他对象，但是对象本身的值是可以改变 这个值在对象初始化的赋值 但是使用static 和final一起修饰 这个值在类生成的时候就会初始化 永远不会改变**

final的方法不能被重写
final的类不能被基层，并且不允许其他人对这个类进行任何改动

## 内部类
在一个类里面可以声明另一个类
这个类称为内部类
内部类可以访问外部类的成员各方法
但是外部类不能访问内部类的成员和方法
内部类依赖于外部类
通过接口实例内部类 并访问其中的方法 从而形成对其封装

使用this来访问内部成员和外部成员 内部 ”this.“  外部OutClass.this.  来访问
## 异常的捕捉

try-catch-finally
		
		
		try{
			//程序代码块
		}
		catch(Exceptiontype1 e){
			//对Exceptiontype1的异常处理
		}
		catch(Exceptiontype2 e){
			//对Exceptiontype2的异常处理
		}
		finally{
			//程序块
		}

无论try中的程序代码块如何退出，都将执行finally

自定义异常

继承Exception类就可以自定义异常通过使用throw关键字来抛出异常
在方法后添加 throws 然后当不符合规定的手通过throw来抛出异常

在使用try-catch可以使用自定义异常和已经封装好的异常。
## Swing
Object -> Component - > Container -> JComponent
## 集合类
object  -> Map   Map->HashMap Map->TreeMap
Object-> Collection -> Set  HashSet  TreeSet
Object-> Collection -> List  ArrayList LinkedList

Collection接口包括函数：
1. add()
1. remove()
1. isEmpty()
1. iterator()   迭代器可以输出集合类中的每一个元素
1. size()

**List**
List的中的特有方法：
get(int index)
set(int index,Oject o)

ArrayList可以实现可变的数组，允许保存所有的元素包含null，可以快速根据index隔得元素 缺点插入和删除

LinkedList 链表的结构保存对象 方便插入和删除

**Set**
set中特有的方法
fist() 第一个元素
last() 最后一个元素
comparator  对set中的元素进行排序的比较器
headSet(E toE) 返回一个新的集合 新的集合是toE之前的元素集合 不包含toE
subSet(E h,E t) 返回之间的 包含h，不包含t
tailSet(E fromE) 返回fromE之后的 包含fromE
Set集合中的对象不按特定的方式排序，只是简单地把对象加入到集合中，但Set集合中不能包含重复对象。
包含相同对象的时候前面的会被覆盖
HashSet，由HashMap支持，他不暴增Set的迭代顺序
TreeSet 不仅实现的Set接口，还实现了until.SortedSet接口，因为TreeSet在遍历集合的时候按照自然顺序递增排序。 不允许存在空对象
也可以按照比较器规定排序，comparator()  比较器 存入TreeSet类实现的Set集合必须实现Comparable接口，该接口中的CompareTo(oBject o)
方法比较此2对象与制定对象的顺序，小于负整数 等0 大于正整数


**map**
Map没有继承Collection接口 其提供的是Key到Value的映射。Map中不能包含相同的Key，每个key只能映射一个Value
Map中的方法
put()
containKey()
containValue()
get(Object key)
keySet()  返回该集合中所有对象形成的set集合
values() 返回该集合中所有值对象形成的Collection集合
HashMap效率更高 允许存在null对象
TreeMap还继承了SortedMap接口 所以存储是有序的 不允许null.

## I/O
包括 File 读取文件或者文件夹 可以通过isDictionary来判断是否是文件夹
File file = new File("文件路径")
1. getName
2. canRead
3. canWrite
4. exits
5. length 获取文件的长度
6. getAbsolutePath 绝对路径
7. getPatent 获取文件的父路径
8. isFile 是否是文件
9. isDictionary 是否是目录
10. isHidden 是否为隐藏文件
11. lastModifid 文件最近一次修改时间

### outStream and inputSream
输出流，将信息输出到输出目标中 文件、网络、压缩包、其它数据输出目标中
输入流，将信息从文件、网络等中读取出来放到目的地

InputStream 字节输入流 ，所有字节输入流的父类
子类包括：audio、byteArray、stringBuffer、file、filter、object、sequence、piped
其中filter子类又包括：buffer、data、pushback等
方法：
1. read() 读取下一个自己 返回0~255的int字节值 到达流的末尾 返回-1
2. read(byte[] b) 读取一定长度的字节，整数形式返回字节数
3. mark(int readlimit) 在输入流的当前位置放置一个标记，readlimit参数告知此输入流在标记失效之前允许读取的字节数
4. reset() 讲治镇返回到当前所做的标记处
5. skip(long n) 跳过输入流上的n字节 并返回跳过的字节数
6. MarkSupported 是否支持标记
7. close 关闭流

除了字节输入流还有字符输入流
**Reader** 子类：CharArray、Buffered ->:LineNumber、Filter->pushBack、InputStream-file、piped、string

输出流 outStream
子类： bytearray、file、filter->buffered和data、object、piped
方法：
1. write: 参数类型 int、byte[]、byte[] off len
2. flush() 彻底完成输出并且清空缓存区
3. close

**writer:**子类和reader 相比 多一个print 加上一个file

**file类**
file类唯一代表磁盘文件本身的对象 可以读取文件、文件夹、压缩等一系列的磁盘文件
数据流可以将数据写入到文件中

### 文件的输入输出流
FileInputStream/FileOutputStream 文件字节输入输出流
通过FileOutputStream流将字节数组写进文件中，要先将数据转化为字节数组
通过FileInputStream read到byte[] 数组中 read在将数据写入到数组的同是会将长度返回 然后将字节数组装换位string打印

除了字节输入输出之外还有字符输入和输出
FileReader和Filewriter
可以以字符串的形式写入到文件中，以char[] 的形式从文件中读出

### 带缓冲区的输入和输出
数据信息在进入流之前可以缓存在缓存区当中 每当缓存区满了往流中注入一次数据，避免频繁的I/O操作
文件->inputstream->bufferedinputdtream->target
data->bufferWriter->outputStreamWriter->outputstream->文件

### 数据输入输出流
在文件输入输出流之后可以添加数据输入输出流  然后写入文件和读取文件
### zip输入输出流(自己有时间要再次实现 imp)
ZipOutinputStream
putNextEntry(new Entry(base)) //创建进入节点
进行递归
一直到所有的文件都被压缩
是文件夹 fls = f.listFiles();然后分别压缩
不是文件夹 ： 是文件 先读取 然后写入out中

解压缩 遍历获取 getNextEntry() 是否为空和文件夹 然后通过输入流转换 再通过文件输出流输出到文件中

## 反射
Java 通过反射机制，可以在程序中访问已经装载到JVM中的Java对象的描述
实现访问、检测和修改描述Java对象本身的信息功能。
getClass() 方法，返回一个类型为Class的对象。
1. getPackage()  包名
2. getName()   名字
3. getSuperClass()  父类
4. getInterfaces() 该类实现的所有接口
5. getConstructors()  权限为public的构造函数组
6. getConstructor(Class<?> ...) //Class.int or string ...  根据参数获得构造函数
7. getDeclaredConstructors()  所有构造函数
8. getDeclaredConstructor(Class<?> ...)
9. getMethods()  方法
10. getMethod(Class<?> ...)
11. getDeclaredMethods()
12. 带参数的
13. getFields()  //成员变量
14. getField(String name)
15. getDeclaredFields)
16. getDeclaredField(String name)
17. getClasses()  //内部类 public
18. getDeclaredClasses() //所有
19. getDeclaringClass() //如果该类为内部类，则返回它的成员类，否则返回null
**attention：**
getFields和个人Methods 获得权限为public的，包含父类的
getDeclared 获得只是在本类中的所有

### 反射的获取和执行
#### 1. 访问构造方法
首先获取对象的类 ,然后获取所有声明的构造方法
选择一个构造方法进行实现
其中如果方法为private 需要设置accessible为true 才能调用
如果参数为不确定个 即：...
需要使用二维数组,即二维数组的第一个参数
```java
object[]  {new String[]{"10","20"}}
```		
```java
Clasee<? extends Example> exampleC = example.getClass();
Construct[] constructs = exampleC.getDeclaredConstructors();
Construct construct = constructs[0];
Example example1 = null;
while(example1 == null){
	try{
		example1 = (Example)construct.newInstence();
	}catch(Exception e){
		printStackTrace();
		construct.setAccessible(true);
	}
}
```
**方法中常用的方法**
1. isVarArgs()  是否允许带有可变参数变量
2. getParameterTypes 以Class数组的方式获取改构造函数的各个参数
3. newInstence(Obeject...initargs)//new一个对象
4. setAccessible(boolean flag)
5. getModifiers()  获得可以解析出该构造方法所采用的修饰符整数
6. isPublic(int mod)  参数就是修饰该构造方法的整数
7. isProtected(int mod)
8. isPrivate(int mod)
9. isStatic(int mod)
10. isFinal(int mod)
11. toString(int mod)//以字符串的形式返回所有的修饰符

#### 2.访问成员变量

```java
Field[] fields = exampleC.getDeclaredFields();
Field field = fields[0];
```		
下面是Field类中常用的方法
**在下面的参数obj为实例化出来的对象** 
**被getClass出来的Class并不是对象，而是一个类，类似模板**
**判断一个成员的类型使用 int.class float.class ...**
1. getName()
2. getType()  获得表示该成员变量的名称
3. get(Object obj) //获得指定对象obj中成员变量的值，返回值为Object
4. set(Object obj,Object value)
5. getInt(Obejct obj) 还有Float，Boolean以及setInt...   
6. setAccessible(boolean flag)
7. getModifiers()  修饰符的整数

#### 3.访问方法
```java
Method[] methods = exampleC.getDeclaredMethods();
Mothed mothed = methods[0];
```		
1. geName()
2. getParameterTypes()
3. getRetyrnType()
4. getExceptionTypes()
5. invoke(Object obj,Object...args)
6. isVarArgs()
7. getModifiers()

### Annnotation(注解)
注解在平时使用的时候可能用的不深的时候没什么作用，但是如果进行深层次的应用真的非常重要，所以还请认真学习。
注解类的声明：
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface test{
	String name() default "";
}
```	

@Target来约束注解可以使用的地方：


 	/**标明该注解可以用于类、接口（包括注解类型）或enum声明*/
    TYPE,

    /** 标明该注解可以用于字段(域)声明，包括enum实例 */
    FIELD,

    /** 标明该注解可以用于方法声明 */
    METHOD,

    /** 标明该注解可以用于参数声明 */
    PARAMETER,

    /** 标明注解可以用于构造函数声明 */
    CONSTRUCTOR,

    /** 标明注解可以用于局部变量声明 */
    LOCAL_VARIABLE,

    /** 标明注解可以用于注解声明(应用于另一个注解上)*/
    ANNOTATION_TYPE,

    /** 标明注解可以用于包声明 */
    PACKAGE,

    /**
     * 标明注解可以用于类型参数声明（1.8新加入）
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * 类型使用声明（1.8新加入)
     * @since 1.8
     */
    TYPE_USE
	当注解未指定Target的时候，则此注解可以用于任何元素只上，多个值使用{}包含并用逗号隔开 如下
``` java
@Target(value={ElementType.CONSTRUCTOR, ElementType.FIELD, ElementType.LOCAL_VARIABLE, ElementType.METHOD, ElementType.PACKAGE, ElementType.PARAMETER, ElementType.TYPE})
```
使用@Retention用来约束注解的声明周期，分别有三个值，源码级别（source），类文件级别（class）或者运行时级别（runtime）.
1. SOURCE：注解将被编译器丢弃（该类型的注解信息只会保留在源码里，源码经过编译后，注解信息会被丢弃，不会保留在编译好的class文件里）
2. CLASS：注解在class文件中可被使用，但会被JVM丢弃（该类型的注解信息会保留在源码里和class文件里，在执行的时候，不会加载到虚拟机中），请注意，当注解未定义Retention值时，默认值是CLASS，如Java内置注解，@Override、@Deprecated
3. RUNTIME：注解信息在运行期（JVM）也保留，因此可以通过反射机制读取注解的信息（源码、class文件和执行的时候都有注解的信息），如SpringMvc中的@Controller、@Autowired、@RequestMapping等。

注解支持的元素数据类型
* 所有基本类型（int,float,boolean,byte,double,char,long,short）
* String
* Class
* enum
* Annotation
* 上述类型的数组

```java
package com.zejian.annotationdemo;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface Reference{
    boolean next() default false;
}

public @interface AnnotationElementDemo {
    //枚举类型
    enum Status {FIXED,NORMAL};

    //声明枚举
    Status status() default Status.FIXED;

    //布尔类型
    boolean showSupport() default false;

    //String类型
    String name()default "";

    //class类型
    Class<?> testCase() default Void.class;

    //注解嵌套
    Reference reference() default @Reference(next=true);

    //数组类型
    long[] value();
}
```
**编译器对默认值的限制**
1. 元素不能有不确定的值，要么具有默认值，要么在使用注解时提供元素的值
2. 对于非基本类型的元素，无论是在源代码中声明，还是在注解接口中定义默认值，都不能以null作为值：只能定义一些特殊的值，例如空字符串或负数，表示某个元素不存在

**注解不支持继承** 不能使用关键字extends来继承某个@interface，但注解在编译后，编译器会自动继承java.lang.annotation.Annotation接口
#### 快捷方式
所谓的快捷方式就是注解中定义了名为value的元素，并且在使用该注解时，如果该元素是唯一需要赋值的一个元素，那么此时无需使用key=value的语法，而只需在括号内给出value元素所需的值即可。这可以应用于任何合法类型的元素，记住，这限制了元素名必须为value，简单案例如下
```java
package com.zejian.annotationdemo;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

//定义注解
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@interface IntegerVaule{
    int value() default 0;
    String name() default "";
}

//使用注解
public class QuicklyWay {

    //当只想给value赋值时,可以使用以下快捷方式
    @IntegerVaule(20)
    public int age;

    //当name也需要赋值时必须采用key=value的方式赋值
    @IntegerVaule(value = 10000,name = "MONEY")
    public int money;

}
```
**Java8新增@Repeatable原注解**
```java
//使用Java8新增@Repeatable原注解  下面这两种方式都可以 第一种为新特性
@Target({ElementType.TYPE,ElementType.FIELD,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Repeatable(FilterPaths.class)//参数指明接收的注解class
public @interface FilterPath {
    String  value();
}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface FilterPaths {
    FilterPath[] value();
}

//使用案例
@FilterPath("/web/update")
@FilterPath("/web/add")
@FilterPath("/web/delete")
class AA{ }
```
## 枚举
枚举和类没有什么区别 只不过不可以继承了而已，因为他已经继承了枚举基类，但是他还是可以实现接口
枚举常用的函数：
1. values()  //以数组的形式返回所有的枚举成员
2. valueOf() //将普通字符串转换为枚举实例
3. comareTo() //比较两个枚举对象在定义时的顺序
4. ordinal（） //获取枚举对象的位置索引

## 泛型
> 泛型实质上是使程序员定义安全的类型。
> 泛型的机制 类名\<T\>

```java
public class OverClass<T>{
	T value;
	public T getValue(){
		return this.value;
	}

	public void setValue(T value){
		this.value = value;
	}
}

OverClass<Boolean> overClass = new OverClass<Boolean>();
overClass.setValue(true);
System.out.println(overClass.getValue);
```
**在定义泛型类的时候，一般类型名称使用T来表示，而容器的元素使用E来表示**

1. 定义泛型类时声明多个类型
```java
MutiOverClass<T1,T2>
MutiOverClass:泛型名称
MutiOverClass<Boolean, Float> muti = new MutiOverClass<Boolean, Float>();

```

2. 定义泛型类时生声明数组类型
```java
public class ArrayClass<T>{
	T[] value;
	public T getValue(){
		return this.value;
	}

	public void setValue(T[] value){
		this.value = value;
	}
}

OverClass<Boolean> overClass = new OverClass<Boolean>();
Boolean[] booleans = {true,false,true}
overClass.setValue(booleans);
System.out.println(overClass.getValue);

```
2. 集合类声明容器的元素

* ArrayList  ArrayList<E>
* HashMap  ArrayList<K,V>
* HashSet  ArrayList<E>
* Vector  ArrayList<E>

###泛型的高级应用
1. 限制泛型可用类型
	
```java
class 类名称<T extends anyClass>
```
其中anyClass指的是某个接口或者类，被这个接口限制后，泛型类的类型必须实现或者继承了这个类

2. 使用类型通配符
其作用是在创建一个泛型类对象时限制这个泛型类的类型实现或继承某个接口或类的子类。
<? extends List> 表示类型未知，但限制为List
```java
泛型名称A<？ extends List> = null
a = new A<ArrayList>();
a = new A<LinkedList>();
```

3. 继承泛型类与实现泛型接口
subclass 继承ExA时保留父类的泛型类型
```java
public class ExA<T1>{}
class subClass<T1,T2,T3> extends Exa<T1>{}
泛型名称A<？ extends List> = null
a = new A<ArrayList>();
a = new A<
```

