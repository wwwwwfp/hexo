---
title: Python学习——流程控制语句、函数定义
index_img: /img/cover/24.jpg
categories:
  - Python
tags:
  - Python
abbrlink: 721c0ba
date: 2019-06-09 23:26:15
---
### 分支语句 if

+ 单分支语法格式
   ```python
   if 条件:
       满足条件的执行代码
   ```

+ 双分支语法格式：
   ```python
   if 条件:
       满足条件的执行代码
       ...
   else :
       条件不满足时执行逻辑
   ```
+ 多分支语法格式：
   ```python
   if 条件1:
       满足条件一的执行代码
       ...
   elif 条件2:
       满足条件二的执行代码
       ...
   elif 条件3:
       ...
   else :
       以上条件都不满足时执行逻辑
   ```
### while循环
+ 基本循环和死循环语法格式：
   ```python
   while 判断条件:
       代码块
       
   while True:
       print("这是一个死循环")
   ```
+ while…else语法格式：
   ```python
   while 判断条件:
       代码块
   else:
       代码块
   ```
### for循环
+ 基本语法格式
   ```python
   for 临时变量 in 可迭代对象:
       代码块
   # 注：可迭代对象可以时list，也可以通过索引进行迭代
   ```
+ for…eles 语法格式
   ```python
   同while…eles 用法基本一致
   ```
### 循环控制语句
+ break

  终止整个循环 例如：如果时在for…eles 的for块中，如果for循环正常结束，else中语句执行。如果是break的，则不执行。
+ continue

  跳过本次循环，执行下一次循环
+ pass

  pass语句是个空语句，只是为了保持程序结构的完整性，没有什么特殊含义。pass语句并不是只能用于循环语句中，也可以用于分支语句中。

### Python 函数定义
#### 定义一个函数
以下是简单的规则：

函数代码块以 def 关键词开头，后接函数标识符名称和圆括号()。

任何传入参数和自变量必须放在圆括号中间。圆括号之间可以用于定义参数。

函数的第一行语句可以选择性地使用文档字符串—用于存放函数说明。

函数内容以冒号起始，并且缩进。

return [表达式] 结束函数，选择性地返回一个值给调用方。不带表达式的return相当于返回 None。

语法：
```python
def functionname( parameters ):
   "函数_文档字符串"
   function_suite
   return [expression]
```
注：如果不主动调用函数，函数时不会执行的。