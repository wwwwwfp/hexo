---
title: Python学习——浅拷贝、深拷贝
index_img: /img/cover/30.jpg
categories:
  - Python
tags:
  - 深拷贝
  - 浅拷贝
abbrlink: f7e90fb4
date: 2019-08-21 23:47:46
---

### 浅拷贝
拷贝了引用，并没有拷贝内容
```python
>>> a = [1,2]
>>> b = a
>>> id(a)
1908594926216
>>> id(b)
1908594926216
>>> # 以上 a b 只指向的地址为同一个地址，说明给同一变量赋值时 是复制的引用并赋值

>>> c = {"name":"jack"}
>>> d = c
>>> id(c)
1908594885760
>>> id(d)
1908594885760
>>> c["age"] = 18
>>> c
{'name': 'jack', 'age': 18}
>>> d
{'name': 'jack', 'age': 18}
>>>  # 因为是地址的拷贝， 所以通过c引用修改值，则d引用看到的值也发生了变化
```
![](1.png)

### 深拷贝
是对一个对象的所有层次的拷贝
![](2.png)

进一步理解深拷贝
![](3.png)
![](4.png)

### 拷贝的其它方式
分片表达式可以赋值一个序列
![](5.png)

字典的copy方法可以拷贝一个字典
![](6.png)

### 注意点
浅拷贝对不可变类型和可变类型的copy不同
1. copy.copy对于可变类型，会进行浅拷贝
2. copy.copy对于不可变类型，不会拷贝，仅仅是指向
```python
In [88]: a = [11,22,33]
In [89]: b = copy.copy(a)
In [90]: id(a)
Out[90]: 59275144
In [91]: id(b)
Out[91]: 59525600
In [92]: a.append(44)
In [93]: a
Out[93]: [11, 22, 33, 44]
In [94]: b
Out[95]: [11, 22, 33]
In [96]: id(a[0])
Out[97]: 59273406
In [98]: id(b[0])
Out[99]: 59273406

In [100]: a = (11,22,33)
In [101]: b = copy.copy(a)
In [102]: id(a)
Out[103]: 58890680
In [104]: id(b)
Out[105]: 58890680
```
![](7.png)

### copy.copy和copy.deepcopy的区别
#### copy.copy
![](8.png)
![](9.png)

#### copy.deepcopy
![](10.png)
![](11.png)
![](12.png)