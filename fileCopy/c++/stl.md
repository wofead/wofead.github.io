---
layout:     post
title:      c++的迭代器
subtitle:   c++
date:       2019-8-19
author:     Jow
header-img: img/post-bg-infinity.jpg
catalog: 	 true 
tags:
    - C++
    - STL

---

### 目录
1. vector的使用
2. 迭代器
3. deque（双端队列）
4. list
5. queue
6. stack
7. 容器的常用操作函数
8. set
9. map
10. initializer_list


> 持之以恒，有尝试或许才有结果。


## vector的使用
stl都是泛型类，可以定义任意类型的矢量等。C++标准要求容器的实现内存必须是连续的，唯一可以和标准c兼容的stl容器，任意元素的读取、修改具有常数时间复杂度，在序列尾部进行插入、删除是常数时间复杂度，但是在头部的插入和删除的时间复杂度是O(N)，可以在任意位置插入新元素，有随机访问功能，插入删除操作需要考虑。

* push_back():添加一个拷贝到最后一位
* sort排序，前面两个是从哪里开始和结束的迭代器，第三个参数为可选的，如果没有排序函数worseThan，需要重构运算符'<'
* random_shuffle():随机排序

* vector是最简单也是最重要的一个容器。其头文件为<vector>.
* vector是数组的一种类表示，它有以下优点：自动管理内存、动态改变长度并随着元素的增减而增大或缩小。
* 在尾部添加元素时间是固定的，在头部或者中间添加或者删除元素是线性时间。
* vector是可反转容器。


```c++
#include<iostream>
#include<string>
#include<vector>
#include<algorithm>

struct Review
{
	std::string title;
	int rating;
};

bool operator<(const Review& r1, const Review& r2);
bool worseThan(const Review& r1, const Review& r2);
bool FillReview(Review& rr);
void ShowReview(const Review& rr);

int main()
{
	using namespace std;
	vector<Review> books;
	Review temp;
	while (FillReview(temp))
	{
		books.push_back(temp);
	}
	if (books.size() > 0)
	{
		for_each(books.begin(), books.end(), ShowReview);
		sort(books.begin(), books.end());
		for_each(books.begin(), books.end(), ShowReview);
		sort(books.begin(), books.end(),worseThan);
		for_each(books.begin(), books.end(), ShowReview);
		random_shuffle(books.begin(), books.end());
		for_each(books.begin(), books.end(), ShowReview);
	}
}

bool operator<(const Review& r1, const Review& r2)
{
	if (r1.title < r2.title)
	{
		return true;
	}
	else if (r1.title == r2.title && r1.rating < r2.rating)
	{
		return true;
	}
	return false;
}

bool worseThan(const Review& r1, const Review& r2)
{
	if (r1.rating < r2.rating)
	{
		return true;
	}
	return false;
}


bool FillReview(Review& rr)
{
	std::cout << "Enter bool title (quit with quit):" << std::endl;
	std::getline(std::cin  , rr.title);
	if (rr.title == "quit")
	{
		return false;
	}
	std::cout << "Enter the bool rating:" << std::endl;
	std::cin >> rr.rating;
	if (!std::cin)
	{
		return false;
	}
	while (std::cin.get() != '\n')
	{
		continue;
	}
	return true;
}

void ShowReview(const Review& rr)
{
	std::cout << rr.rating << "\t" << rr.title << std::endl;
}

```

循环遍历除了for_each还可以使用基于范围的for循环:

```c++
double prices[5] = {4.99, 10.99, 6.87, 7.99, 8.49};
for(double x : prices)
	cout << x << endl;
```

## 迭代器
什么是迭代器？它是广义指针。事实上，它可以是指针，也可以是一个可对其执行类似指针的操作-如通过*来解除引用以及通过++进行自身的自增。

在容器中，一般begin()返回的迭代器指向第一个元素，而end()则返回的是超尾元素，什么是超尾？指向容器最后一个元素后面的那个元素。

模板是的算法独立于存储的数据类型，而迭代器使算法独立于使用的容器类型。

