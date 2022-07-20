---
title: Python闭包、Java闭包、JS闭包学习
index_img: /img/cover/05.jpg
categories:
  - - Java基础
  - - Python
  - - JS
tags:
  - 闭包
abbrlink: 6cd4b971
date: 2019-10-11 11:20:41
---
  
### 概念
> 闭包概念——摘自百度百科
>> 闭包就是能够读取其他函数内部变量的函数。例如在javascript中，只有函数内部的子函数才能读取局部变量，所以闭包可以理解成“定义在一个函数内部的函数“。在本质上，闭包是将函数内部和函数外部连接起来的桥梁。
>>
>>闭包包含自由（未绑定到特定对象）变量，这些变量不是在这个代码块内或者任何全局上下文中定义的，而是在定义代码块的环境中定义（局部变量）。“闭包” 一词来源于以下两者的结合：要执行的代码块（由于自由变量被包含在代码块中，这些自由变量以及它们引用的对象没有被释放）和为自由变量提供绑定的计算环境（作用域）。在PHP、Scala、Scheme、Common Lisp、Smalltalk、Groovy、JavaScript、Ruby、 Python、Go、Lua、objective c、swift 以及Java（Java8及以上）等语言中都能找到对闭包不同程度的支持。
个人理解：内部函数将整个函数体返回给外部函数

### 作用：
+ 外部函数的生命周期完成后，不会回收掉它所占用的资源，仍然保存在内容中
+ 读取函数内部的变量
### 优点：
+ 变量长期保存在内容中
+ 避免全局变量的污染
### 缺点：
+ 占用内存，使用不当会导致内存泄漏
### Python 闭包：
示例：
```python
def outFun(price,discount):
    def innerFun(count):
        return price * discount * count
    return innerFun
amount = outFun(10,0.5)  #单价10元  折扣5折
print(amount(1))   # 5.0
print(amount(2))  #10.0

''' 
注：python中闭包调用 return 的方法名是不带括号的
原因：因为outFun 返回的是 整个innerFun 方法体
当执行完amount = outFun(10,0.5)  初始化变量后，会将整个innerFun 函数作为返回值
所以这个时候 amount 的地址是指向 innerFun(count) 这个函数
这就是  amount(1) 这一行代码这么使用的原因
'''
```
**说明：**

该示例中，当outFun函数调用完成后，不会释放掉他的资源，在第二次调用的时候更改数量count值后，依然会拿到对应的变量。对于内部函数而言，外部函数的局部变量对于它来说可以理解成“全局变量”。当外部函数的生命周期结束后，会返回内部函数的整个函数体，如果内部函数中引用到了外部函数的临时变量，会把该临时变量绑定到外部函数中。所以在外部函数的调用周期接受后，内部函数依然可以使用这个临时变量。这就是不会释放内存的原因。

**如何在闭包中修改外部函数的变量**

在内部函数中给要修改的外部变量前加 nonlocal 关键字即可

### Java闭包：
在java中，闭包是通过接口和内部类实现的
```java
// Action.java
public interface Action {
    void run();
}
```
```java
// Closure.java
public class Closure {
    List<Action> list = new ArrayList<>();
    public void input(){
        for (int i = 0; i < 10; i++) {
            //闭包共享变量
            final Integer copy = i;
            list.add(new Action() {
                @Override
                public void run() {
                    System.out.print(copy +",");
                }
            });
        }
    }

    public void output(){
        for (Action a: list) {
            a.run();
        }
    }

    public static void main(String[] args) {
        Closure sc = new Closure();
        sc.input();   
        sc.output();   //0,1,2,3,4,5,6,7,8,9,
    }
}
```

### Js 闭包：
参考：http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html
```js
//说明可参考 python闭包
function f1(){
    var n=999;
    nAdd=function(){n+=1}
    function f2(){
　　　　　	alert(n);
　　　　	}
　　　　	return f2;
}
var result=f1();
result(); // 999
nAdd();
result(); // 100
```

