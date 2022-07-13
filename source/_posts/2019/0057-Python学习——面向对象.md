---
title: Python学习——面向对象
index_img: /img/cover/27.jpg
categories:
  - Python
tags:
  - 面向对象
abbrlink: 67d5f522
date: 2019-07-04 09:07:36
---

### 基本特征
```
类(Class): 用来描述具有相同的属性和方法的对象的集合。它定义了该集合中每个对象所共有的属性和方法。对象是类的实例。
方法： 类中定义的函数。
类变量： 类变量在整个实例化的对象中是公用的。类变量定义在类中且在函数体之外。类变量通常不作为实例变量使用。
数据成员： 类变量或者实例变量用于处理类及其实例对象的相关的数据。
方法重写： 如果从父类继承的方法不能满足子类的需求，可以对其进行改写，这个过程叫方法的覆盖（override），也称为方法的重写。
局部变量： 定义在方法中的变量，只作用于当前实例的类。
实例变量： 在类的声明中，属性是用变量来表示的。这种变量就称为实例变量，是在类声明的内部但是在类的其他成员方法之外声明的。
继承： 即一个派生类（derived class）继承基类（base class）的字段和方法。继承也允许把一个派生类的对象作为一个基类对象对待。例如，有这样一个设计：一个Dog类型的对象派生自Animal类，这是模拟"是一个（is-a）"关系（例图，Dog是一个Animal）。
实例化： 创建一个类的实例，类的具体对象。
对象： 通过类定义的数据结构实例。对象包括两个数据成员（类变量和实例变量）和方法。
```

#### 常用内置函数
```
__new__ 创建对象时会自动调用
__init__ 对象初始化时会自动调用
__del__ 对象从内存中销毁时会自动调用
__str__ 返回对象的描述信息，print函数输出使用 类似java中重写toString()
```
```python
eg:
class People:
    def __init__(self,name):
        self.name = name
    def __del__(self):
        print("....")
    def __str__(self):  # __str__ 返回的必须是字符串
        return "自定义内容： %s" % self.name

xiaoming = People("xiao ming")
print(xiaoming)   # 自定义内容： xiao ming
```
注：在定义属性时，如果不确定设置什么初始值，可以设置为None

#### 类的定义
```python
class 类名:
    def 方法1(self,参数列表):
        pass
    def 方法2(self,参数列表):
        pass
    ...
```
#### 创建对象
格式：
对象变量 = 类名()

#### 对象的生命周期
``` 
一个对象从调用 类名() 创建，生命周期开始
一个对象的__del__ 方法一旦调用，生命周期结束
在对象的生命周期内，可以访问对象的属性，或让对象调用方法
```
#### self 关键字
``` 
self 代表类的实例，而非类
标识当前调用该函数的对象，即当前对象的引用（默认会传递）
在方法内部，可以通过self.语法访问对象的属性和其他方法
self 的名字并不是规定死的，也可以使用 this，但是最好还是按照约定是用 self
self 表示声明的 对象
```
```python
eg:  #继承下面有说明
class A:
	def test(self):
		print("A test")
		self.run()
	def run(self):
		print("A  run")
class B:
	def run(self):  
		print("B run")
class C(B,A):
	pass
obj = C()
obj.test()
```
``` 
上面结果为：
A test
B run
注：因为方法调用时通过类.方法调用，self表示声明的对象，且子类拥有父类的所有属性和方法，
所以在test 中 调用run方法中的self 即表示的是 声明的obj
而由于是多继承（下面有相应介绍），且父类有相同方法，所以会根据父类顺序(python中 MRO方法搜索顺序)先找到 B 中的run 方法
```
#### is 与 ==
+ is 判断两个标识符是否引用的同一对象 x is y 类似 id(x) == id(y)
+ is not 判断两个标识符是不是引用不同对象 x is not y 类似 id(x) != id(y)
+ is用于判断两个变量引用是否相同 ==用于判断引用变量的值是否相同
#### 私有属性和私有方法 
+ 在定义属性和方法时，在属性名或方法前增加两个下划线 即可 
+ 私有属性外部不能直接访问 
+ 私有方法外部不能直接调用
```python
eg：
class People:
    def __init__(self):
        self.__name = "xiao ming"
    def __secret(self):
        print("name 是 %s" % self.__name)

xiaoming = People()
print(xiaoming._People__name)
xiaoming._People__secret()
```
#### 伪私有属性和私有方法（不建议使用）
+ 在python中没有真正意义的私有
+ 在给属性方法命名时，实际是对名称做了特殊的处理，使得外界无法访问
+ 处理方法是在名称前加 _类名 即 _类名__方法名或 _类名__属性
```python
eg：
xiaoming = People()
print(xiaoming._People__name)  # xiao ming
xiaoming._People__secret() # name 是 xiao ming
```
### 继承
**概念：** 子类拥有弗雷德所有方法和属性。父类又称子类的基类，子类又称父类的派生类。

