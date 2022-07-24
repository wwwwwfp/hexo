---
title: HashMap源码（六）—— 扩容 resize()，和获取 get()
index_img: /img/cover/19.jpg
categories:
  - - Java基础
  - - HashMap
tags:
  - HashMap
abbrlink: 6fb89e28
date: 2020-06-01 18:21:22
---
  
### 扩容 resize()
#### 扩容机制
当HashMap中的元素个数超过数组大小(数组长度) * loadFactor(负载因子)时，就会进行数组扩容，loadFactor的默认值(DEFAULT_LOAD_FACTOR)是0.75,这是一个折中的取值。也就是说，默认情况下，数组大小为16，那么当HashMap中的元素个数超过16×0.75=12(这个值就是阈值或者边界值threshold值)的时候，就把数组的大小扩展为2×16=32，即扩大一倍，然后重新计算每个元素在数组中的位置，而这是一个非常耗性能的操作，所以如果我们已经预知HashMap中元素的个数，那么预知元素的个数能够有效的提高HashMap的性能。

当HashMap中的其中一个链表的对象个数如果达到了8个，此时如果数组长度没有达到64，那么HashMap会先扩容解决，如果已经达到了64，那么这个链表会变成红黑树，节点类型由Node变成TreeNode类型。当然，如果映射关系被移除后，下次执行resize方法时判断树的节点个数低于6，也会再把树转换为链表。
#### HashMap 扩容重新分配位置原理
在进行扩容的时候，会伴随着一次hash的重新分配，并且会遍历hash表中所有的元素，是非常耗时的，所以在编写程序的时候要尽量避免resize().

而在HashMap在进行扩容的时候，重新计算hash的方式非常的巧妙。因为每次扩容是翻倍操作，也就是原来的容量*2，并且因为计算index值得公式为 (n-1)&hash, hash值不变，影响的只是n的2倍。所以n 的二进制扩容后就是在原来的基础上向左移动了 1为 也就是说 扩容后的 2n-1 的二进制有效位比原来的多一个1 （如：原来n-1的二进制为1111，扩容后则是11111）。所以与相同的hash与计算后，index要么在原来的位置要么是 原来位置+原来的容量值

#### 分析：
![](1.png)

#### 结论：
在HashMap进行扩容的时候不需要重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就可以，如果是0，则索引没变 如果是1则索引变成原来位置+旧容量

