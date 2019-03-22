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
1. 版本的管理
2. 地区的管理
3. 包名的配置
4. 更新包的配置
5. 资源压缩配置
6. Jenkins
7. 包体的放置

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

##内部类
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