---
title: Python学习——数据类型之非数字型
index_img: /img/cover/25.jpg
categories:
  - Python
tags:
  - Python
abbrlink: 543ce00e
date: 2019-06-14 09:04:28
---
**Python 中数据类型非数字型包括：**
+ 字符串
+ 列表（List）
+ 元组
+ 字典
```
注：非数字型变量（有索引）都支持以下特点
1.都是一个序列 sequence，也可以理解成容器
2.取值[]
3.遍历 for in
4.计算长度、最大/最小值、比较、删除
5.链接 +   和重复  *
6.切片
```

**切片**
切片操作基本表达式：object[start_index:end_index:step]

参考：https://www.jianshu.com/p/15715d6f4dad

### 字符串
字符串是 Python 中最常用的数据类型。可以使用引号( ’ 或 " )来创建字符串。eg：name = ‘张三’

Python 不支持单字符类型，单字符在 Python 中也是作为一个字符串使用

字符串常用方法：
```python
str = "Hello World"
print(str[0]) # H   通过索引获取字符串中字符
print(str[1:3]) # el 注：左闭右开，含头不含尾  str[1:] 表示截取下标1(包含)以后的所有字符  str[:3]截取下标3以前(不含)的所有字符串
print("H" in str) # True    如果字符串中包含给定的字符返回 True
print("S" not in str) # True    如果字符串中不包含给定的字符返回 True
print("A" + "b") # Ab   字符串拼接
print("Ab" * 2) # AbAb  重复输出两次Ab
print (r'\n')  # \n  原始字符串：所有的字符串都是直接按照字面的意思来使用，没有转义特殊或不能打印的字符。 原始字符串除在字符串的第一个引号前加上字母 r（可以大小写）以外，与普通字符串有着几乎完全相同的语法。
print (R'\n')  # \n   同上
para_str = """这是一个多行字符串的实例    
多行字符串可以使用制表符
TAB ( \t )。
也可以使用换行符 [ \n ]。
"""
print (para_str)    #python三引号允许一个字符串跨多行，字符串中可以包含换行符、制表符以及其他特殊字符  输出结果如下
"""
这是一个多行字符串的实例
多行字符串可以使用制表符
TAB ( 	 )。
也可以使用换行符 [ 
 ]。
"""
```

Python 的字符串内建函数

```python
capitalize()  # 将字符串的第一个字符转换为大写
center(width, fillchar)  #返回一个指定的宽度 width 居中的字符串，fillchar 为填充的字符，默认为空格。
count(str, beg= 0,end=len(string))  # 返回 str 在 string 里面出现的次数，如果 beg 或者 end 指定则返回指定范围内 str 出现的次数
bytes.decode(encoding="utf-8", errors="strict") #Python3 中没有 decode 方法，但我们可以使用 bytes 对象的 decode() 方法来解码给定的 bytes 对象，这个 bytes 对象可以由 str.encode() 来编码返回。
encode(encoding='UTF-8',errors='strict') # 以 encoding 指定的编码格式编码字符串，如果出错默认报一个ValueError 的异常，除非 errors 指定的是'ignore'或者'replace'
endswith(suffix, beg=0, end=len(string)) # 检查字符串是否以 obj 结束，如果beg 或者 end 指定则检查指定的范围内是否以 obj 结束，如果是，返回 True,否则返回 False.
expandtabs(tabsize=8) # 把字符串 string 中的 tab 符号转为空格，tab 符号默认的空格数是 8 。
find(str, beg=0, end=len(string)) # 检测 str 是否包含在字符串中，如果指定范围 beg 和 end ，则检查是否包含在指定范围内，如果包含返回开始的索引值，否则返回-1
index(str, beg=0, end=len(string)) # 跟find()方法一样，只不过如果str不在字符串中会报一个异常.
isalnum() # 如果字符串至少有一个字符并且所有字符都是字母或数字则返 回 True,否则返回 False
isalpha() # 如果字符串至少有一个字符并且所有字符都是字母则返回 True, 否则返回 False
isdigit() # 如果字符串只包含数字则返回 True 否则返回 False..
islower() # 如果字符串中包含至少一个区分大小写的字符，并且所有这些(区分大小写的)字符都是小写，则返回 True，否则返回 False
isnumeric() # 如果字符串中只包含数字字符，则返回 True，否则返回 False
isspace() # 如果字符串中只包含空白，则返回 True，否则返回 False.
title() #返回"标题化"的字符串,就是说所有单词都是以大写开始，其余字母均为小写(见 istitle())
istitle() # 如果字符串是标题化的(见 title())则返回 True，否则返回 False
isupper() # 如果字符串中包含至少一个区分大小写的字符，并且所有这些(区分大小写的)字符都是大写，则返回 True，否则返回 False
join(seq) #以指定字符串作为分隔符，将 seq 中所有的元素(的字符串表示)合并为一个新的字符串  eg:print("aaa".join("1234")) > 1aaa2aaa3aaa4   eg:print(" ".join("1234")) > 1 2 3 4
len(string) #返回字符串长度
ljust(width[, fillchar]) #返回一个原字符串左对齐,并使用 fillchar 填充至长度 width 的新字符串，fillchar 默认为空格。
lower() # 转换字符串中所有大写字符为小写.
lstrip() # 截掉字符串左边的空格或指定字符。
maketrans() # 创建字符映射的转换表，对于接受两个参数的最简单的调用方式，第一个参数是字符串，表示需要转换的字符，第二个参数也是字符串表示转换的目标。
max(str) # 返回字符串 str 中最大的字母。
min(str) # 返回字符串 str 中最小的字母。
replace(old, new [, max]) # 把 将字符串中的 str1 替换成 str2,如果 max 指定，则替换不超过 max 次。
rfind(str, beg=0,end=len(string)) # 类似于 find()函数，不过是从右边开始查找.
rindex( str, beg=0, end=len(string)) # 类似于 index()，不过是从右边开始.
rjust(width,[, fillchar]) # 返回一个原字符串右对齐,并使用fillchar(默认空格）填充至长度 width 的新字符串
rstrip() #删除字符串字符串末尾的空格.
split(str="", num=string.count(str)) #num=string.count(str)) 以 str 为分隔符截取字符串，如果 num 有指定值，则仅截取 num+1 个子字符串
splitlines([keepends]) #按照行('\r', '\r\n', \n')分隔，返回一个包含各行作为元素的列表，如果参数 keepends 为 False，不包含换行符，如果为 True，则保留换行符。
startswith(substr, beg=0,end=len(string)) #检查字符串是否是以指定子字符串 substr 开头，是则返回 True，否则返回 False。如果beg 和 end 指定值，则在指定范围内检查。
strip([chars]) #在字符串上执行 lstrip()和 rstrip()
swapcase() #将字符串中大写转换为小写，小写转换为大写
translate(table, deletechars="") # 根据 str 给出的表(包含 256 个字符)转换 string 的字符, 要过滤掉的字符放到 deletechars 参数中
upper() # 转换字符串中的小写字母为大写
zfill (width) #返回长度为 width 的字符串，原字符串右对齐，前面填充0
isdecimal() #检查字符串是否只包含十进制字符，如果是返回 true，否则返回 false。
```

### 列表（List）
列表是最常用的Python数据类型，它可以作为一个方括号内的逗号分隔值出现。

列表的数据项不需要具有相同的类型 。eg: list1 = [‘physics’, ‘chemistry’, 1997, 2000]

定义： 如 names = [“name1”,“name2”]
```python
列表对 + 和 * 的操作符与字符串相似。+ 号用于组合列表，* 号用于重复列表。
Python 表达式	结果	描述
len([1, 2, 3])	3	长度
[1, 2, 3] + [4, 5, 6]	[1, 2, 3, 4, 5, 6]	组合
['Hi!'] * 4	['Hi!', 'Hi!', 'Hi!', 'Hi!']	重复
3 in [1, 2, 3]	True	元素是否存在于列表中
for x in [1, 2, 3]: print(x, end=" ")	1 2 3	迭代

列表还支持拼接操作：
>>>squares = [1, 4, 9, 16, 25]
>>> squares += [36, 49, 64, 81, 100]
>>> squares
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

使用嵌套列表即在列表里创建其它列表，例如：
>>>a = ['a', 'b', 'c']
>>> n = [1, 2, 3]
>>> x = [a, n]
>>> x
[['a', 'b', 'c'], [1, 2, 3]]
>>> x[0]
['a', 'b', 'c']
>>> x[0][1]
'b'
```

```python
Python列表函数
len(list) # 列表元素个数
max(list) # 返回列表元素最大值
min(list) #返回列表元素最小值
list(seq) # 将元组转换为列表

Python列表方法
list.append(obj) # 在列表末尾添加新的对象
list.count(obj) # 统计某个元素在列表中出现的次数
list.extend(seq) # 在列表末尾一次性追加另一个序列中的多个值（用新列表扩展原来的列表）
list.index(obj) #从列表中找出某个值第一个匹配项的索引位置
list.insert(index, obj) #将对象插入列表
list.pop([index=-1]) # 移除列表中的一个元素（默认最后一个元素），并且返回该元素的值
list.remove(obj) # 移除列表中某个值的第一个匹配项
list.reverse() # 反向列表中元素
list.sort( key=None, reverse=False) # 对原列表进行排序
list.clear() # 清空列表
list.copy() #复制列表
注： del关键字也可以删除列表中元素  eg：del list[0] 删除列表中第一个元素
     del关键字是将变量从内存中删除  eg name = "zhangsan"  del后如果再使用name属性会报错
```

### 元组（Tuple）
元组与列表类似，不同之处时元组的元素不能更改

元组表示多个元素组成的序列

元组用 () 定义，用于存储一串信息，数据之间用，分割

元组的索引从0开始
```python
元组创建
>>>tup1 = ('Google', 'Runoob', 1997, 2000);
>>> tup2 = (1, 2, 3, 4, 5 );
>>> tup3 = "a", "b", "c", "d";   #  不需要括号也可以
创建空元组
tup1 = ();

元组中只包含一个元素时，需要在元素后面添加逗号，否则括号会被当作运算符使用：
>>> name = ("a")
>>> type(name)
<class 'str'>
>>> age = (23)
>>> type(age)
<class 'int'>
正确案例
>>> tup1 = (50,)
>>> type(tup1)     # 加上逗号，类型为元组

元组与字符串类似，下标索引从0开始，可以进行截取，组合等
tup1 = ('Google', 'Runoob', 1997, 2000)
tup2 = (1, 2, 3, 4, 5, 6, 7 )
print ("tup1[0]: ", tup1[0])      # tup1[0]:  Google
print ("tup2[1:5]: ", tup2[1:5])      # tup2[1:5]:  (2, 3, 4, 5)

元组中的元素值是不允许修改的，但我们可以对元组进行连接组合
>>> test = (12,)
>>> test[0] = 11
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment

连接组合
>>> temp1 = (1,2,3)
>>> temp2 = ("a","b")
>>> temp2+temp1
('a', 'b', 1, 2, 3)
>>>
元组中的元素值是不允许删除的，但我们可以使用del语句来删除整个元组
del temp2
```
元组运算符
```python
len((1, 2, 3))	   # 3	计算元素个数
(1, 2, 3) + (4, 5, 6)	 # (1, 2, 3, 4, 5, 6)	连接
('Hi!',) * 4        # 	('Hi!', 'Hi!', 'Hi!', 'Hi!')	复制
3 in (1, 2, 3)	   #True	元素是否存在
for x in (1, 2, 3): 
    print (x,)	   #1 2 3	迭代
```

元组内置函数
```python
len(tuple)    #计算元组元素个数。
max(tuple)   #返回元组中元素最大值。
min(tuple)    #返回元组中元素最小值。
tuple(seq)    #将列表转换为元组。
```

### 字典
字典是另一种可变容器模型，且可存储任意类型对象。

字典的每个键值(key=>value)对用冒号(:)分割，每个对之间用逗号(,)分割，整个字典包括在花括号({})中 ，类似于java中的map

字典时无序的对象集合

键必须是唯一的，但值则不必。

值可以取任何数据类型，但键必须是不可变的，如字符串，数字或元组。

例如：stu = {“name”:“张三”,“age”:19}

dict2 = { ‘abc’: 123, 98.6: 37 }

常用方法
```python
取值： stu["name"] #张三    注如果没有key值，取值会报错
新增/修改字典： stu["name"] = "李四"  #如果字典没有name键为新增，如果有则修改
删除属性：del stu["name"]
清空：stu.clear()
删除字典: del stu
```

内置函数

```python
len(dict)  # 计算字典元素个数，即键的总数。
str(dict)  #输出字典，以可打印的字符串表示。 
type(variable) # 返回输入的变量类型，如果变量是字典就返回字典类型。
```

内置方法
```python
radiansdict.clear()    #删除字典内所有元素
radiansdict.copy()    #返回一个字典的浅复制
radiansdict.fromkeys()  #创建一个新字典，以序列seq中元素做字典的键，val为字典所有键对应的初始值
radiansdict.get(key, default=None)  #返回指定键的值，如果值不在字典中返回default值
key in dict    # 如果键在字典dict里返回true，否则返回false
radiansdict.items()    # 以列表返回可遍历的(键, 值) 元组数组
radiansdict.keys()  # 返回一个迭代器，可以使用 list() 来转换为列表
radiansdict.setdefault(key, default=None) # 和get()类似, 但如果键不存在于字典中，将会添加键并将值设为default
radiansdict.update(dict2)  # 把字典dict2的键/值对更新到dict里
radiansdict.values()  # 返回一个迭代器，可以使用 list() 来转换为列表
pop(key[,default])   #删除字典给定键 key 所对应的值，返回值为被删除的值。key值必须给出。 否则，返回default值。
popitem()   #随机返回并删除字典中的一对键和值(一般删除末尾对)。
```