---
title: Python学习—— 基础入门篇 变量类型、逻辑运算（3.X）
index_img: /img/cover/23.jpg
categories:
  - Python
tags:
  - Python
abbrlink: ff1c286e
date: 2019-06-09 21:50:22
---

#### Pycharm更改解释器设置，其它基本设置同IntelliJ IDEA大概一致

```
File -> Setting -> Project -> Project Interpreter     选择更改即可
```
#### 命名规范
1. 建议使用小写字母、数字和下划线
2. 文件名不能以数字开始
#### 注释

1. 单行注释

   程序上方： # 注释内容 注：#号后留一空格
   程序右侧： # 注释内容 注：程序和注释间至少留两个空格
2. 多行注释（块注释）

   “”"注释内容“”"
#### 编码规范

参考：https://zh-google-styleguide.readthedocs.io/en/latest/google-python-styleguide/python_style_rules/

注：Python 对于编码的规范时比较严格的，在Python开发中，代码的缩进为一个Tab键或4个空格（官方建议四个空格），Tab和空格不可混用。

#### 算术运算符
```
   +        -        *        /        //        %        **
  /  ： 除， 如 10/20 = 0.5 同java有区别
  //  ：取整数 9//2=4 同java中 / 一致
  %  ：取余数
  **   ：幂 2**3=8
  *  ： 在python中 * 号可以用于字符串用于计算字符串重复出现次数 如：“SS”*2 =“SSSS”
```

#### python程序执行原理
1. 操作系统首先会让CPU把python解释器的程序复制到内存中
2. python解释器根据语法规则，从上向下让CPU翻译python程序中的代码
3. cpu负责执行翻译完成的代码

#### 变量类型
在Python中定义变量类型是不需要制定类型的，数据类型可以分为数字型和非数字型，在允许Python时，Python解释器会根据赋值语句等号右侧的数据自动推导出保存数据的准确类型。

1. 数字型

    整型（int) 注：在Python 2.X中整型根据长度不同分为 int 和 long

    浮点型（float）

    布尔型（bool) : True/Fales 非0 即真

    附属型（complex）：主要用于科学计算
2. 非数字型
   字符串

   列表

   元组

   字典

#### Type() 函数
作用：查看变量类型

如： name = “张三”

type(name) # str 类型

#### input() 函数
作用：控制台通过键盘输入数据保存到变量中

如：name = input(“请输入姓名”)；

#### 变量的格式化输出
```
提供了“%”对各种类型的数据进行格式化输出，对应格式如下：
%s 字符串 (采用str()的显示)
%r 字符串 (采用repr()的显示)
%c 单个字符
%b 二进制整数
%d 十进制整数
%i 十进制整数
%o 八进制整数
%x 十六进制整数
%e 指数 (基底写为e)
%E 指数 (基底写为E)
%f 浮点数
%F 浮点数，与上相同
%g 指数(e)或浮点数 (根据显示长度)
%G 指数(E)或浮点数 (根据显示长度)
```
例如：
```
user = "Charli"
age = 8
# 格式化字符串有两个占位符，第三部分提供2个变量
print("%s is a %s years old boy" % (user , age)) # Charli is a 8 years old boy
```

#### 比较（关系）运算符
```
== 相等
!= 不相等 注：在python2.X中可以使用 <> 来判断不等于
 > 大于
 < 小于
 >= 大于等于
 <= 小于等于
```

#### 逻辑运算
```
and 且
or 或
not 非
```