---
title: 力扣【LeetCode】—— 283.零移动【java】
index_img: /img/cover/01.jpg
categories:
  - LeetCode
tags:
  - LeetCode
abbrlink: e750af81
date: 2020-11-24 15:47:49
---

题目地址：https://leetcode-cn.com/problems/move-zeroes/

> 题目：
> 题目：
> 
>  给定一个数组 nums，编写一个函数将所有 0 移动到数组的末尾，同时保持非零元素的相对顺序。
> 
> 示例:
> 
> 输入: [0,1,0,3,12]
> 
> 输出: [1,3,12,0,0]
> 
> 说明:
> 
>  必须在原数组上操作，不能拷贝额外的数组。
> 
>  尽量减少操作次数。

+ **方案1（两次循环）**

  思路：统计所有非0数字的个数，统计的过程中依次将非0的数字存放到数组，最后填充0即可

  ```java
  class Solution {
      public void moveZeroes(int[] nums) {
          int j=0;
          for(int i=0;i<nums.length;i++){
              if(nums[i] != 0){ 
                  //如果不等于0，则依次存储在j的下标
                  // j++ 方便后面补0
                  nums[j] = nums[i];
                  j++;  // ++可以直接放在上面下标中
              }
          }
          // 补 0
          for(int i=j;i<nums.length;i++){
              nums[i] = 0;
          }
      }
  }
  ```

+ **方案2（一次循环）:**

  思路：快慢指针,slow 指针从前往后遍历，找到0后停下来，fast 依次遍历，如果找到非0的值，并且slow 和fast 不是同步的时候（即fast>slow），说明slow 位置一定为0 ，替换两个值即可，最后slow ++（因为原来0的位置已经替换成非0数字了）

  ```java
  class Solution {
      public static int[] moveZeroes2(int[] arrays) {
        int slow = 0;  // 慢指针
        for (int fast = 0; fast < arrays.length;fast++) {
            if (arrays[fast] != 0) {
                if (fast > slow) {
                    int temp = arrays[fast];     // 替换位置
                    arrays[fast] = arrays[slow];
                    arrays[slow] = temp;
                }
                slow ++; 
            }
        }
        return arrays;
      }
  }
  ```