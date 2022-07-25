---
title: 力扣【LeetCode】——1. 两数之和【java】
index_img: /img/cover/03.jpg
categories:
  - LeetCode
tags:
  - LeetCode
abbrlink: b4fb84e0
date: 2020-11-24 16:17:52
---

题目链接：https://leetcode-cn.com/problems/two-sum/

```text
题目
 给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。
说明：
 可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。
示例
 给定 nums = [2, 7, 11, 15], target = 9
 因为 nums[0] + nums[1] = 2 + 7 = 9
 所以返回 [0, 1]
```

#### 解法一（暴力枚举）

这也是开始只能想到的一个解法

```java
// 暴力求解，枚举遍历
public int[] twoSum(int[] nums, int target) {
    for (int i = 0; i < nums.length; i++) {
        for (int j = i+1; j < nums.length; j++) {
            if (nums[i] + nums[j] == target) {
                return new int[]{i,j};
            }
        }
    }
    return null;
}
```
时间复杂度：O(N^2)，空间复杂度：O(1)

#### 解法二（hash表）

思路：通过一个hash表记录数组的元素和下标（key保存数组，value保存元素下标），因为题目中说明了“可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍 ”，所以不需要考虑hash碰撞问题。

因为key保存的是数组的元素，所以通过一直的target - x（当前遍历的元素）的到 y，判断y是否在 hash表中，如果找到 ，则 y 对应的value 和 x 对应的下标 index 就是要返回的两个索引

参考如下图示，图片来源：https://leetcode-cn.com/problems/two-sum/solution/jie-suan-fa-1-liang-shu-zhi-he-by-guanpengchn/

![](1.gif)

具体代码如下：

```java
public int[] twoSum1(int[] nums, int target) {
	  Map<Integer, Integer> map = new HashMap<Integer, Integer>();
	  for (int i = 0; i < nums.length; i++) {
	      if (map.containsKey(target - nums[i])) {
	          return new int[]{map.get(target-nums[i]), i};
	      } else {
	          map.put(nums[i], i);
	      }
	  }
	  return null;
}
```

时间复杂度：O(n)，空间复杂度：O(N)、主要是hash 表的开销
