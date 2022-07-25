---
title: 力扣【LeetCode】——环形链表（判断是否有环）、环形链表（返回入环节点）【java】
index_img: /img/cover/08.jpg
categories:
  - LeetCode
tags:
  - LeetCode
abbrlink: 5c9bbaca
date: 2020-12-02 09:02:41
---

### 环形链表（判断是否有环）
题目地址：https://leetcode-cn.com/problems/linked-list-cycle/
```text
题目：
给定一个链表，判断链表中是否有环。
如果链表中有某个节点，可以通过连续跟踪 next 指针再次到达，则链表中存在环。 为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。注意：pos 不作为参数进行传递，仅仅是为了标识链表的实际情况。
如果链表中存在环，则返回 true 。 否则，返回 false 。
进阶：
你能用 O(1)（即，常量）内存解决此问题吗？
提示：
链表中节点的数目范围是 [0, 10000]
-100000 <= Node.val <= 100000
pos 为 -1 或者链表中的一个 有效索引 。
```
![](1.png)

**Node类：**

```java
class ListNode {
    int val;
    ListNode next;
    ListNode(int x) {
        val = x;
        next = null;
    }
}
```
#### 解法一：利用集合判断
思路：依次将节点放到一个Set集合里面，如果添加失败，说明有重复的，反之没有重复的

```java
public static boolean hasCycleBySet(ListNode head) {
    Set<ListNode> set = new HashSet<ListNode>();
    while (head != null) {
        if (!set.add(head)) {
            return true;
        }
        head = head.next;
    }
    return false;
}
```
#### 解法二：快慢指针
思路：设置两个指针，一个一次跳一个节点，另一个跳两个节点，如果第二个节点能追上第一个节点（即两个节点重合），说明有环
```java
public static boolean hasCycleByFastSlowPointer(ListNode head) {
    if (head == null || head.next == null) {
        return false;
    }
    ListNode slow = head;
    ListNode fast = head.next;
    while (slow != fast) {
        if (fast == null || fast.next == null) {
            return false;
        }
        slow = slow.next;
        fast = fast.next.next;
    }
    return true;
}
```
#### 解法三：计数——投机取巧
思路：因为题目中有提示 链表中节点的数目范围是 0 - 10000，如果遍历整个链表的节点，遍历次数超过10000，就说明有环。

```java
public static boolean hasCycleByCountNumber(ListNode head) {
    int count = 0;
    while (head != null) {
        head = head.next;
        if (++count > 10000) {
            return true;
        }
    }
    return false;
}
```
#### 解法四：环形链表反转后的头节点和原始链接的头节点相等 (参考思路)

```java
public static boolean hasCycleByReverseList(ListNode head) {
    if (head != null && head.next != null && head == reverseList(head)) {
        return true;
    }
    return false;
}
public static ListNode reverseList(ListNode head) {
    ListNode pre = null;
    while (head != null) {
        ListNode next = head.next;
        head.next = pre;
        pre = head;
        head = next;
    }
    return pre;
}
```
#### 解法五：会破坏链表结构，将每一个指针的next指向下自己，这样在遍历过程中，如果找到 current.next = current 的情况下就表示有环。
![](2.png)
```java
public static boolean hasCycleByRemoveNextReference(ListNode head) {
    if (head == null || head.next == null) {
        return false;
    }
    ListNode next = null;
    while (head != null) {
        next = head.next;
        if (head == next) {
	            return true;
        }
        head.next = head;
        head = next;
    }
    return false;
}
```


### 环形链表（返回入环节点）
题目链接：https://leetcode-cn.com/problems/linked-list-cycle-ii/

```text
题目：
给定一个链表，返回链表开始入环的第一个节点。 如果链表无环，则返回 null。
为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。注意，pos 仅仅是用于标识环的情况，并不会作为参数传递到函数中。
说明：
不允许修改给定的链表。
进阶：
你是否可以使用 O(1) 空间解决此题？
```

#### 解法一：集合处理，相对较简单
思路：通过Hash表处理，同上一个题目的解法一致，

```java
/**
 * Hash 表解题
 * @param head
 * @return
 */
public ListNode detectCycleByHash(ListNode head) {
    Set<ListNode> set = new HashSet<>();
    while (head != null) {
        if (set.contains(head)) {
            return head;
        } else {
            set.add(head);
            head = head.next;
        }
    }
    return null;
}
```

#### 解法二：快慢指针
思路：通过公式可以证明，相遇点到入口点的节点数 = 头节点到入口点的节点数，所以相遇点和头节点一起向后遍历，如果相等则就是入口点。

证明过程：图片内容来自力扣网站
![](3.png)

```java
/**
 * 利用公式
 * @param head
 * @return
 */
public ListNode detectCycleByDoublePointer(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;
    while (slow != null && fast.next != null && fast.next.next != null) {
        slow = slow.next;
        fast = fast.next.next;
		// 找到相遇点
        if (slow == fast) {
				// 开始遍历，找入环的节点位置
            while (slow != head) {
                slow = slow.next;
                head = head.next;
            }
            return slow;
        }
    }
    return null;
}
```
