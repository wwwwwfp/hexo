---
title: 力扣【LeetCode】——70. 爬楼梯【java】
index_img: /img/cover/05.jpg
categories:
  - LeetCode
tags:
  - LeetCode
abbrlink: 2647e9af
date: 2020-11-24 16:18:55
---

题目地址：https://leetcode-cn.com/problems/climbing-stairs/

```text
题目：
假设你正在爬楼梯。需要 n 阶你才能到达楼顶。
每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？
注意：
给定 n 是一个正整数
```
![](0.png)
#### （动态规划）

分析
```text
1种          1 层        1 
2种          2 层        11        2                                         
3种 （2+1）   3 层        111      12    21                                
5种 （3+2）   4 层        1111     112   121   211    22                               
8种 （5+3）   5 层        11111   1112   1121  1211   122    2111   212    221
```
不是那么容易的可以得出第n层的计算公式：

f(n) = f(n-1) + f(n-2) 【斐波那契数列】

所以可以通过递归的方式计算出前面的值，又因为该题计算第n层的时候只需要关注n-1 和 n-2 层的值，所以只需要有三个变量即可完成统计。

```java
public static int climbStairs(int n) {
    // 如下三个参数分别为 n-2  n-1  n 层的种类数
    // 由于第一层只有一种方式（即step_n = 1），并且 f(n) = f(n-2) + f(n-1),所以  step_n_2 和 step_n_1 的初始值为0
    int step_n_2 = 0,step_n_1 = 0,step_n = 1;
    for (int i = 1; i <= n; i++) {
        step_n_2 = step_n_1;
        step_n_1 = step_n;
        step_n = step_n_2 + step_n_1;
    }
    return step_n;
}
```
可以对上面逻辑进行简单的优化

因为上面是定义了三个变量来记录 三个位置的值，我们可以通过一个长度为3的数组存放三个位置的值。

```java
public static int climbStairs(int n) {
        // 思想同上面解法一样，只是用一个变量arr[] 来存放三个值
        int arr[] = new int[]{0, 0, 1};
        for (int i = 1; i <= n; i++) {
        arr[0] = arr[1];
        arr[1] = arr[2];
        arr[2] = arr[0] + arr[1];
        }
        return arr[2];
        }
```


#### 解法二（通项公式）
如下内容摘自官网，至于怎么推算，我已流下了没有好好学习的眼泪 =，=

![](1.png)

知道具体通向公式的话，代码相对来说就比较简单了，如下：

```java
public static int test3(int n){
    double sqrt5 = Math.sqrt(5);
    double temp = Math.pow((1 + sqrt5) / 2, n + 1) - Math.pow((1 - sqrt5) / 2, n + 1);
    return (int) (temp / sqrt5);
}
```

**关于上面代码中 n + 1 的说明**
```text
因为斐波那契数列为 f(1)=1，f(2)=1，f(3)=2，f(4)=3
而这儿爬楼梯数列为 f(1)=1，f(2)=2，f(3)=3， 差一项，所以要+1
```