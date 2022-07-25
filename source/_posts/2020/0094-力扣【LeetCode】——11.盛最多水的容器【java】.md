---
title: 力扣【LeetCode】——11. 盛最多水的容器【java】
index_img: /img/cover/04.jpg
categories:
  - LeetCode
tags:
  - LeetCode
abbrlink: c9fe24d0
date: 2020-11-24 16:18:31
---

题目地址：https://leetcode-cn.com/problems/container-with-most-water/

```text
题目：
 给你 n 个非负整数 a1，a2，…，an，每个数代表坐标中的一个点 (i, ai) 。在坐标内画 n 条垂直线，垂直线 i 的两个端点分别为 (i, ai) 和 (i, 0) 。找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。
说明：
 不能倾斜容器。
示例：
```
![](1.png)

#### 解法一（暴力求解）：
思路：遍历计算最大值，两层循环，选中一条垂直线，然后根据这条线依次遍历后面的，根据最短的线计算面积，获取最大值即可

```java
public class ContainerWithMostWater {
    // 暴力求解
    public static int maxArea(int[] height) {
        int area = 0;
        for (int i = 0; i < height.length; i++) {
            // j 从 i位置后面一个开始
            for (int j = i + 1; j <height.length; j++) {
            	// Math.min(height[i],height[j]) * (j-i) 计算面积
            	// 和原来计算结果比较大小 ，择大即可
                area = Math.max(area,Math.min(height[i],height[j]) * (j-i));
            }
        }
        return area;
    }
    public static void main(String[] args) {
        int[] arr = new int[]{4,3,2,1,4};
        System.out.println(maxArea(arr));
    }
}
```

#### 解法二（双指针移动）— 参考官网：
思路：首先用第一根和最后一根做比较，可以计算出矮的那根对应的最大面积（装水只能装到矮的那个的高度），同时可以在后面的计算中排除矮的这根，这样依次按照同样的方式从两边向中间靠拢，并计算面积，找到最大值即可。

示例图（来源：leetcode官网）
![](2.gif)

##### 写法一（参考官网）：
```java
/**
 * 参考官方解法（双指针）
 */
public int maxArea3(int[] height) {
    int area = 0;
    int i = 0,j = height.length-1;
    while (i < j) {
        int minH = Math.min(height[i],height[j]);
        area = Math.max(area,minH * (j-i));
        if (height[i] < height[j]) {
            i ++;
        }else {
            j --;
        }
    }
    return area;
}
```
##### 另一种写法

到网站翻到一种牛掰写法（原理和上一种解法一致）

```java
/**
 * 到国外网站翻到了一种牛掰写法（原理和上一种解法一致）
 */   
public int maxArea(int[] height) {
    int area = 0;
    for (int i = 0,j = height.length-1; i<j;) {
        int minH = height[i] < height[j] ? height[i++] : height[j--];
        //j-i加1的原因是上面i++/j--的时候向右或向左移动了一格，方便下次遍历
        //所以在进行当前计算的时候要还原到原来横坐标的宽度
        area = Math.max(area, minH * (j - i +1));
    }
    return area;
}
```
上一种写法进行简单的优化
```java
// 再优化，将j-i的操作提到 i++ j-- 操作的前面
public int maxArea(int[] height) {
    int area = 0;
    for (int i = 0,j = height.length-1; i<j;)
        area = Math.max((j-i)* (height[i] < height[j]?height[i++]:height[j--]),area);
    return area;
}
```
参考如上写法，对第二种解法简单进行简单改造

后面几种的写法思路、本质都是一样，只是写法有些不同而已

```java
public static int maxArea4(int[] height) {
    int area = 0;
    int i = 0,j = height.length-1;
    while (i < j) {
        int minH = height[i] < height[j] ? height[i++] : height[j--];
        area = Math.max(area,minH*(j-i+1));
    }
    return area;
}
```
