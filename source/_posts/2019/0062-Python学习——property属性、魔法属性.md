---
title: Python学习——Property属性、魔法属性
index_img: /img/cover/02.jpg
categories:
  - Python
tags:
  - property
  - 魔法属性
abbrlink: 5974a3dd
date: 2019-08-29 16:12:55
---
### property属性
#### property属性作用
通过使用property属性，能够简化调用者在获取数据的流程
#### property属性的两种方式
+ 装饰器方式：在方法使用@property、@price.setter、@price.deleter 等
+ 类属型方式：在类中定义值为property对象的类属性
##### 装饰器方式
```text
在经典类中只有一种property装饰器：@property
在新式类(继承object)中有三种property装饰器：@property、@xxx.setter  @xxx.deleter
```
###### 示例代码
````python
class Person(object):
    def __init__(self):
        self.name = "张三"

    @property
    def nickname(self):
        print(self.name)

    @nickname.setter
    def nickname(self,val):
        self.name = val

    @nickname.deleter
    def nickname(self):
        del self.name

obj = Person();
obj.nickname     # 结果：张三    --- 通过@property可以将name方法以属性的形式调用
obj.nickname = "xiao qiang"   # ---通过@方法名.setter 将val值以属性赋值的方式通过方法设置给name
obj.nickname    # 结果：小强

obj.name
print(obj.__dict__)  # 结果：{'name': 'xiao qiang'}
del obj.nickname   # --- 通过@方法名.deleter，将name属性按照del属性的方式删除
print(obj.__dict__)  # 结果：{}
````
##### 类属性方式
```text
使用类属型方式创建property属性时，经典类和新式类无区别
在property方法中有四个参数
1.第一个参数是方法名：调用 对象.属性 时自动触发执行方法
2.第二个参数是方法名：调用 对象.属性 = xxx 时自动触发执行方法
3.第三个参数是方法名：调用 del 对象.属性 时自动触发执行方法
4.第四个参数是字符串：调用 对象.属性.__doc__ ，此参数是该属性的描述信息
```
###### 示例代码
```python
class Person(object):
    def __init__(self):
        self.name = "张三"

    def getName(self):
        print(self.name)

    def setName(self,val):
        self.name = val

    def delName(self):
        del self.name

    nickname = property(getName,setName,delName,"昵称...")

obj = Person();
obj.nickname     # 结果：张三    --- 调用property中第一个参数中的getName方法
obj.nickname = "xiao qiang"   # ---调用property中的第二个参数中的setName方法
obj.nickname    # 结果：小强

print(obj.__dict__)  # 结果：{'name': 'xiao qiang'}
del obj.nickname   # --- 调用property中的第三个参数中的delName方法
print(obj.__dict__)  # 结果：{}

description = Person.nickname.__doc__   # 获取property中第四个参数
print(description)  # 结果：...
```
### 魔法属性
Python中，具有特殊含义的类属型
#### 1. \_\_doc\_\_
获取类的描述信息
```python
class Foo(object):
    """这是描述信息"""
    def run(self):
        pass

print(Foo.__doc__) # 这是描述信息
```
#### 2. \_\_module\_\_ 、 \_\_class\_\_
+ __ module __ 表示当前操作的对象在那个模块

+ __ class __ 表示当前操作的对象的类是什么
```python
test.py
class Person(object):
    def __init__(self):
        pass
        
main.py
from test import Person
obj = Person()
print(obj.__module__)  # test
print(obj.__class__)  # <class 'test.Person'>
```
#### 3. \_\_init\_\_
初始化方法，通过类创建对象时，自动触发执行
```python
class Person(object):
    def __init__(self):
        self.name = "xiao ming"

object = Person()
```
#### 4. \_\_del\_\_
当对象在内存中被释放时，自动触发执行
```text
注：此方法一般无须定义，因为Python是一门高级语言，程序员在使用时无需关心内存的分配和释放，
因为此工作都是交给Python解释器来执行，所以，__del__的调用是由解释器在进行垃圾回收时自动触发执行的。
```
```python
class Person(object):  
    def __del__(self):
        pass
```
#### 5. \_\_call\_\_
对象后面加括号，触发执行
```text
注：__init__方法的执行是由创建对象触发的，即：对象 = 类名() ；
而对于 __call__ 方法的执行是由对象后加括号触发的，即：对象() 或者 类()
```
```python
class Person(object):
    def __call__(self, *args, **kwargs):
        print("call....") # call....
        print(*args) # 1 2
        print(kwargs) # {'name': 'zhang san', 'age': 22}
obj = Person()
obj("1","2", name="zhang san",age = 22)
```
#### 6. \_\_dict\_\_
类或对象中的所有属性
```python
class Person(object):
    country = "china"
    def __init__(self):
        self.name = "zhangsan"
        self.age = 22
obj = Person()

print(Person.__dict__)  # {'__module__': '__main__', 'country': 'china', '__init__': <function Person.__init__ at 0x000000A7F64EAB70>, '__dict__': <attribute '__dict__' of 'Person' objects>, '__weakref__': <attribute '__weakref__' of 'Person' objects>, '__doc__': None}
print(obj.__dict__) # {'name': 'zhangsan', 'age': 22}
```
#### 7. \_\_str\_\_
如果一个类中定义了__str__方法，那么在打印 对象 时，默认输出该方法的返回值
```python
class Person(object):
    def __str__(self):
        return "str..........."
obj = Person()
print(obj) #str...........
```
#### 8. \_\_getitem\_\_、\_\_setitem\_\_、\_\_delitem\_\_
用于索引操作，如字典。以上分别表示获取、设置、删除数据
```python
class Course(object):
    def __init__(self):
        self.language = {"python":"最好用的语言","php":"世界上最好的语言"}

    def __getitem__(self, key):
        print(self.language[key])

    def __setitem__(self, key, value):
        self.language[key] = value

    def __delitem__(self, key):
        del self.language[key]

obj = Course()
obj['python']         # 自动触发执行 __getitem__      结果：最好用的语言 
obj['c'] = 'c很牛逼'   # 自动触发执行 __setitem__
obj["c"]              # c很牛逼
obj["php"]       #世界上最好的语言
del obj["php"]   # 自动触发执行 __delitem__
obj["php"]       # 报错 KeyError: 'php
```
#### 9. 等等
