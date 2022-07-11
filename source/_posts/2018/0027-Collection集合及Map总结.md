---
title: Collection 集合 及 Map 总结
index_img: /img/cover/27.jpg
categories:
  - [Java基础]
  - [集合]
tags:
  - Map
  - Collection
abbrlink: b2c7f784
date: 2018-08-26 23:35:35
---

### Collection
+ 各自的原理、特点及区别都在图中有相应的体现。
  ![](1.png)

### Map
+ HashMap是基于哈希表的Map接口的非同步实现。在JAVA编程中，最基本的结构就是两种。一个是数组，另一个是模拟指针（引用），所有的数据结构都可以用这两个基本结构来构造的，HashMap也不例外。HashMap实际上是一个“链表数组”的数据结构，每个元素存放链表节点的数组，即数组和链表的结合体。HashMap底层就是一个数组结构，数组中的每一项又是一个链表。当新建一个HashMap的时候，就会初始化一个数组。Entry就是数组中的元素，每个Map.Entry其实就是一个key-value对，它持有一个向下元素的引用，这就构成了链表。
+ HashMap的存储。当我们往HashMap中put元素的时候，先根据key的hashCode会重新计算hash值，根据hash值得到这个元素在数组中的位置（即下标），如果数组该位置上已经存放又其他元素了，那么这个位置上的元素将以链表的形式存放，新加入的放在链表头，最先加入的放在链尾。如果数组在该位置上没有元素，就直接将该元素放到此数组中的该位置上。
+ HashMap的读取。从HashMap中get元素时，首先计算key的hashCode，找到数组中对应位置的某一元素，然后通过key的eques方法在该位置的链表中找到需要的元素。
+ HashMap中的resize（rehash）。当HashMap中的元素越来越多的时候，hash冲突的几率也就越来越高，因为数组的长度是固定的。所以为了提高查询的效率，就要对HahsMap的数组进行扩容，数组扩容这个操作也会出现在ArrayList中。这是一个常用的操作，而在HashMap数组扩容之后，最消耗性能的点就出现了：数组中的数据必须重新计算在新数组中的位置，并放进去，这就是resize。那么HashMap 什么时候进行扩容呢？当HashMap中的元素个数超过数组大小*loadFactor时就会进行数组扩容，loadFactor默认值时 0.75，这是一个折中的值。也就是说默认情况下数组大小为16，那么当HashaMap中元素个数超过 16*0.75=12 （临界值）的时候，就把这个数组的大小扩展为2*32，即扩大一倍，然后重新计算每个元素在数组中的位置，而这是一个非常消耗性能的操作，所以我们如果已经预知HashMap中的个数，那么预设元素的个数能搞有效的提高HashMap的性能。
+ 与HashTable的区别：HahsMap没有排序，允许一个null键和多个null值，而Hashatable不允许。HashMap把Hashtable的contains方法去掉了，改成containsvalue和containsKey，因为contains方法容易让人引起误解。Hashtable继承自Dictionary类，HashMap是java1.2引进的Map接口的实现。Hashtable的方法是线程安全的，而HashMap不是，在多个线程访问Hashtable时，不需要自己为它的方法实现同步，而HashMap就必须为之提供外同步。Hashtable和HashMap采用的has/rehash算法大致一样，所以性能不会有很大差异。
