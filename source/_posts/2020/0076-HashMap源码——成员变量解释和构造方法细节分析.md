---
title: HashMap源码（三）—— 成员变量解释和构造方法细节分析
index_img: /img/cover/16.jpg
categories:
  - - Java基础
  - - HashMap
tags:
  - HashMap
abbrlink: abcd31cc
date: 2020-05-27 18:40:09
---

+ HashMap数组容量，默认是16
    ```java
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
    ```
+ HashMap最大容量：2的30次方
    ```java
    static final int MAXIMUM_CAPACITY = 1 << 30;
    ```
+ 默认负载因子，也叫加载因子 0.75
    ```java
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    ```
  作用：影响着hashMap扩容的因素

  如：默认容量是16，保存的容量超过 16*0.5 =12 的时候就会扩容
+ 链表转红黑树临界值（jdk8之后新增的）
    ```java
    //当链表的值超过8则会转红黑树
    static final int TREEIFY_THRESHOLD = 8;
    ```
  **注：为什么这个节点临界值会选择8呢？**
  
    是因为根据统计学上的泊松分布计算出节点数为8的的概率是0.00000006，
  
    而链表结构转换成红黑树要进行一系列的计算和拷贝且红黑树占用的空间是链表结构的两倍。
    
    所以主要是考虑到时间和空间之前的权衡，8的时候效率和性能上为最佳。
  
    参考：https://blog.csdn.net/wj1314250/article/details/90438488
  
+ 红黑树转链表结构的临界值
    ```java
    //当桶(bucket)上的节点数小于6的时候，则会从红黑树转换成链表结构
    static final int UNTREEIFY_THRESHOLD = 6;
    ```
+ 桶中转化为红黑书对应的数组长度最小的值限制
    ```java
    static final int MIN_TREEIFY_CAPACITY = 64;
    ```
    注：当数组长度大于64并且节链表节点数大于8时才会转换成红黑树结构
  
    而当数组长度小于64链表长度大于8时，map会扩容。
+ 存储元素的数组
    ```java
    // 在jdk8 之前 是 Entry<K,V>[] table
    transient Node<K,V>[] table;
    ```
+ HashMap将数据转换成set的另一种存储形式，这个变量主要用于迭代功能。
    ```java
    transient Set<Map.Entry<K,V>> entrySet;
    ```
+ 存放的的节点个数 不是数组长度
    ```java
    transient int size;
    ```
+ HashMap的数据被修改的次数
    ```java
    //这个变量用于迭代过程中的Fail-Fast机制，
    //其存在的意义在于保证发生了线程安全问题时，能及时的发现（操作前备份的count和当前modCount不相等）并抛出异常终止操作。
    transient int modCount;
    ```
+ 加载因子，默认0.75
    ```java
    // 计算公式 = size / capacity  即:存放个数/数组容量
    final float loadFactor;
    ```
    注：loadFactor太大，导致元素查找效率低，太小会导致数组利用率低，默认0.75是官方给出的一个比较好的临界值
  
    当HashMap里面容纳的元素已经达到HashMap数组长度的75%时，表示HashMap太挤了，需要扩容，而扩容这个过程涉及到 rehash、复制数据等操作，非常消耗性能。，所以开发中尽量减少扩容的次数，可以通过创建HashMap集合对象时指定初始容量来尽量避免。也可以指定loadFactor，不建议修改。 可以通过该构造函数指定初始容量和加载因子，public HashMap(int initialCapacity, float loadFactor)

    **为什么加载因子设置为0.75,初始化临界值是12**

    因为loadFactor越趋近于1，那么 数组中存放的数据(entry)也就越多，也就越密，也就是会让链表的长度增加，loadFactor越小，也就是趋近于0，数组中存放的数据(entry)也就越少，也就越稀疏。所以兼顾数组利用率和链表结构不要太长，经官方大量的测试后，0.75是最佳方案。

+ 扩容临界值
    ```java
    //当实际大小（容量*负载因子）超过这个值时，会进行扩容
    int threshold;
    ```
    注：临界值threshold的计算公式 capacity(数组长度默认16)*0.75.
    
    当size >= threshold 时，就会对数组resize(扩容),也就是说threshold是衡量数组是否扩容改的一个标准，扩容后的map是原来的两倍
+ 构造方法 public HashMap(Map<? extends K, ? extends V> m) 分析
    ```java
    //构造方法，其它几个构造相对简单，不作说明
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
    //具体实现
    final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
        int s = m.size();  //获取参数集合长度
        if (s > 0) {
            if (table == null) {   // 判断table是否初始化，未初始化，即第一次创建时table为null
                float ft = ((float)s / loadFactor) + 1.0F;   //s/loadFactor  计算容量，值为负数 ， +1.0F 作用是向上取整，保证更大容量，减少扩容次数
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
                if (t > threshold)             //初始的时候 边界值是0
                   
                   //初始化临界值
                   //对于这儿初始化的临界值是一个2的n次幂值的原因，下面会做解释
                   //tableSizeFor 方法解释在之前文章中有介绍
                    threshold = tableSizeFor(t);  
            }
            // 已初始化，并且m元素个数大于阈值，进行扩容处理
            else if (s > threshold)
                resize();
             //将m中的元素遍历添加到hashmap中
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);
            }
        }
    }
    ```
    **对于 this.threshold = tableSizeFor(initialCapacity); 疑问解答**
    
    tableSizeFor(initialCapacity) 判断指定的初始化容量是否是2的n次幂，如果不是那么会变为比指定初始化容量大的最小的2的n次幂。这个之前文章有介绍到。
    
    但是注意，在tableSizeFor方法体内部将计算后的数据返回给调用这里了，并且直接赋值给threshold边界值了。
    
    有些人会觉得这里是一个bug,应该这样书写：
    
    this.threshold = tableSizeFor(initialCapacity) * this.loadFactor;
    
    这样才符合threshold的意思（当HashMap的size到达threshold这个阈值时会扩容）。
    
    但是，请注意，在jdk8以后的构造方法中，并没有对table这个成员变量进行初始化，table的初始化被推迟到了put方法中，在put方法中会对threshold重新计算。
    
    
    **float ft = ((float)s / loadFactor) + 1.0F;这一行代码中为什么要加1.0F?**
    
    s/loadFactor的结果是小数，加1.0F与(int)ft相当于是对小数做一个向上取整以尽可能的保证更大容量，更大的容量能够减少resize的调用次数。所以 + 1.0F是为了获取更大的容量。
    
    例如：原来集合的元素个数是6个，那么6/0.75是8，是2的n次幂，那么新的数组大小就是8了。然后原来数组的数据就会存储到长度是8的新的数组中了，这样会导致在存储元素的时候，容量不够，还得继续扩容，那么性能降低了，而如果+1呢，数组长度直接变为16了，这样可以减少数组的扩容。