**语法：**
```python
class 类名(父类名):
    pass
```
**父类的私有属性和私有方法（用法同java类似）**
+ 子类对象不能再自己的内部方法中直接访问父类的私有属性和私有方法
+ 子类对象可以通过父类的公有方法可以间接访问到父类的私有属性或私有方法

**多继承**
+ 在python语言中是支持多继承的
+ 子类可以拥有多个父类，并且具有所有父类的属性和方法

语法：
```python
class 子类名(父类1,父类2,...):
    pass
```
```
注：
1. 在多继承中，如果父类中如果有相同的方法名，而在子类中使用时未指定，且在子类中找不到该方法时，
python会根据继承父类从左至右的顺序查找，默认调用顺序靠前的父类的方法
2. 在多继承中，如果子类没有构造方法时，子类会根据父类的先后顺序继承父类的构造方法，
默认继承顺序最前且有构造的父类的构造方法
3. 在多继承开发中尽量避免同名方法的这种情况

在python 3.x 定义类时如果密友制定父类，默认会以object 为基类，而在 python 2.x中不会 
所以在变编写代码时，考虑兼容性建议都添加
```
### 重写
```
当父类的方法实现不能满足子类的需求时，可以对方法进行重写
重写父类方法有两种情况
1.覆盖父类方法
   具体实现方式是在子类中定义一个和父类同名的方法并实现
   这样重写运行时只调用子类的方法，不会调用父类方法
2.对父类方法扩展
   具体实现是在子类中重写父类的方法
   需要使用super().父类的方法来调用父类的执行
   在子类中编写子类特有的逻辑
```
### Super 关键字
``` 
super 主要是用来调用父类的方法的
Python 3 和 Python 2 的另一个区别是: Python 3 可以使用直接使用 super().xxx 代替 super(Class, self).xxx :
调用 super() 的时候，实际上是实例化了一个 super 类
参考：https://mozillazg.com/2016/12/python-super-is-not-as-simple-as-you-thought.html
```
### 多态
``` 
当同一个变量在调用同一个方法时，完全可能呈现出多种行为（具体呈现出哪种行为由该变量所引用的对象来决定），这就是所谓的多态
不同的子类对象调用相同的父类方法，产生不同的执行结果。
多态可以增加代码的灵活度，以继承和重写父类方法为前提，是调用方法的技巧，不会影响到类的内部设计
```
```python
eg:
class People(object):
    def __init__(self,name):
        self.name = name
    def pee(self):
        print("%s 撒尿..." % self.name)
    def run(obj):
        obj.pee()
class Man(People):
    def pee(self):
        super(Man, self).pee() # 也可以写super().pee()
        print("站着来...")
class Woman(People):
    def pee(self):
        super().pee()
        print("蹲着来...")
man = Man("小明")
woman = Woman("小花")

man.pee()
woman.pee()

People.run(woman)
```
### 类对象、静态方法
**类属性** 就是类对象中定义的属性用于记录类相关的特征，不会记录具体对象的特征
```python
eg：
class People(object):
	age = 0  # 类属性
```
**类方法** 就是针对类对象定义的方法，在类方法中可以直接访问类属性或调用其它类方法

类方法需要用 @classmethod 标识，告诉解释器是一个类方法，第一个参数 为 cls ，和实例方法中的self 类似， cls 表示当前类的引用
```python
class People(object):
    age = 0
    @classmethod
    def eat(cls):
        cls.age
        pass
```
**静态方法**

静态方法是类中的函数，不需要实例。静态方法主要是用来存放逻辑性的代码，主要是一些逻辑属于类，但是和类本身没有交互，即在静态方法中，不会涉及到类中的方法和属性的操作。可以理解为将静态方法存在此类的名称空间中。与类相关但是不依赖类与实例的方法
静态方法 需要用 @staticmethod 标识，静态方法不能访问实例属性，参数不能传入self 
```python
class People(object):
    def __init__(self,name):
        self.name = name
    age = 0
    @classmethod
    def eat(cls):
        cls.age
        pass
    @staticmethod
    def run():
        pass
```

**属性方法区别**

**属性**
```
实例属性
  __init__(self,…)中初始化 或 创建对象时初始化
  内部调用时都需要加上self.外部调用时用instancename.propertyname
类属性
  __init__(self,…)外初始化
  在内部用classname.类属性名调用. 外部既可以用classname.类属性名又可以instancename.类属性名来调用
私有属性：
  1）：单下划线_开头：只是告诉别人这是私有属性，外部依然可以访问更改
  2）：双下划线__开头：外部不可通过instancename.propertyname来访问或者更改实际将其转化为了_classname__propertyname
```
**方法**
```
1. 普通类方法：外部用实例调用
2. 静态方法：不能访问实例属性, 参数不能传入self,与类相关但是不依赖类与实例的方法！！
3. 类方法：不能访问实例属性, 参数必须传入cls,必须传入cls参数（即代表了此类对象-----区别------self代表实例对象），并且用此来调用类属性：cls.类属性名
*静态方法与类方法都可以通过类或者实例来调用。其两个的特点都是不能够调用实例属性
```