泛型编程旨在使用同一个find函数来处理数组、链表或任何其它容器类型。即函数不仅独立于容器中存储的数据类型，而且独立于容器本身的数据结构。迭代器应该具有的特征：
* 应能够对迭代器执行解除引用的操作
* 应能够将一个迭代器赋值给另一个
* 应能够将一个迭代器和另一个进行比较
* 能够遍历容器中的所有值


迭代器有begin、end、rbegin以及rend。

STL定义了5种迭代器，都可以执行解除引用操作，也可以进行比较，看其是相同还是不相同。
1. 输入迭代器：即来自容器的信息被视为输入，就像来自键盘的的信息对于程序来说输入一样。
2. 输出迭代器：stl使用术语"输出"来指用于将信息从程序出输给容器的迭代器，因此程序的输出就是容器的输入。
3. 正向迭代器：只能使用++运算符来遍历容器。
4. 双向迭代器：双向迭代器具有正向迭代器的所有特征，同时支持两种递减运算符。
5. 随机访问迭代器：能够直接跳到容器中的任意一个元素。

迭代器形成了一个层次结构，正向迭代器具有输入和输出迭代器的功能，同时具有自己的功能，后面的两个拥有前面的功能。

迭代器是广义的指针，而指针满足所有的迭代器要求。迭代器是STL算法的接口，而指针是迭代器，所以STL算法可以使用指针来对基于指针的非STL容器进行操作。

预定义的迭代器：copy(),ostream_iterator,istream_iterator.

```c++
int casts[10] = {6, 7, 2, 9, 4, 11, 8, 7, 10, 5};
vector<int> dice[10];
copy(casts, casts +10, dice.begin());
ostream_iterator<int, char> out_iterator(cout," ");
copy(dice.begin(),dice.end(),ostream_iterator<int,char>(cout," ");
copy(istream_iterator<int,char>(cin), istream_iterator<int,char>(),dice.begin());
```

copy的前两个参数表示复制的范围，最后一个参数表示要将第一个元素复制到什么位置。

out_iterator迭代器是一个接口，让您能够使用cout来显示信息。第一个模板参数指出了被发送给输入流的数据类型；第二个模板参数指出了输出流使用的字符类型。这个字符是在发送给输入流每个数据项显示的分隔符。

istream_iterator和ostream类似一个是输入另一个是输出。

除了上述的迭代器还有reverse_iterator,back_insert_iterator,front_insert_iterator和insert_iterator.

reverse_iterator是反向迭代器，对于这种迭代器执行递增操作，将导致其被递减。rbegin和rend就是反向迭代器，前者指向超尾，后者指向第一个元素。反向指针总是**先通过递减，再解除引用的**。

后面的三种插入迭代器，解决不覆盖原来容器的值进行元素的新增。back负责插入到容器的尾部，front负责插入到容器的头部，insert负责插入到参数指定位置的前面。

```c++
#include<iostream>
#include<string>
#include<iterator>
#include<vector>
#include<algorithm>
using namespace std;

void output(const string& str) {
	cout << str << " ";
}

int main() {
	string s1[4] = { "fine","fish","fashion","face" };
	string s2[2] = { "boys","buy" };
	string s3[2] = { "singer","silly" };
	vector<string> words(4);// 初始化words的大小为4
	for_each(words.begin(), words.end(), output);
	copy(s1, s1 + 4, words.begin());
	copy(s2, s2 + 2, back_insert_iterator<vector<string>>(words));
	copy(s3, s3 + 2, insert_iterator<vector<string>>(words, words.begin()));
	for_each(words.begin(), words.end(), output);
	return 0;
}
```

## deque（双端队列）
序列容器，内存特使连续的，和vector相似，区别在序列的头部插入和删除操作也是常数时间复杂度，可以在任何位置插入新元素，有随机访问功能。
* 头文件deque
* 在STL中deque类似vector，并且支持随机访问。区别在于：从deque起始位置插入删除元素时间是固定的。
* 为了实现在deque俩段执行插入和删除操作的时间为固定这一目的，deque对象设计比vector设计更为复杂一些。因此，在序列中部执行插入删除操作时，vector更快一些。

