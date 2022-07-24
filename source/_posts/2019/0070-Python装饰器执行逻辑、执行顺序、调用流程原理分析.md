---
title: Python学习——Python装饰器执行逻辑、执行顺序、调用流程原理分析
index_img: /img/cover/10.jpg
categories:
  - Python
tags:
  - 装饰器
abbrlink: 6a357542
date: 2019-11-25 23:14:15
---

### 装饰器

#### 介绍
是Python的一个重要组成部分，可以有效的增强一个函数的功能。

可以在不修改原函数的情况下对其进行功能扩展，遵循开放封闭原则
#### 举个栗子
要给好多个index函数添加验证，

常规的做法如下，显然违反了封闭原则，并且实施起来也比较困难。
```python
def verification():
    print("验证代码逻辑")
def index1():
    #验证
    verification()
    print("index1....")
def index2():
    #验证
    verification()
    print("index2....")
def indexn():
    #验证
    verification()
    print("indexn....")
```
#### 装饰器初体验
```python
def verify(func):
    def run():
        print("权限验证逻辑")
        func()
    return run
@verify
def index1():
    print("index1....")
@verify
def index2():
    print("index2....")
index1()
index2()
```
#### 运行结果：
```text
权限验证逻辑
index1....
权限验证逻辑
index2....
```
#### 原理分析
以index2()为例

装饰 @verify 实际上等价于 index2 = verify(index2)

前面的index2 表示一个变量，用于指向后面函数的引用，可以用任意变量接收

后面的index2 表示def index2()这个函数引用

通过原函数名index2 接收的原因是为了不影响函数调用

即整个调用逻辑是：
```text
index2 = verify(index2)
index2()   # 所以通过装饰器扩展原函数时，只需要添加装饰器函数即可，不需要更改调用
```
#### 验证
可以将index对应的装饰过程改造如下：
```python
def verify(func):
    def run():
        print("权限验证逻辑")
        func()
    return run

def index2():
    print("index2....")

#调用逻辑如下
index2 = verify(index2)
index2()
#上面两行代码等价于  verify(index2)()
"""
验证结果
    权限验证逻辑
    index2
"""
```
### 多个装饰器的执行顺序

#### 先看一个例子
```python
def verify1(func):
    print("装饰器 1 ...")
    def run():
        print("权限验证逻辑 1 ...")
        func()
        print("111111111111111")
    return run
def verify2(func2):
    print("装饰器2 ...")
    def run2():
        print("权限验证逻辑 2 ...")
        func2()
        print("22222222222222")
    return run2


@verify1
@verify2
def index1():
    print("index1....")

index1()
"""
执行结果
	装饰器2 ...
    装饰器 1 ...
    权限验证逻辑 1 ...
    权限验证逻辑 2 ...
    index1....
    22222222222222
    111111111111111
"""
```
#### 结论
```text
装饰顺序：先装饰，后执行（根据装饰器顺序从下往上执行）
执行顺序：先装饰，先执行（根据装饰器顺序从上往下调用）
```
#### 原理：
```text
实际上是最上面的装饰器装饰的是整个下面的逻辑（装饰器+逻辑）
可以将上述代码稍加改造。用@verify1 装饰器装饰 @verify2装饰的index1的返回结果
```
```python
#代码改造如下 
@verify2
def index1():
    print("index1....")

# 将@verify2 装饰的 index1() 用@verify再装饰
@verify1
def test22():
    return index1()
test22()
```

#### 执行过程分析：
+ 如上面所说 @verify2 装饰的函数等价于 index1 = verify2 (index1)

    这时 将执行 verify2() 函数中的 print(“装饰器2 …”) 这一行语句
    
    接着返回 run2() 函数的引用 , 即index1 指向 run2() 方法
    
    遗留： 参数func 是原来index1 指向的是index1() 这个方法 （先不关注）

    @verify1

    def test22():

    return index1()

+ 在执行到上面的代码块时

    装饰器@verify1 等价于 test22 = verify1(test22)
    
    这时候 将执行 verify1 函数中的 print(“装饰器1 …”) 这一行语句
    
    同时会返回 run() 方法的引用 ， 即 test22 将指向 run() 方法

+ 最后面在调用 test22() 方法时，即调用的是 run() 方法

    这时将打印 run() 方法中的 print(“权限验证逻辑 1 …”) 这一行
    
    接下来执行 run() 方法中的 func() 时， 会调用verify1(test22) 中的参数test22 指向的 test22() 方法,
    
    而test2() 的返回值是 return 后 index1 指向的run2()方法

