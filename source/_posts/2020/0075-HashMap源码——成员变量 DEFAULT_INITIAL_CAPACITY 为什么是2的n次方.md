---
title: HashMap源码（二）—— 成员变量 DEFAULT_INITIAL_CAPACITY 为什么是2的n次方？？？
index_img: /img/cover/15.jpg
categories:
  - - Java基础
  - - HashMap
tags:
  - HashMap
abbrlink: d4463b2c
date: 2020-05-27 13:31:05
---

### HashMap 初始化容量
```text
// 默认容量 16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;  //16
```

### 为什么必须是2的n次幂
因为只有是2n,才可以通过 hash & (leng-1) 计算出的索引尽可能保证数据分布均匀.

如果不是2的n次幂，计算出的索引特别容易相同，很容易发生hash碰撞，导致其余数组空间很大程度上没有存储数据，链表或者红黑树过长，效率较低.

#### 说明：为什么是2的n次方？
2的n次幂的二进制是一个首位是1 后面为是0的数，如 2的3次方 二进制为00001000 2的3次方-1 二进制为00000111

hash & (2n-1) 表示 hash值对应的二进制与n个1 做与运算 都为1 则为1 否则为0

只有当 一个数的二进制有效位全是1的情况下 ，不同hash值计算的结果差异性才会更多。

如： 以下结果很明显全为1的情况计算的差异性会更大
```text
  1111111      100000
& 001001      &001001
—————————————————————————
  001001       000000
```
### 如果在实例化的时候传入的值不是2的n次方会怎样？
当实例化HashMap实例时，如果给定number不是2的n次方 initialCapacity(number),由于HashMap的capacity必须是2的次幂，会通过以下方法 （tableSizeFor ） 计算找到大于等于number最小的2n的数。
```java
计算容量分析
HashMap map = new HashMap<>(10)
//获取大于等于传入容量的最小二次幂值 
static final int tableSizeFor(int cap) {
        int n = cap - 1;  // n = 9 
        n |= n >>> 1;     // 即 n = n | n >>> 1; 结果13    
        n |= n >>> 2;	  // n = 15
        n |= n >>> 4;	  // n =  
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```
// 代码分析
```text
cap - 1 的操作是为了规避如果传入的cap值本身就是2的n次方类型的数时的计算结果
           
	cap = 10;     //     计算过程（二进制）
	n = cap-1;            0000 1001   
———————————————————————————————————————————————
	n |= n>>>1;           0000 1101
						| 0000 0100
					   ————————————————————
					      0000 1101
———————————————————————————————————————————————
	n |= n>>>2;           0000 1101
						| 0000 0011
					   ————————————————————
					      0000 1111
———————————————————————————————————————————————
	n |= n>>>4;           0000 1111
						| 0000 0000
					   ————————————————————
					      0000 1111
———————————————————————————————————————————————
	.......
	对于 0000 1111 这个数右移8位和右移16位都已经没有任何意义，最终结果都是0000 1111
———————————————————————————————————————————————
	n = n + 1             0000 1111
						+ 0000 0001
					   ————————————————————
					      0001 0000    =    16  
```
### 结论
以上一系列或运算的目的就是将 cap - 1 的二进制有效位全部变成1

最后再用这些有效位是1的二进制+1 ，就会变成一个大于等于传入容量的最小2的n次幂的数

注：容量最大也就是32bit的正数，因此最后n|n>>>16,最多也是32个1（这是个负数，在执行tableSizeFor之前做了判断，如果大于最大的则取最大值）
```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)    
        initialCapacity = MAXIMUM_CAPACITY;   // 1 << 30
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```
