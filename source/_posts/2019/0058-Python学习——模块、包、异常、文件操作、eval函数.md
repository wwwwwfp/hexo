---
title: Python学习——模块、包、异常、文件操作、eval函数
index_img: /img/cover/28.jpg
categories:
  - Python
tags:
  - 模块
  - Eval
abbrlink: 2b9f88cb
date: 2019-07-24 11:38:17
---

### 模块
#### 定义和好处
+ 定义：模块是一个包含所有你定义的函数和变量的文件，其后缀名是.py。模块可以被别的程序引入，以使用该模块中的函数等功能。这也是使用 python 标准库的方法。
+ 优点：使用模块可以提高代码的可维护性和重复使用，还可以避免函数名和变量名冲突。相同名字的函数和变量完全可以分别存在不同的模块中，所以编写自己的模块时，不必考虑名字会与其他模块冲突，但要注意尽量不要与内置函数名字冲突。
#### 模块引入语法
import语句是可以在程序中的任意位置使用的,且针对同一个模块很import多次,为了防止你重复导入，python的优化手段是：第一次导入后就将模块名加载到内存了，后续的import语句仅是对已经加载大内存中的模块对象增加了一次引用，不会重新执行模块内的语句
```text
想使用 Python 源文件，只需在另一个源文件里执行 import 语句，语法如下：
1. import  
   语法：
   import module1[, module2[,... moduleN]
   推荐
   import module1
   import module2
   ...
   这样便于排错
   如果 模块名太长  可以使用 as 关键字重命名
   如：  import module1 as m
2. from...import 导入
	Python 的 from 语句让你从模块中导入一个指定的部分到当前命名空间中，语法如下：
	from modname import name1[, name2[, ... nameN]]
	例如，要导入模块 fibo 的 fib 函数，使用如下语句：
	from fibo import fib, fib2
	这样导入之后，不需要通过模块名.访问，可以直接使用模块提供的工具------函数、类、全局变量等
	如果两个模块存在同名的函数，后倒入模块的函数会覆盖掉先导入的函数
	from...import * 同 import module 作用一样，都是导入所有工具

```
注：Python解释器在导入模块时，首先会搜索当前目录指定模块名的文件，如果有就直接导入。如果没有，再搜索系统目录
所以在开发时应尽量避免和系统模块同名的情况
#### __name__属性
一个模块被另一个程序第一次引入时，其主程序将运行。如果我们想在模块被引入时，模块中的某一程序块不执行，我们可以用__name__属性来使该程序块仅在该模块自身运行时执行。
```python
if __name__ == '__main__':
   print('程序自身在运行') # 如果在当前模块  执行
else:
   print('我来自另一模块') # 如果在其它模块调用  执行
```
说明： 每个模块都有一个__name__属性，当其值是’main’时，表明该模块自身在运行，否则是被引入。
### 包
包是一种管理 Python 模块命名空间的形式，采用"点模块名称"。
在导入一个包的时候，Python 会根据 sys.path 中的目录来寻找这个包中包含的子目录。
目录只有包含一个叫做 __init__.py 的文件才会被认作是一个包，主要是为了避免一些滥俗的名字（比如叫做 string）不小心的影响搜索路径中的有效模块。

最简单的情况，放一个空的 :file:__init__.py就可以了。当然这个文件中也可以包含一些初始化代码或者为 __all__变量赋值。
```text
分层文件目录样例：
sound/                          顶层包
      __init__.py               初始化 sound 包
      formats/                  文件格式转换子包
              __init__.py
              wavread.py
              ...
      effects/                  声音效果子包
              __init__.py
              echo.py
              surround.py
              reverse.py
              ...
```