+ 调用run2 方法时，会先打印 print(“权限验证逻辑 2 …”) 这一行

    执行到 run2 方法中的 func2()时，可以回到原来遗留的那个地方。

    即 调用 func2() 时 实际调用的是 index1() 方法

    这时会打印print(“index1…”) 这一行

    接下来 会将 run2() 方法中后面的代码执行完，即print(“22222222222222”)

    最后会将 run 方法中的 print(“111111111111111”) 这一行指向完

+ 执行完毕

### 装饰器类型

#### 装饰器无参，被装函数无参
上文中的示例中都是这种情况，不再赘述
#### 装饰器无参，被装函数有参数
```python
def verify(func):
    print("=========装饰器 ...")
    def run(uerId,token):
        print("==========权限验证逻辑 ...")
        func(uerId,token)
        print("==========执行完毕")
    return run

@verify
def index(userId,token):
    print("userId : %d   \ntoken: %s"  % (userId,token))

index(1,'dfsdkdsjljdslfjl')
"""
运行结果：
	=========装饰器 ...
	==========权限验证逻辑 ...
	userId : 1   
	token: dfsdkdsjljdslfjl
	==========执行完毕
"""
```
注：在装饰器重，调用被装饰函数的逻辑是调用的装饰器函数中的func() 函数，所以当被装饰函数有参数时，装饰函数中调用的func（）函数也要添加对应的参数。如果不加会报如下错误
```text
Traceback (most recent call last):
=========装饰器 ...
  File "D:/dev/workspace_pycharm/decorator/verification2.py", line 13, in <module>
    index(1,'dfsdkdsjljdslfjl')
==========权限验证逻辑 ...
  File "D:/dev/workspace_pycharm/decorator/verification2.py", line 5, in run
    func()
TypeError: index() missing 2 required positional arguments: 'userId' and 'token'
```
#### 装饰器无参，被装函数有不定长参数
```python
def verify(func):
    print("=========装饰器 ...")
    def run(*args,**kwargs):
        print("==========权限验证逻辑 ...")
        func(*args,**kwargs)
        print("==========执行完毕")
    return run

@verify
def index(userId,account,**token):
    print(userId)
    print(account)
    print(token)

index(1,"chase",name="chase",pwd="******",token="sdfsdf")

"""
运行结果
	=========装饰器 ...
	==========权限验证逻辑 ...
	1
	chase
	{'name': 'chase', 'pwd': '******', 'token': 'sdfsdf'}
	==========执行完毕
"""
```
#### 装饰器中的 return
```python
def verify(func):
    print("=========装饰器 ...")
    def run(*args,**kwargs):
        print("==========权限验证逻辑 ...")
        return func(*args,**kwargs)
        print("==========执行完毕")
    return run

@verify
def index(userId,account,**token):
    print(userId)
    print(account)
    print(token)
    return "indexxxxxxx"

result = index(1,"chase",name="chase",pwd="******",token="sdfsdf")

print(result)
"""
执行结果
	=========装饰器 ...
	==========权限验证逻辑 ...
	1
	chase
	{'name': 'chase', 'pwd': '******', 'token': 'sdfsdf'}
	indexxxxxxx   
"""
```
注：只需要将 原来装饰器稍作修改即可，return func()

所以一般情况下为了让装饰器更通用，都添加return

#### 装饰器带参数
在原有装饰器的基础上，设置外部变量
```python
def verify_param(sysId=22):
    print("111111 %d" % sysId)
    def verify(func):
        print("222222 %d" % sysId)
        print("=========装饰器 ...")
        def run(*args,**kwargs):
            print("333333 %d" % sysId)
            print("==========权限验证逻辑 ...")
            return func(*args,**kwargs)
            print("444444 %d" % sysId)  # 前面已经return，所以不执行
            print("==========执行完毕")
        return run

    print("555555 %d" % sysId)
    return verify

@verify_param(100)
def index(userId,account,**token):
    print(userId)
    print(account)
    print(token)
    return "indexxxxxxx"

result = index(1,"chase",name="chase",pwd="******",token="sdfsdf")

print(result)
"""
执行结果：
	111111 100
	555555 100
	222222 100
	=========装饰器 ...
	333333 100
	==========权限验证逻辑 ...
	1
	chase
	{'name': 'chase', 'pwd': '******', 'token': 'sdfsdf'}
	indexxxxxxx
"""
```

**执行过程分析**

+ 上面装饰过程可以理解为如下代码

    index = verify_param(100)(index)
    
    index() = verify_param(100)(index)()

+ 执行 print(“111111 %d” % sysId)

    定义 verify(index)
    
    执行 print(“555555 %d” % sysId)
    
    返回 verify(index) 即 index = verify(index)

+ 这时候剩下的执行过程就和前面没有参数的装饰器的执行过程就一致了（略）