#### 源码分析
```java
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table; //当前table
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold; //当前阈值 默认16*0.75 = 12
        int newCap, newThr = 0;
        if (oldCap > 0) {
        	// 超过最大值就不再扩充了
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 计算新的容量和新的临界值    分别左移 1 位
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; //新的边界值= 旧的边界值 * 2
        } else if (oldThr > 0) //map创建后第一次扩容，老阈值赋值给新的数组长度
        	//oldThr = threshold  = this.threshold = tableSizeFor(initialCapacity); 
        	//初始化时计算的这个值就等于容量，并不是容量 * 0.75
            newCap = oldThr; 
        else {  // 直接使用默认值，调用无参构造， oldThr 为0 ，设置默认的容量和阈值
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        // 计算新的resize最大上限
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        // 根据扩容后的参数，创建新的数组并赋值
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
            	// 遍历hash表中的每一个桶，重新计算桶中元素的新位置
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                    	// 没有下一个引用，说明只有一个键值对，直接插入
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                    	// 红黑处理逻辑，调用split()把树拆分开
                    	// 该方法会在下面详细说明
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { 
                    	// 链表结构处理逻辑
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            //判断扩容后的位置是在原来位置还是需要+旧数组容量
                            // (e.hash & oldCap) = 0 为true ，e这个节点在resize后不需要移动位置
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            // 原索引+oldCap
                            } else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        // 原索引放到bucket里
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        // 原索引+oldCap放到bucket里
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab; //返回新的table
    }
```
split() 方法说明, ((TreeNode<K,V>)e).split(this, newTab, j, oldCap)
```java
 /** 
  * map 需要扩容的hashmap
  * tab 新创建的数组
  * index 旧数组的索引
  * bit 就数组的容量
  */
  final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
            //做个赋值，因为这里是((TreeNode<K,V>)e)这个对象调用split()方法，所以this就是指(TreeNode<K,V>)e对象，所以才能类型对应赋值
            TreeNode<K,V> b = this;
            //设置低位首节点和低位尾节点
            TreeNode<K,V> loHead = null, loTail = null;
            //设置高位首节点和高位尾节点
            TreeNode<K,V> hiHead = null, hiTail = null;
            //定义两个变量lc和hc，初始值为0，后面比较要用，他们的大小决定了红黑树是否要转回链表
            int lc = 0, hc = 0;
            //这个for循环就是对从e节点开始对整个红黑树做遍历
            for (TreeNode<K,V> e = b, next; e != null; e = next) {
                //取e的下一节点赋值给next遍历
                next = (TreeNode<K,V>)e.next;
                //取好e的下一节点后，把它赋值为空，方便GC回收
                e.next = null;
                //以下的操作就是做个按位与运算，按照结果拉出两条链表，具体的操作可以参考这篇博客@2
                if ((e.hash & bit) == 0) {
                    if ((e.prev = loTail) == null)
                        loHead = e;
                    else
                        loTail.next = e;
                    loTail = e;
                    //做个计数，看下拉出低位链表下会有几个元素
                    ++lc;
                }
                else {
                    if ((e.prev = hiTail) == null)
                        hiHead = e;
                    else
                        hiTail.next = e;
                    hiTail = e;
                    //做个计数，看下拉出高位链表下会有几个元素
                    ++hc;
                }
            }
            //如果低位链表首节点不为null，说明有这个链表存在
            if (loHead != null) {
                //如果链表下的元素小于等于6
                if (lc <= UNTREEIFY_THRESHOLD)
                    //那就从红黑树转链表了，低位链表，迁移到新数组中下标不变，还是等于原数组到下标
                    tab[index] = loHead.untreeify(map);
                else {
                    //低位链表，迁移到新数组中下标不变，还是等于原数组到下标，把低位链表整个拉到这个下标下，做个赋值
                    tab[index] = loHead;
                    //如果高位首节点不为空，说明原来的红黑树已经被拆分成两个链表了
                    if (hiHead != null)
                        //那么就需要构建新的红黑树了
                        loHead.treeify(tab);
                }
            }
            //如果高位链表首节点不为null，说明有这个链表存在
            if (hiHead != null) {
                 //如果链表下的元素小于等于6
                if (hc <= UNTREEIFY_THRESHOLD)
                    //那就从红黑树转链表了，高位链表，迁移到新数组中的下标=【旧数组+旧数组长度】
                    tab[index + bit] = hiHead.untreeify(map);
                else {
                    //高位链表，迁移到新数组中的下标=【旧数组+旧数组长度】，把高位链表整个拉到这个新下标下，做赋值
                    tab[index + bit] = hiHead;
                    如果低位首节点不为空，说明原来的红黑树已经被拆分成两个链表了
                    if (loHead != null)
                        //那么就需要构建新的红黑树了
                        hiHead.treeify(tab);
                }
            }
        }

```


### 获取 get()
#### 入口方法 get
```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```
#### getNode() 方法
```java
/**
 * 实现步骤
 * 1）通过hash值获取该key映射到的桶
 * 2）桶上的key就是要查找的key,则直接找到并返回
 * 3）桶上的key不是要找的key,则查看后续的节点：
 * 		a:如果后续节点是红黑树节点，通过调用红黑树的方法根据key获取value
 * 		b:如果后续节点是链表节点，则通过循环遍历链表根据key获取value
 */
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    //如果哈希表不为空并且key对应的桶上不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        /* 
        	判断数组元素是否相等
        	根据索引的位置检查第一个元素
        	注意：总是检查第一个元素
        */
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 如果不是第一个元素，判断是否有后续节点
        if ((e = first.next) != null) {
            // 判断是否是红黑树，是的话调用红黑树中的getTreeNode方法获取节点
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                // 不是红黑树的话，那就是链表结构了，通过循环的方法判断链表中是否存在该key
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}

//红黑数的调用逻辑处理
final TreeNode<K,V> getTreeNode(int h, Object k) {
            return ((parent != null) ? root() : this).find(h, k, null);
 }
 final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
            TreeNode<K,V> p = this;
            do {
                int ph, dir; K pk;
                TreeNode<K,V> pl = p.left, pr = p.right, q;
                if ((ph = p.hash) > h)
                    p = pl;
                else if (ph < h)
                    p = pr;
                else if ((pk = p.key) == k || (k != null && k.equals(pk)))
                    return p;//找到之后直接返回
                else if (pl == null)
                    p = pr;
                else if (pr == null)
                    p = pl;
                else if ((kc != null ||
                          (kc = comparableClassFor(k)) != null) &&
                         (dir = compareComparables(kc, k, pk)) != 0)
                    p = (dir < 0) ? pl : pr;
                //递归查找
                else if ((q = pr.find(h, k, kc)) != null)
                    return q;
                else
                    p = pl;
            } while (p != null);
            return null;
        }
```