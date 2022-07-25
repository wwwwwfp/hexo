---
title: 力扣【LeetCode】——合并有序数组、合并有序链表【java】
index_img: /img/cover/07.jpg
categories:
  - LeetCode
tags:
  - LeetCode
abbrlink: d0e0b821
date: 2020-12-02 09:02:10
---

### 合并有序数组

题目地址： https://leetcode-cn.com/problems/merge-sorted-array/

```text
题目：
给你两个有序整数数组 nums1 和 nums2，请你将 nums2 合并到 nums1 中，使 nums1 成为一个有序数组。
说明：
初始化 nums1 和 nums2 的元素数量分别为 m 和 n 。
你可以假设 nums1 有足够的空间（空间大小大于或等于 m + n）来保存 nums2 中的元素。
示例：
输入：nums1 = [1,2,3,0,0,0], m = 3、nums2 = [2,5,6], n = 3
输出：[1,2,2,3,5,6]
```
#### 解法一：暴力美学（追加+排序）
将nums2 中前n个元素 追加到 nums1 m位置

然后将 前面 m + n 个元素排序
```java
public void merge(int[] nums1, int m, int[] nums2, int n) {
    for (int i = 0; i < n; i++) {
        nums1[m++] = nums2[i];
    }
    // 也可以直接用如下函数copy 到 nums1 中
    // System.arraycopy(nums2, 0, nums1, m, n);
    Arrays.sort(nums1, 0, m);
}
```
时间复杂度 : O((n+m)log(n+m))。

空间复杂度 : O(1)。

#### 解法二：多指针（从前到后）
创建一个新数组temp，比较temp 和 nums2 中元素，因为两个数组是有序的，所以按照顺序，择二者中较小的元素放置到 nums1 中
```java
public void merge(int[] nums1, int m, int[] nums2, int n) {
    // copy 前m个元素到新数组
    int[] temp = new int[m];
    System.arraycopy(nums1, 0, temp, 0, m);
    int num1_i = 0; 
    int temp_i = 0;
    int num2_i = 0;
    while (num1_i < m + n) {
				// 表示nums2 中的元素已经全部放到nums1 中，只需要关注temp中的即可
        if (num2_i >= n) {
            nums1[num1_i++] = temp[temp_i++];
        } else if (temp_i >= m) {
						// temp 中的元素已全部放到nuns1 中，关注 nums2 中
            nums1[num1_i++] = nums2[num2_i++];
        } else {
						// 将较小的 先放到nums1 中
            nums1[num1_i++] = temp[temp_i] < nums2[num2_i] ? temp[temp_i++] : nums2[num2_i++];
        }
    }
}
```
时间复杂度 : O(n+m)。

空间复杂度 : O(m)。

#### 解法三：多指针（从后到前）

由于题目中不考虑nums1 中后面的元素，所以采用覆盖的操作，将 nums1 中m位置后面的元素从大到小依次从后到前设置到nums1 中即可完成合并

```java
// 同样采用三个指针完成解题，last、m、n
public void merge2(int[] nums1, int m, int[] nums2, int n) {
    // last 标识最后一个元素的位置
		int last = m-- + n-- - 1;
    while (n >= 0) {
				// m < 0 说明 nums1 中的元素已经全部移动完毕，只需要关注nums2数组即可
				// 其它情况的思想同上一种一样，比较大小，择大者先放
	      nums1[last--] = m < 0 ? nums2[n--] : nums2[n] > nums1[m] ? nums2[n--] : nums1[m--];
    }
}
```
时间复杂度 : O(n+m)。

空间复杂度 : O(1)。

### 合并有序链表
题目地址：https://leetcode-cn.com/problems/merge-two-sorted-lists/

```text
题目：
将两个升序链表合并为一个新的 升序 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。
示例：
输入：1->2->4, 1->3->4
输出：1->1->2->3->4->4
```
ListNode 类
```java
public class ListNode {
    int val;
    ListNode next;
    ListNode() {}
    ListNode(int val) { 
			this.val = val; 
		}
    ListNode(int val, ListNode next) { 
				this.val = val; this.next = next; 
		}
}
```

#### 解法一：多指针
思路：定义一个头节点 temp，然后依次比较 l1、l2 的头节点，选择较小的一个作为 temp 的next节点（temp = l1 < l2 ? l1 : l2）。最后将非null的 节点追加到 baseNode后面，因为最后是要返回整个链，所以需要前面再定义一个临时节点 baseNode 指向temp 的头节点。最终返回 baseNode的next 对应的链。

```java
public static ListNode mergeTwoLists2(ListNode l1, ListNode l2) {
    if (l1 == null || l2 == null) {
        return l1 == null ? l2 : l1;
    }
    ListNode baseNode = new ListNode(0);
    ListNode temp = baseNode;
    while (l1 != null && l2 != null) {
        if (l1.val < l2.val) {
            temp.next = l1;
            l1 = l1.next;
        } else {
            temp.next = l2;
            l2 = l2.next;
        }
        temp = temp.next;
    }
    temp.next = l1 == null ? l2 : l1;
    return baseNode.next;
}
```
空间复杂度：O(m + n)

时间复杂度：O(1)

#### 解法二：递归
思路：比较 l1 和 l2 , 如果 l1 < l2 ,再通过l1.next 和 l2 比较，直至递归调用完成

```java
public static ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    if (l1 == null || l2 == null) {
        return l2 == null ? l1 : l2;
    }
    if (l1.val < l2.val) {
        l1.next = mergeTwoLists(l1.next, l2);
        return l1;
    } else {
        l2.next = mergeTwoLists(l1, l2.next);
        return l2;
    }
}
```
时间复杂度：O(n+m)

空间复杂度：O(n+m)