## list
* list表示双向链表。头文件list
* list为可反转容器。
* list不支持数组表示法和随机访问。
* 与矢量迭代器不同，从容器中插入或删除元素之后，链表迭代器指向的元素不变。这与链表的特性有关，删除链表中的元素并不改变其他元素的位置。
* list强调的是快速的插入和删除，而不是随机回合快速访问。

1. merge(list a):调用merge函数的list和a进行合并。这两个链表要求已经排序
2. remove(val):删除链表中所有值为val的对象。
3. sort():使用<运算符对链表进行排序
4. splice(iterator pos, list x):将链表x插入到pos前面，x将为空。
5. unique():将连续相同的元素压缩为单个元素。

insert()与splice()之间的不同主要在与：insert()将原始区间的副本插入到目标地址，而splice()则将原始区间移到目标地址。splice()执行后，迭代器仍有效。也就是说原本指向two中一个元素的迭代器，在使用过splice后仍然指向它。

## queue
* 头文件queue
* queue即队列，可以从队尾插入，队头删除，获取队尾和队头的值。

1. empty():是否为空。
2. size()：大小；
3. front()：返回队首的引用；
4. back()：返回队尾元素的引用；
5. push()：队尾插入
6. pop()：队首删除；

## stack
* 头文件<stack>
* 压栈、出栈、栈顶、元素个数、是否为空。

1. empty()：是否为空。
2. size()：大小；
3. top()：栈顶引用；
4. push()：队尾插入
5. pop()：队首删除；

## 容器的常用操作函数
**基本特征：**
以下用X表示容器类型（后面会讲到），T表示储存的对象类型（如int）；a和b表示为类型X的值；u表示为一个X容器的标识符（如果X表示vector<int>，则u是一个vector<int>对象。）

* X::iterator：指向T的迭代器类型
* X u：创建一个名为u的空容器
* X():创建一个匿名空容器
* X u(a)：创建一个u的容器，并用a来初始化
* a.begin()：指向容器第一个元素的迭代器
* a.end()：返回超尾值的迭代器
* a.size()：容器大小
* a.swap()：交换a和b的值
* a == b：长度一样，且每个值相同
* a != b：与上面相反

**序列容器基本特征**：vector、deque、list、queue、stack。以下用t表示类型为T（储存在容器中的值的类型）的值，n表示整数，p、q、i和j表示迭代器。

* X a(n,t)：序列a，由n个t组成
* X(n,t):匿名序列，由n个t组成
* X a(i,j)：创建a序列，并将其初始化为区间[i,j)的内容
* X(i,j):创建一个匿名序列，并将其初始化为区间[i,j)的内容
* a.insert(p,t):t插入到p前面
* a.insert(p,n,t):n个t插入到p前面
* a.insert(p,i,j)：将区间[i,j)的元素插入到p前面
* a.erase(p):删除p所指向的元素
* a.erase(p,q)：删除区间[p,q)
* a.clear():清空容器


下面是[i,j)的正确用法。

```c++
int a[] = {1,5,4,3};
dice.insert(dice.begin(),a,a+4);
```

* a.front():vector,list,deque
* a.back():vector,list,deque
* a.push_front():list、deque
* a.push_back():vector,list,deque
* a.pop_front():list、deque
* a.pop_back():vector,list,deque
* a[n]:vector、deque
* a.at[t]:vector、deque

**a[n]和a.at(n)都返回一个指向容器中第n个元素的引用。区别在于：如果n落在容器有效区间之外，a.at(n)将执行边界检查，并引发out_of_range异常。**

>之所以vector没有push_front()，是因为vector执行此表达式复杂度为线性时间，而deque为固定时间.

## set
* 储存同一类型的数据元素
* 每个元素的值唯一
* 根据元素的值自动排列大小(有序性)
* 无法直接修改元素
* 高效的插入删除操作