```text
可以每次只导入一个包里面的特定模块，比如:
import sound.effects.echo
这样必须使用全名去访问
sound.effects.echo.echofilter(input, output, delay=0.7, atten=4)

导入子模块的方法是:
from sound.effects import echo
这同样会导入子模块: echo，并且他不需要那些冗长的前缀，所以他可以这样使用:
echo.echofilter(input, output, delay=0.7, atten=4)

还可以直接导入一个函数或者变量:
from sound.effects.echo import echofilter
这种方法会导入子模块: echo，并且可以直接使用他的 echofilter() 函数:
echofilter(input, output, delay=0.7, atten=4)

相对导入：
点号表示使用当前路径的database模块
    from .database import Database
使用两个点号表示访问上层的父类
    from …database import Database
如果ecommerce有contact包，该包里有email模块，需要将该模块的sendEmail函数导入到paypal模块中，
    from …contact.email import sendEmail
```
### 异常
#### 异常关键字
+ try/except	捕获异常并处理
+ pass	忽略异常
+ as	定义异常实例（except MyError as e）
+ else	如果try中的语句没有引发异常，则执行else中的语句
+ finally	无论是否出现异常，都执行的代码
+ raise	抛出/引发异常
#### 语法
```python
try:
	raise MyError('error message') # 手动抛出异常
except exception1：
	pass
except (exception2,exception2,...) as e:
	pass
except Exception as e:
	pass
else:
	pass
finally:
	pass
```
#### 自定义异常
需要继承Exception 类
```python
# 如：
class MyError(Exception):
        def __init__(self, value):
            self.value = value
        def __str__(self):
            return repr(self.value)
```
### 文件读写操作
#### 件操作步骤
+ 打开文件
+ 读写文件（将文件内容读入内存和将内存内容写入文件）
+ 关闭文件
#### 操作文件的函数方法
+ open，打开文件，并返回文件对象
+ read，将文件内容读到内存
+ write，将内容写入文件
+ close，关闭文件
```python
# eg:
import io
file = open("test")
text = file.read()
print(text)
file.close()
```
#### 文件指针
+ 文件指针标记从那个位置开始读取数据
+ 第一次挡开文件是，通常文件指针会只想文件的初始位置
+ 当执行了read方法后，文件指针会移动到读取内容的末尾
+ 如果执行了一次read方法读取了所有内容，再次执行read时由于指针在末尾位置，则将读取不到内容
#### 打开文件的方式
open 函数默认打开文件的方式为只读方式

语法： file = open(“文件名”,“访问方式”)

访问方式	说明 如下：
+ r	: 以 只读 的方式打开文件，默认模式。如果文件不存在则会抛出异常
+ w	: 以 只写 的方式打开文件，如果文件存在，会被覆盖，如果不存在创建新文件
+ a	: 以 追加 的方式打开文件，如果文件已存在，文件指针将会放在文件末尾。如果不存在则创建新文件写入
+ r+	: 以 读写 的方式打开文件，文件指针会放在文件的开始位置，如果文件不存在则抛出异常
+ w+	: 以 读写 的方式打开文件， 如果文件存在会被覆盖，如果不存在则创建新文件
+ a+	: 以 读写 的方式打开文件，如果文件存在，文件指针将会放在文件的结尾，如果文件不存在，创建文件进行写入
#### 按行读写文件内容
read会将文件内容一次性读入到内存，如果文件太大可以 通过readline 方法读，这样一次只读取一行内容
```python
# eg:
import io
file = open("test")
while True:
    text = file.readline()
    print(text,end="")
    if not text:
        break
file.close()
```
### 文件目录操作
在python中如果要实现对文件目录的操作等，需要导入os模块

文件或者目录操作都支持相对路径和绝对路径
#### 文件操作
| 方法名 | 说明     | 示例                    |
|--------|-----------------------| --- |
| rename	| 重命名文件	 | os.rename(源文件名，目标文件名) |
|remove | 	删除文件	 | os.remove(文件名)        |
#### 目录操作
| 方法名 | 说明                          | 示例                   |
|--------|-----------------------------|----------------------|
|listdir	| 目录列表	                       | os.listdir(目录名)      |
|mkdir	| 创建目录	                       | os.mkdir(目录名)        |
|rmdir	| 删除目录	                       | os.rmdir(目录名)        |
|getcwd	| 获取当前目录	                     | os.getcwd()          |
|chdir	| 修改工作目录	                     | os.chdir(目标目录)       |
|path.isdir	| 判断是否是文件| 	os.path.isdir(文件路径) |
### eval 函数
将字符串当作有效的表达式来求值并返回计算结果
```python
# eg:
# 基本数学计算
print(eval("1+2+3+4")) # 10
# 字符串重复
print(eval("'st'*3")) # ststst
#将字符串转换成列表
print(type(eval("[1,2,3,4,6]"))) # <class 'list'>
#将字符串转换成字典
print(type(eval("{'name':'ttt','age':22}"))) # <class 'dict'>
#转换空字符串
print(eval('9.9\n\t\r  \t\r\n')) #9.9
#变量计算
x = 1
y = 2
print(eval("x+y")) # 3
```
eval 函数使用注意事项
参考：https://blog.csdn.net/zoulonglong/article/details/80446373