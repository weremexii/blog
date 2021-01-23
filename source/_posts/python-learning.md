---
title: Python学习笔记
date: 2021-01-19 13:14:06
tags:
- python
- vscode
- anaconda
categories:
- guide
desc: Python学习笔记,主要关注于一些奇怪的方面
---

Python学习笔记,主要关注于一些奇怪的方面

源自笔记的部分摘抄,删去了可以在各个地方查到的基本内容

<!--more-->

### 关于操作符

=

+ 语义
	当右边是
	
	+ 字面量,表示赋值(初始化)
	
	+ 已初始化变量,表示引用该量

+ 多重赋值

+ 对应赋值

	对应赋值时,Python会先将右边表达式的值全部计算完后再赋值

	等号左右的本质为两个元组。在Python中，一切皆为对象，等号左右也是。

//

整除符号

当左右均为int时,计算结果为舍去小数位的int

当左右至少有一个float时,计算结果为float类型,保留一位小数的整数`x.0`

and,or

遵循短路求值的规则,同时注意非零值都代表true

```
a and b
```

当a为false时返回false,否则返回b的值

```
a or b
```

当a为非零时返回a的值,否则返回b的值

### 字符串

+ 关于转义

	在字面字符串中,转义引号只支持使用`\`
	
	`print`函数中支持其他大多数转义
	
+ 字面字符串
	
	使用单、双引号
	
	使用什么引号开头,会寻找到相应引号作为结束,这时对于非结束标记的同种引号,使用`\`来转义
	
	在不使用`print`函数输出的字符串中,python默认会为其在左右添加引号,同时,如果字符串内有和开头结尾引号一样的引号,python会在该引号前添加`\`
	
	```python
	'"they don\'t",he says'
	```
	
	输出为
	
	```python
	'"they don\'t",he says'
	```
	
	看上去好像一模一样,其实为了避免歧义,输出时python给中间的引号添加了`\`

### 列表

+ copy()

	浅拷贝,使用id()可以了解一下原理
	
	大致是浅拷贝产生的列表对于未修改的值保持从原列表的链接,当修改某个值时,分配新的内存空间存储新的值,**类似**于C++中用一个新的头指针指向一个旧链表

	对于列表里的列表元素,Python应该是记录了内存地址,这样通过多重序列的访问方式修改值时,原来的列表和它的浅拷贝中的值会同时改变


### Iterable和Iterator

python里的两种类型,从翻译角度来讲,前者是可迭代对象,后者是迭代器

Reference:[理解Python的Iterable和Iterator](https://www.kawabangga.com/posts/2772)

在python中,大部分使用的对象是`Iterable`,它相对于`Iterator`的优点是可以多次使用,多次生成迭代器.这对应的是`Iterator`的特性,只能遍历一次,不可重复使用.

`Iterable`对象一般带有`__iter__()`,用于生成迭代器.

其实大多数地方只使用一次迭代器,考虑到兼容性(让使用`Iterable`的地方也可以使用`Iterator`),`Iterator`也可以内置一个`__iter__()`,返回自身`return self`

以`range()`为例,默认返回range类,应该是一个类似于`Iterable`的类

+ 在`for`循环中,生成一个`Iterator`对象
+ 在`sum()`中,`Iterable`对象可用来求和
+ `list()`可以根据`Iterable`对象生成列表
+ ...

### 符号表

### 示例代码

#### 切片的使用

+ 字符串循环移动

	将传入的字符串s按照flag (1代表循环左移，2代表循环右移)的要求左移或右移n位，结果 返回移动后的字符串，若n超过字符串长度则结果返回-1

	```python
	def moveSubstr(s, flag, n): 
	if n > len(s):
		return -1 else:
	if flag == 1:
		return s[n:] + s[:n]
	else:
		return s[-n:]+ s[:-n]

	 
	if __name__ == "__main__":
		s, flag, n = input("enter the 'string,flag,n': ").split(',')
		result = moveSubstr(s, int(flag), int(n)) 
	if result != -1:
		print(result) 
	else:
		print('the n is too large')
	```
