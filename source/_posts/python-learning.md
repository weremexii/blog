---
title: python-learning
date: 2021-01-19 13:14:06
tags:
- python
- vscode
- anaconda
categories:
- guide
desc: Python学习笔记,主要关注于一些奇怪的方面
---

### 关于操作符 =

右边是

+ 字面量,表示赋值(初始化)
+ 已初始化变量,表示引用该量

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

### Iterable和Iterator

python里的两种类型,从翻译角度来讲,前者是可迭代对象,后者是迭代器

Reference:[理解Python的Iterable和Iterator](kawabangga.com/posts/2772kawabangga.com/posts/2772)

在python中,大部分使用的对象是`Iterable`,它相对于`Iterator `的优点是可以多次使用,多次生成迭代器.这对应的是`Iterator`的特性,只能遍历一次,不可重复使用.

`Iterable`对象一般带有`__iter__()`,用于生成迭代器.

其实大多数地方只使用一次迭代器,考虑到兼容性(让使用`Iterable`的地方也可以使用`Iterator`),`Iterator`也可以内置一个`__iter__()`,返回自身`return self`

以`range()`为例,默认返回`Iterable`对象

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