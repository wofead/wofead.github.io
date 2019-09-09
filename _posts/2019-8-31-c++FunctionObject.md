---
layout:     post
title:      函数对象
subtitle:   C++
date:       2019-8-31
author:     Jow
header-img: img/post-bg-infinity.jpg
catalog: 	 true 
tags:
    - C++

---

### 目录
1. 函数对象
2. 函数符概念
3. 预定义的函数符
4. stl算法


> study and think. then you will get more.


## 函数对象
很对stl算法都是用函数对象--也叫函数符。函数符是可以以函数方式与()结合使用的任意对象。这包括函数名、指向函数的指针和重载了()运算符的类对象(即定义了函数operate()()的类）。
```c++
for_each(books.begin(), books.end(), ShowReview);

template<class InputIterator, class Function>
Function for_each(TnputIterator first, InputIterator last, Function f);
void ShowReview(const Review &);
```
第三个参数可以是常规函数，也可以是函数符。怎么声明for_each函数呢？

上面说明了for_each的函数声明，标识符ShowReview的类型将为void(*)(const Review&),这也是赋给模板参数Function的类型。Function参数可以表示具有重载()运算符的类类型。最周，for_each()代码具有一个使用f()的表达式。在ShowReview()示例中，f是指向函数的指针，而f()调用该函数。如果是一个对象，则调用其重载的()的运算符。

## 函数符概念
* 生成器:是不用参数就可以调用的函数符
* 一元函数：一个参数可以调用的函数符
* 二元函数：两个参数可以调用的函数符

* 返回bool值的一元函数是谓词
* 返回bool值的二元函数是二元谓词

sort的第三个参数就是二元谓词。而list中的remove_if函数就是一员谓词。
```c++
template<class T>
class TooBig {
private:
	T cutofff;
public:
	TooBig(const T& t) :cutofff(t) {}
	bool operator()(const T& v){
		return v > cutofff;
	}
};

void outint(int n) { cout << n << " ";}

int main() {
	TooBig<int> f100(100);
	int vals[10] = { 50,100,90,180,60,210,415,88,188,200 };
	list<int> yadayada(vals, vals + 10);
	list<int> etcetera(vals, vals + 10);
	yadayada.remove_if(f100);
	etcetera.remove_if(TooBig<int>(200));
	for_each(yadayada.begin(), yadayada.end(), outint);
	cout << endl;
	for_each(etcetera.begin(), etcetera.end(), outint);
}
```

## 预定义的函数符
* transform:他有两个版本，第一个版本就是函数符是一元谓词，第三个参数就是输出到那里的迭代器。另一个就是函数符是二元谓词，第三个参数是另一个对象的开头迭代器，第四个参数就是输出到那里的迭代器。
* plus:+
* minus:-
* multiplies:*
* divides:/
* modulus:%
* negate:-
* equal_to:==
* not_equal_to:!=
* greater:>
* less:<
* greater_equal:>=
* less_equal:<=
* logical_and:&&
* logical_or:||
* logical_not:!

```c++
void outint(int n) { cout << n << " ";}
void outDouble(double n) { cout << n << " ";}
int op_increaseTwo(double i, double j) { return i + j; }
int op_increaseOne(double i) { return ++i; }

int main() {
	const int LIM = 5;
	double arr1[LIM] = { 36,39,42,45,48 };
	vector<double> gr8(arr1, arr1 + LIM);
	vector<double> gr9(arr1, arr1 + LIM);
	ostream_iterator<double, char> out(cout, " ");
	transform(gr8.begin(), gr8.end(), gr9.begin(), op_increaseOne);
	for_each(gr9.begin(), gr9.end(), outDouble);
	transform(gr8.begin(), gr8.end(), gr9.begin(), gr9.begin(), plus<double>());
	cout << endl;
	for_each(gr9.begin(), gr9.end(), outDouble);
}

```

## stl算法
stl包含很多处理容器的非成员函数，前面我们用过sort(),copy(),find(),random_shuffle(),set_union(),set_intersection(),set_difference()和transform。

stl将算法库分成4组：
* 非修改式序列操作；
* 修改式序列操作；
* 排序和相关操作；
* 通用数字运算。

前三组头文件在algorithm中描述，disizu是用于数组数据的，有自己的头文件，称为numeric。

1. 非修改式的序列操作对区间中的每个元素进行操作。这些操作不修改容器总的内容。find和for_each.
2. 修改式的操作对区间中的每个元素操作。但是他们可以修改其中的内容。transform,random_shuffle()和copy().
3. 排序和相关的操作包括多个排序函数(sort)和其它操作。

**string虽然不是stl的组成部分，但是设计的时候考虑了STL，它包含begin(),end(),rbegin()和rend()等成员。**
