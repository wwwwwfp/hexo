---
title: 力扣【LeetCode】——15.三数之和【java】
index_img: /img/cover/02.jpg
categories:
  - LeetCode
tags:
  - LeetCode
abbrlink: 5d09b1
date: 2020-11-24 16:17:20
---

题目链接：https://leetcode-cn.com/problems/3sum/

```text
题目：
 给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有满足条件且不重复的三元组。
注意：
 答案中不可以包含重复的三元组。
示例：
 给定数组 nums = [-1, 0, 1, 2, -1, -4]，
 返回：[[-1, 0, 1],[-1, -1, 2]]
```

该题目的难点是如何保证不重复，因为题目中要求“不可以包含重复的三元组”

如：原始数组 nums = [1, 0, 1, 2, -1, -4] 中，依次穷举结果会出现 [1,0,-1] 和 [0,1,-1]的结果集，这显然不合理

所以在解题之前将原始数组进行排序操作，这样在遍历的途中 如果 nums[i-1] = nums[i] 时，跳过即可。


所以、二话没说，和原来题目一样，上来一顿暴力解答，执行并提交。不过这次玩出事了。

如下是问题代码

```java
// 暴力求解代码 ，验证没通过
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> lists = new ArrayList<>();
        Arrays.sort(nums);
        for (int i = 0; i < nums.length; i++) {
            if (i > 0 && nums[i] == nums[i - 1]) continue;
            for (int j = i + 1; j < nums.length; j++) {
                if (j > i+1 &&  nums[j] == nums[j - 1]) continue;
                for (int k = j + 1; k < nums.length; k++) {
                    if (k > j+1 &&  nums[k] == nums[k - 1]) continue;
                    if (nums[i] + nums[j] + nums[k] == 0) {
                        List<Integer> cur = new ArrayList<>();
                        cur.add(nums[i]);
                        cur.add(nums[j]);
                        cur.add(nums[k]);
                        lists.add(cur);
                    }
                }
            }
        }
        return lists;
    }
}
```

力扣官网提交结果：
![](1.png)
![](2.png)

So，暴力不可以解决所有问题！！！

偷得解题思路、跌跌撞撞编完码（排序+双指针）

```java
/**
 * i  当前元素下标
 * l  左指针元素下标
 * r  右指针元素下标
 * @param nums
 * @return
 */
public List<List<Integer>> threeSum2(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    Arrays.sort(nums);
    for (int i = 0; i < nums.length; i++) {
        if (nums[i] > 0) break;  // 升序后的数组，第一个元素 > 0 直接返回即可
        if (i > 0 && nums[i] == nums[i - 1]) continue;  // 相邻数字一样时，遍历一次即可
        for (int l = i + 1,r = nums.length-1; l < r;) {
            if (nums[l] + nums[r] == - nums[i]) {
                List<Integer> list = new ArrayList<>();
                list.add(nums[i]);
                list.add(nums[l++]);  // 满足条件后，左指针后移，右指针左移
                list.add(nums[r--]);
                result.add(list);
                while (l < nums.length && nums[l] == nums[l - 1]) l++;  // 左指针移动后判断是否和前一个元素相等，相等继续后移动
                while (r > l && nums[r] == nums[r + 1]) r--; // 右指针同理
            } else if (nums[l] + nums[r] > -nums[i]) {
                r--;  // > -nums[i]  说明  l r 位置对应的元素之和 太大，只有把 r 向左移才会减小
            } else {
                l++;  // 同理
            }

        }
    }
    return result;
}
```

参考官方解法（和上面本质一样，是用while遍历的，可能更容易理解一点）

```java
public List<List<Integer>> threeSum3(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    Arrays.sort(nums);
    for (int i = 0; i < nums.length; i++) {
        if (nums[i] > 0) break; // 升序后的数组，第一个元素 > 0 直接返回即可
        if (i > 0 && nums[i] == nums[i - 1]) continue; // 相邻数字一样时，遍历一次即可
        int l = i +1,r = nums.length-1;
        while (l < r) {
            if (nums[l] + nums[r] == -nums[i]) {
                List<Integer> list = new ArrayList<>();
                list.add(nums[i]);
                list.add(nums[l++]);
                list.add(nums[r--]);
                result.add(list);
                while (l<nums.length && nums[l] == nums[l-1]) l++;
                while (r > l && nums[r] == nums[r+1]) r--;
            } else if (nums[l] + nums[r] > - nums[i]) {
                r--;
            } else {
                l++;
            }
        }
    }
    return result;
}
```
