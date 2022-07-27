---
title: 认识Class文件
index_img: /img/cover/12.jpg
categories:
  - Java基础
tags:
  - class
abbrlink: 13ba09a2
date: 2021-06-29 11:27:16
---

### 概念

Class文件是一组以8字节为基础单位的二进制流，各个数据项目严格的按照顺序紧凑地排列在文件之中，中间没有添加任何分割符，这使得整个Class文件存储的内容几乎全部是程序运行的必要数据，没有空隙存在。

《Java虚拟机规范》规定了Class文件格式采用一种类似C语言结构体的伪结构来存储数据，这种伪结构只包含两种数据类型，即无符号数和表。

class文件通过固定的数据结构排列顺序并且每种数据结构指定了占用的字节长度来紧凑的在组成了完整的可读文件，jvm只需要从文件开始的地方一步一步的读取能够完全的解析出这个类文件的内容。

###数据类型：
u1 u2 u4 u8 和 _info(表类型)

u——unsigned 无符号 ，u 后面的数字表示字节数

### ClassFile 查看方式
可以借助很多工具或者插件查看，比如： sublime/notepad等

IDEA 插件 - BinEd

在Idea界面中，按照如下步骤：File → Open as Binary 就可以查看Class文件对应的十六进制内容
![](1.png)

**如下为一个class文件的16进制内容**

![](2.png)

**如下是一个Class文件所表示的含义**
![](3.png)

### 查看ByteCode的方法

1、javap —— jdk 自带

2、JClassLib - IDEA插件（看着方便）

&nbsp;安装插件——鼠标光标放在类体 —— View —— Show Bytecode With jclasslib 会出现如下

General Information 详细信息
![](4.png)

#### General Information 详细信息

![](5.png)

+ Constant pool count ：常量池个数 16
+ Access flags： 0x0021 = ACC_PUBLIC & ACC_SUPER
+ ACC_PUBLIC 0x0001 是否为public类型
+ ACC_SUPER 0x0020 该标记必须为true，jdk1.0.2之后编译出的内容必须为真
+ This class： 当前类的位置，在 Constant Pool（第一个编号是1） 的 2号位置
+ Super class： 父类的位置，在Constant Pool 的第3号位置
+ Interface count: 实现了多少个接口
+ Fields count：属性的个数
+ Methods count：方法个数
+ Attributes count：附加属性个数

#### Constant Pool 详细信息

0 号是预留的，没有任何引用指向这个位置，是从1号开始 的

个数 = constant pool count - 1 = 15 个

**CONSTANT_Methodref_info**

![](6.png)

Class Name 指向 Constant pool的 3号位置

**CONSTANT_NameAndType**

![](7.png)

**CONSTANT_Utf8_info**

![](8.png)

#### Methods 详细信息
![](9.png)

**Methods - Code**

![](10.png)

Code 里面的东西标识方法的实现代码

16进制的Class文件中标识方法的 字节，每一个数字对应一条 java的汇编指令

比如 16进制中 数字 2a 对应的汇编指令时 aload_0

比如 16进制中 数字 2a 对应的汇编指令时 aload_0

每一条汇编质量代表的意思都可以到对应文档中查找意思

文档地址：https://docs.oracle.com/javase/specs/jvms/se14/jvms14.pdf

![](11.png)
![](12.png)

### 总结
```text
JVM在执行一个方法的时候，会从字节码中读取一条指令 如 2a，则再查找对应的汇编指令，即 aload_0，做相应的操作，接下来再读下一条指令 b7，查找到对应的汇编指令 invokespecial （调用构造） ，一直进行类似的操作到结束 到b1指令——return
例如：“2ab7 0001 b1”
这几个字节可以标识一个构造方法的调用过程
2a：aload_0 表示把 this 压栈 扔进去
b7：invokespecial 需要两个参数 00 和 01
01：代表常量池中的第一项java.lang.Object 中的构造方法
即 默认构造调用的是父类Object的构造方法
b1：return用
```