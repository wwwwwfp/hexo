---
title: HashMap源码（四）—— hashMap 之 put方法详解
index_img: /img/cover/17.jpg
categories:
  - - Java基础
  - - HashMap
tags:
  - HashMap
abbrlink: '9220e391'
date: 2020-05-28 19:38:49
---
### put() 方法
```java
//入口方法
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```
### hash() 方法介绍
入口方法中只调用了一个putVal方法，在调用这个方法之前，调用了hash(key)这个方法，

具体源码如下：
```java
/**
说明，当key为null时返回的hash值为0，是固定的，也说明了hashmap中允许且只有一个为null的key
     而在hashTable中，不允许有null的key存在，所以没有key==null的判断
     当key不为null时，首先会计算出key的hashcode赋值给h，然后与h无符号右移16位后按位异（相同二进制位数字相同为0，不同为1）或后的到hash值
*/
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
问题：为什么要通过 hashcode ^ (hashcode >>> 16) 这么计算hash值，而不直接用hashcode呢？

答：一般来说数组长度很小，假设数组长度n=16（默认值），这样通过hashcode值直接与16-1 进行与操作，实际上只使用了hash值得后4位。hash值得的高位变化很大，低位变化很小，这样就很容易造成hash冲突。因为是进行的与操作，所以这儿吧高低位都利用起来，更好的解决hash冲突的问题

### putVal() 方法
```java
/**
   注：jdk8之后对table数组的创建放在了put方法中  jdk8之前是放在构造方法中的
   //如下是jdk8的无参构造
   public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    } 
 */
/**
 * 参数介绍：
 * hash：hashcode ^ hashcode>>>16  计算的hash值
 * key put保存的键 key
 * value 保存的值 value
 * onlyIfAbsent  如果true代表不更改现有的值
 * evict 如果为false表示table为创建状态
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
        	// 部分扩容逻辑源码，扩容详细逻辑在后面章节
       	    /**
       	    // 首次调用put时通过resize扩容方法初始化 table、threshold 的精简逻辑
       	  	final Node<K,V>[] resize() {
			    Node<K,V>[] oldTab = table; // 初始值null
			    int oldCap = (oldTab == null) ? 0 : oldTab.length;  // 0
			    int oldThr = threshold;  // 0
			    int newCap, newThr = 0;
			    //.................省略首次调用不执行的逻辑.................
			    newCap = DEFAULT_INITIAL_CAPACITY;  //初始容量 16
			    newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY); //初始边界值 12
			    //.................省略首次调用不执行的逻辑.................
			    threshold = newThr;   //初始化边界值12   首次put的时候默认是0
			    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
			    table = newTab;   //首次调用初始化 table 数组
			    //.................省略了一大堆首次put时不执行的逻辑.................
			    return newTab;
			}*/
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
        	// 根据数组长度n-1 和 hash值计算出 数组的索引 i
        	// 如果当前位置的table为null，即没有保存数据的情况下 创建一个node节点，并保存到该数组对应的i位置即可
        	// 由于当前位置保存的事新节点，所以next 的值为null
            tab[i] = newNode(hash, key, value, null);
        else {
        	// 计算出的index位置已经保存了数据的情况下
            Node<K,V> e; K k;
            // p 标识当前索引i位置的 Node节点数据   p是上一个if 参数的逻辑中初始化的
            // 当hash值相等（即hash碰撞） 并且 key相同的情况（即key的hashcode相同）
            // 
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
            	// e 和 p表示同一份东西 即同一个node节点
                e = p;
            else if (p instanceof TreeNode)
            	//如果该node节点保存在结构的情况
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                	//遍历当前桶中的所有节点
                    if ((e = p.next) == null) {
                    	//如果当前索引对应节点的next为null的情况，即当前索引对应的链表中只保存了一个节点
                    	//将当前put的key value 创建新的节点（next 为null），原来节点的next指向该新节点
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) 
                        	//如果超过红黑树节点临界值 8 的时候   将链表结构转化成红黑树
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        //hash 冲突 且key一致，结束for循环  覆盖旧值即可
                        break;
                    p = e;  // 如果next 不等于null 更新p节点  方便上面 （(e = p.next) == null） 判断下一个节点是否是null的逻辑
                }
            }
            // e != null 标识保存的时候产生hash碰撞的情况
            if (e != null) {
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                	//onlyIfAbsent  在put的时候 传入的参数是false
                	//用新值覆盖原来的值
                    e.value = value;
                afterNodeAccess(e); //hashMap中没有做任何处理，linkedHashMap才有效
                return oldValue;   //替换新值得情况下返回旧值作为返回结果
            }
        }
        ++modCount;  //map 修改次数+1
        // size +1   并判断是否大于临界值   如果大于则扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict); //hashMap中没有做任何处理,linkedHashMap才有效
        return null;
    }
```

### 红黑树转换方法 treeifyBin()

说明：

调用红黑树转换方法，不一定就进行扩容操作

如果当前数组的长度>=64 （MIN_TREEIFY_CAPACITY = 64）的情况才会进行红黑树转换，否则只进行扩容操作
```java
/**
 *  tab  当前table数组
 *  hase 当前put值得hash值
 */
final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        //如果 tab为null 或者数组长度 < 64 ,不会进行红黑树转化，只进行扩容操作
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY){
            resize(); 
        /**
         * index = (n - 1) & hash 计算出table 的索引位置
         * e 表示table中index 位置的 Node 节点
         */
        }else if ((e = tab[index = (n - 1) & hash]) != null) { 
        	// hd 红黑树的头节点      tl 红黑树的尾节点
            TreeNode<K,V> hd = null, tl = null;
            do {
            	/**
            		//replacementTreeNode 底层
            		TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
				        return new TreeNode<>(p.hash, p.key, p.value, next);
				    }
				    // TreeNode 结构
				    class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
				        TreeNode<K,V> parent;  
				        TreeNode<K,V> left;
				        TreeNode<K,V> right;
				        TreeNode<K,V> prev; 
				        boolean red; 
				     }
								    
				*/
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null) 
                    hd = p;  //空的时候  将第一个设置成头节点
                else {
                    p.prev = tl; //将原来尾节点设置为该节点的上一个节点
                    tl.next = p; //将原来节点的下一个节点设置成当前结点
                } 
                tl = p;//设置尾节点
            } while ((e = e.next) != null);
            // tab[index] = hd 该逻辑标识将当前已经有前后节点关系的树节点的头节点保存到table的对应位置
            if ((tab[index] = hd) != null)
            	// do  while 循环的作用就是将链表结构中的节点变成有首尾关系的树节点结构
            	// treeify 方法是将当前有关系的树结构变成红黑树结构 
                hd.treeify(tab);
        }
    }
```
### 说明：上面逻辑的主要作用是做了如下结构的变化
![](1.png)