```c++
set<int> a={0,1,2,9};
a.insert(6);
for(auto it = a.begin();it != a.end();it++)	cout << *it;//输出01269

set<int> a = {0,1,2,9};
set<int> b = {3,4,5};
auto first = b.begin();
auto second = b.end();
a.insert(first,second);
for(auto it = a.begin();it != a.end();it++)	cout << *it;

#include<iostream>
#include<set>
#include<algorithm>
using namespace std;
int main()
{
	set<int> A = {1,2,3}, B= {2,4,5},C;
	set_union(A.begin(),A.end(),B.begin(),B.end(),
			insert_iterator<set<int> >(C,C.begin()));
	for(auto it = C.begin();it != C.end();it++)
		cout << *it <<" ";
	return 0;
}
```

* a.insert(first,second):其中first为指向区间左侧的迭代器，second为指向右侧的迭代器。作用是将first到second区间内元素插入到a（左闭右开）。
* a.insert(x) :其中a为set<T>型容器，x为T型变量
* a.erase(x)：删除建值为x的元素
* a.erase(first,second)：删除first到second区间内的元素（左闭右开）
* a.erase(iterator):删除迭代器指向的元素
* set中的删除操作是不进行任何的错误检查的，比如定位器的是否合法等等，所以用的时候自己一定要注意。
* a.count(x):返回容器a中元素x的个数
* lower_bound（x1）:返回第一个不小于键参数x1的元素的迭代器
* upper_bound（x2）:返回最后一个大于键参数x2的元素的迭代器
* set_union():对集合取并集。set_union()函数接受5个迭代器参数。前两个迭代器定义了第一个集合的区间，接下来的俩个迭代器定义了第二个集合的区间，最后一个迭代器是输出迭代器，指出将结果集合复制到什么位置。
* set_intersection():对集合取交集，它的接口与set_union()相同。

**set的几个问题**
1. 为何map和set的插入删除效率比用其他序列容器高？因为对于关联容器来说，不需要做内存拷贝和内存移动。set容器内所有元素都是以节点的方式来存储，其节点结构和链表差不多，指向父节点和子节点。
2. 为何每次insert之后，以前保存的iterator不会失效？iterator这里就相当于指向节点的指针，内存没有变，指向内存的指针怎么会失效呢(当然被删除的那个元素本身已经失效了)。
3. 当数据元素增多时，set的插入和搜索速度变化如何？如果你知道log2的关系你应该就彻底了解这个答案。

## map
如果说set对应数学中的“集合”，那么map对应的就是“映射”。map是一种key-value型容器，其中key是关键字，起到索引作用，而value就是其对应的值。与set不同的是它支持下标访问。头文件是<map>。
* 增加和删除节点对迭代器的影响很小。
* 快速的查找
* 自动建立key-value的对应，key和value可以是任何你想需要的类型
* 可以根据key修改value的值
* 支持下标[]操作

```c++
#include<iostream>
#include<map> 
using namespace std;
int main()
{
	map<string,int> m;
	m["abc"] = 5;
	m["cdf"] = 6;
	m["b"] = 1;
	m.insert(make_pair("e",6));//insert插入
	for(auto it = m.begin();it != m.end();it++)
		cout << it->first <<" " << it->second << endl;
	return 0;
}
```

map的常用操作函数：
* erase(key):删除键为key的元素
* erase(it):删除迭代器it所指向的元素
* 使用迭代器遍历map容器，其中每一个元素可以看成是pair类型的，访问第一个位置的key值可以用it->first访问，第二个位置value的值可以用it->second访问，其中it是指向该元素的迭代器。
* m.count(key)：返回map中key出现的次数（0或1）
* m.find(key)：返回指向key位置的迭代器.若无则返回m.end()
* m.insert(make_pair( ) )：插入一个元素(必须以pair形式插入)
* m.lower_bound(key)：返回指向第一个键值不小于key的元素的迭代器
* m.upper_bound(key)：返回指向第一个键值大于key的元素的迭代器

## initializer_list
代码中使用initializer_list对象，需要包含头文件initializer_list。这个模板类中包含成员函数begin()和end(),可以使用者两个函数来访问列表元素。

```c++
initializer_list<double> d1 = {1.1,1.2,1.3};
double total = 0;
for(auto p = d1.begin(); p != d1.end(); p++){
	total += *p;
}

vector<double> payments {45.99, 41.20, 45.16};
```

上面payments可行的原因是容器类中现在包含将initializer_list<T>作为参数的构造函数。
