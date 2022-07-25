---
title: 力扣【LeetCode】—— 26. 排序数组去重、66. 加一【java】
index_img: /img/cover/29.jpg
categories:
  - LeetCode
tags:
  - LeetCode
abbrlink: dcf2e440
date: 2020-11-24 15:46:08
---

### 排序数组去重【双指针】
题目地址：https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/
> **题目：**
给定一个排序数组，你需要在 原地 删除重复出现的元素，使得每个元素只出现一次，返回移除后数组的新长度。
不要使用额外的数组空间，你必须在 原地 修改输入数组 并在使用 O(1) 额外空间的条件下完成。
>
>**示例：**
指定数组nums = [1,1,2] 返回2 nums 前两个元素被修改为 1，2
你不需要考虑数组中超出新长度后面的元素。
> 

注意：该数组为排序好的数组，处理后的数组不需要关注返回数量个数后面的元素

该题目采用的是双指针的思想，用一个指针记录不重复元素的个数（慢指针），另一个指针则用于遍历整个数组（快指针），如果快指针对应的元素和慢指针的不想等，则追加到慢指针后面即可

**具体代码如下**
```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int count = 0;
				// i = 1 第二个元素同第一个元素比较
        for (int i = 1; i < nums.length; i++) {
            if (nums[i] != nums[count]) {
								// 如果不想等，将当前元素追加到 count 位置后面
                nums[++count] = nums[i];
            }
        }
        return count + 1;
    }
}
```

### 加一【取余/9+1】

题目地址：https://leetcode-cn.com/problems/plus-one/

> 给定一个由 整数 组成的 非空 数组所表示的非负整数，在该数的基础上加一。
> 
> 最高位数字存放在数组的首位， 数组中每个元素只存储单个数字。
> 
> 你可以假设除了整数 0 之外，这个整数不会以零开头。

这个题目开始就没看懂

如下是 999 和 929 的简单示例步骤
![](1.png)

```java
public class PlusOne {
    public int[] plusOne2(int[] digits) {
        for (int i = digits.length - 1; i >= 0; i--) {
						// if ((digits[i] + 1) % 10 == 0) { //可以通过对10 取余的方式判断
            if (digits[i] == 9) {
                digits[i] = 0;
            } else {
                digits[i] ++;
                return digits;
            }
        }
        int newArr[] = new int[digits.length + 1];
        newArr[0] = 1;
        return newArr;
    }
}
```