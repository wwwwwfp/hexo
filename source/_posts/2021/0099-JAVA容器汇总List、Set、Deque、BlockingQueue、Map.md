---
title: JAVA容器汇总（原理+使用）——List、Set、Deque、BlockingQueue、Map (部分源码)
index_img: /img/cover/09.jpg
categories:
  - - Java基础
  - - 集合
tags:
  - 集合
abbrlink: be4f3323
date: 2021-03-11 19:46:11
---

## List 接口
### ArrayList
底层是通过Arrays.copyof()和System.arraycopy()操作的。

非线程安全，如果需要保证线程安全，后面会介绍其它相关的容器

底层是通过数组实现，内部通过一个Object对象数组来存放元素

默认容量大小 10 ，不指定初始容量的时，第一次add操作的时候，会初始化容量为10.

扩容后的大小是原来的1.5倍数：newCapacity = oldCapacity + (oldCapacity >> 1)

扩容会有一个将旧数组数据拷贝到新数组数据的过程，所以如果知道具体数据量的情况下，指定好容量

在指定位置存放元素的时候，会右移数组index右边位置的所有元素

删除index位置元素的时候，如果index后面有元素，则全部左移动，保证连续

注：在ArrayList中维护了一个int类型的属性modCount，标识操作的次数

当在进行遍历的过程中，如果list发生了修改操作，会抛ConcurrentModificationException的异常
```java
public void forEach(Consumer<? super E> action) {
    final int expectedModCount = modCount
    final E[] elementData = (E[]) this.elementData;
    final int size = this.size;
    for (int i=0; modCount == expectedModCount && i < size; i++) {
        action.accept(elementData[i]);
    }
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```
### LinkedList
底层是通过双向链表实现的，内部通过Node next 和 Node prev 来维护链表结构

节点的新增和删除等都是通过 Node前后节点地址指向的变化完成

所以LinkedList插入、删除的效率较高。但是在读取效率方面是不及ArrayList，关于get(index)的部分源码
```java
// 将链表长度一分为二，判断传入的index在那个区间，如果在前面部分，从第一个开始依次遍历
// 如果在后面部分，则从最后一个节点向前遍历查找
// 所以说越中间的元素，查找次数会越多  查找的效率越低
Node<E> node(int index) {
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```
### Vector
Vector 和 ArrayList 高度相似，代码逻辑和ArrayList的差不多

不同于ArrayList的主要有如下几点：

1、Vector 线程安全

相较于ArrayList，Vector在很多影响线程安全的方法（如add、remove、get、size等）上面加了synchronized关键字保证线程安全，以add方法为例
```java
public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}
```
2、扩容容量

ArrayList中，扩容后的数量是原来的1.5倍（newCapacity = oldCapacity + (oldCapacity >> 1)）

Vector 中提供了一个可以多传一个影响扩容数量参数的构造方法，如下：
```java
public Vector(int initialCapacity, int capacityIncrement) {
    super();
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+initialCapacity);
    this.elementData = new Object[initialCapacity];
    this.capacityIncrement = capacityIncrement;
}
```
通过传入的这个参数影响着扩容后的具体数量，相关逻辑如下：
```java
private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
		// 如果 capacityIncrement 是大于0的情况下，扩容后的大小是（原来容量数+capacityIncrement）的值
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ? capacityIncrement : oldCapacity);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```
### Stack
通过源码可以看到Stack是继承与Vector的：
```java
public class Stack<E> extends Vector<E> {
	// 省略
}
```
由于Stack继承了Vector，所以它包含了Vector中全部的API，然后再内部又额外的封装了用于栈操作的一系列方法：

push(E object)、pop()、peek()、empty()、search(Object o)

由于Stack是继承于Vector，并且自己内部封装的影响线程安全的方法都是有synchronized关键字修饰的，所以Stack也是线程安全的一个基于数组实现的容器

empty()：实现原理就是判断数组的size是否为空

push(E object)：
```java
// 入栈  并返回入栈元素
public E push(E item) {
		// addElement 逻辑即Vector 中的添加元素逻辑
    addElement(item);
    return item;
}
```
peek():
```java
// 返回最后一个元素
public synchronized E peek() {
    int     len = size();
    if (len == 0)
        throw new EmptyStackException();
    return elementAt(len - 1);
}
```
pop()：
```java
// 返回最后一个元素并移除
public synchronized E pop() {
    E       obj;
    int     len = size();
    obj = peek();  // 调用peek
    removeElementAt(len - 1);  // 父类 Vector 中逻辑
    return obj;
}
```
empty()：
```java
// 判空
public boolean empty() {
    return size() == 0;
}
```
search(Object o)
```java
// 查找元素o在栈的位置
public synchronized int search(Object o) {
    int i = lastIndexOf(o);  // 父类 Vector 中的逻辑
    if (i >= 0) {
        return size() - i;
    }
    return -1;
}
```

### CopyOnWriteArrayList
前面的Vector 是读写的时候都加锁的，其实在有些情况下，读的时候是不需要加锁操作的。CopyOnWrite可以很好的避免这个问题。

CopyOnWrite 读的时候不加锁，只有写操作的时候加锁

应用场景：高并发情况下 写操作明显少于都操作的场景下可以使用CopyOnWrite提高效率

原理：

在进行写操作的时候，会将原来的数据Copy一份，然后写操作在这个新的List上操作，读操作还是原来的list，写操作完成之后改变原来List的引用到当前新的List。

写操作部分代码如下：
```java
// CopyOnWriteArrayList 源码：
public boolean add(E e) {
		// 18 之前和1.8之后的区别是  锁机制不同
		// 1.8 之前是 synchronized   1.8之后是 ReentrantLock
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray(); 
        int len = elements.length;  
				// copy 原来的数组（len+1），将添加的元素追加到copy的这个list后面  
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);  // 将原来数组的引用指向新的引用
        return true;
    } finally {
        lock.unlock();
    }
}

//读的时候不需要枷锁
//因为复制了一份，对原来的没影响
```
## Set 接口
### CopyOnWriteArraySet
CopyOnWriteArraySet 和 CopyOnWriteArrayList 的实现原理基本一致，无非就是set不允许重复的区别，在CopyOnWriteArraySet 多了一个校验元素是否存在的逻辑

部分原理参考上面CopyOnWriteArrayList

简单分析add方法和CopyOnWriteArrayList的不同之处：
```java
// CopyOnWriteArraySet 构造方法实际是创建了一个CopyOnWriteArrayList
private final CopyOnWriteArrayList<E> al;
public CopyOnWriteArraySet() {
    al = new CopyOnWriteArrayList<E>();
}
// add 方法
public boolean add(E e) {
    return al.addIfAbsent(e);
}
// addIfAbsent 方法中添加数据之前进行了一次判断（indexOf）
return indexOf(e, snapshot, 0, snapshot.length) >= 0 ? false :addIfAbsent(e, snapshot);
/**
 * o：add 的元素
 * elements：原数组
 * index  传 0 
 * fence 数组长度
 */
private static int indexOf(Object o, Object[] elements,int index, int fence) {
    if (o == null) {
        for (int i = index; i < fence; i++)
            if (elements[i] == null)
                return i;
    } else {
        for (int i = index; i < fence; i++)
						// 判断是否存在
            if (o.equals(elements[i]))
                return i;
    }
    return -1;
}
```
### HashSet
HashSet 底层实际上是基于HashMap实现的，关于HashSet底层的一些操作，基本都是基于HashMap实现的。具体如下：
```java
// 构造实际是new 了一个hashMap
public HashSet() {
    map = new HashMap<>();   
}
public HashSet(Collection<? extends E> c) {
		// 一次计算出HashMap的容量，防止 c 数据量太多造成多次扩容操作
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}
// 获取HashSet的大小
public int size() { 
		return map.size(); 
}
// add  方法
/**
 * 因为HashSet实际是利用Map的key存数据的，所以不关注map对应的value，
 * 所以 HashSet地层中Map中对应的Key都是 new Object()
 * HashSet设计思想就是利用了Map 中key的不重复性实现的
 * 所以对于HashSet底层的实际操作都是对于HashMap  key的操作
 */
public boolean add(E e) { 
		//PRESENT 的定义：private static final Object PRESENT = new Object();
    return map.put(e, PRESENT)==null; 
}
// 通过判断HashMap中key的存在来判断HashSet是否存在
public boolean contains(Object o) { 
    return map.containsKey(o); 
}
// 删除也是类似，通过移除HashMap中的key 完成对HashSet的删除操作的，细节可以参考后面HashMap
```
注：关于HashMap的细节后面部分会详细介绍。
### LinkedHashSet
查看LinkedHashSet 相关源码，发现该类中只提供了5个方法，其中四个是构造方法，一个迭代器。
```java
public class LinkedHashSet<E> extends HashSet<E> implements Set<E>, Cloneable, java.io.Serializable {
   
    public LinkedHashSet(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor, true);
    }

    public LinkedHashSet(int initialCapacity) {
        super(initialCapacity, .75f, true);
    }

    public LinkedHashSet() {
        super(16, .75f, true);
    }    

    public LinkedHashSet(Collection<? extends E> c) {
        super(Math.max(2*c.size(), 11), .75f, true);
        addAll(c);
    }
		
	 /**
		* 可分割迭代器，主要用于多线程并发处理
		*/
    @Override
    public Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, Spliterator.DISTINCT | Spliterator.ORDERED);
    }
}
```
LinkedHashSet是继承与HashSet的，因为在LinkedHashSet中没有自身的 get()、size()、remove、add() 等相关的增删改查的方法，所以这些方法都是继承父类HahsSet的对应方法的。

差异就是LinkedHashSet底层是通过LinkedHashMap来存储数据的

LinkedHashMap是支持按元素访问顺序遍历元素的，详细原理后面部分有介绍

总结：所以LinkedHashSet 可以根据插入元素的顺序遍历访问的特性就是利用了LinkedHashMap中顺序存储的原理实现的。

而在LinkedHashMap中是支持访问顺序遍历的，但是在LinkedHashSet的构造入口进去之后，实际是调用了如下方法：
```java
public LinkedHashMap(int initialCapacity, float loadFactor) {
    super(initialCapacity, loadFactor);
    accessOrder = false;   // ★★★★★★★
}
```
所以：LinkedHashSet 只可以根据插入数据的顺序进行遍历，二不能根据访问顺序对数据遍历
### SortedSet 接口
SortedSet 是一个接口，它扩展了Set接口，并提供了元素的排序功能

SortedSet 接口中定义的方法如下
```java
// 返回用于对此集合中的元素进行排序的比较器，如果此集合使用其元素的自然顺序，则返回null
Comparator<? super E> comparator();
// 返回此set的子集,[from,to)
SortedSet<E> subSet(E fromElement, E toElement);
// 返回此Set的子集，元素都小于 toElement [first,toElement)
SortedSet<E> headSet(E toElement);
// 返回此Set的子集，元素都大于等于fromElement,[fromElement, last]
SortedSet<E> tailSet(E fromElement);
// 第一个元素（最小元素）
E first();
// 最后一个元素（最大元素）
E last();
// 在此有序集中的元素上创建Spliterator
@Override
default Spliterator<E> spliterator() {
    return new Spliterators.IteratorSpliterator<E>(
            this, Spliterator.DISTINCT | Spliterator.SORTED | Spliterator.ORDERED) {
        @Override
        public Comparator<? super E> getComparator() {
            return SortedSet.this.comparator();
        }
    };
}
```
注：

SortedSet是根据对象的比较顺序排序的，和插入顺序无关，这就是为什么插入到有序集合中的元素必须实现Comparable接口或被指定的Comparator接受的原因了（元素之间必须要有可比性，如 A.compareTo(B )）。

因SortedSet是一个接口，部分细节会在后面它一个实现类TreeSet中介绍

### TreeSet
TreeSet 是 SortedSet的实现类，实现了元素的自动排序（元素大小，非插入顺序），前面SortedSet有相关说明

TreeSet 的底层存储数据是基于 Map 的数据结构实现的，默认是TreeMap，通过其构造可以看到：
```java
//默认构造
public TreeSet() {
    this(new TreeMap<E,Object>());
}
//其它构造方法
TreeSet(NavigableMap<E,Object> m) {    // 用来指定存储数据的map结构类型
    this.m = m;
}
public TreeSet(Comparator<? super E> comparator) {   // 通过自定义比较器，对TreeMap数据惊醒排序
    this(new TreeMap<>(comparator));
}
public TreeSet(Collection<? extends E> c) {  // 传入一个集合，保存到TreeMap数据结构中
    this();
    addAll(c);
}
public TreeSet(SortedSet<E> s) {  // 初始化的时候，传入一个SortedSet，比较器复用SortedSet的
    this(s.comparator());
    addAll(s);
}
```
TreeMap的底层数据结构是通过红黑书实现的，关于TreeMap的数据结构和相关细节在后面TreeMap部分会详细介绍，这儿只记录关于TreeSet相关的细节。

在TreeSet中，因为底层也是Map结构，所以也是利用了map中key的特性，保证存储元素的不重复特点。

在TreeSet中，大部分方法的本质其实是通过调用底层Map的的方法实现的，如下：
```java
public boolean isEmpty() { return m.isEmpty(); }  // 判空
public boolean contains(Object o) { return m.containsKey(o); }  //判断是否包含 o 元素
public boolean remove(Object o) { return m.remove(o)==PRESENT; }  // 移除元素o
public void clear() { m.clear();  }  //清空set
public E first() { return m.firstKey(); }  // 返回第一个元素
public E last() { return m.lastKey(); }   // 返回最后一个元素
public E lower(E e) { return m.lowerKey(e); }  // 返回小于e的最大元素
public E floor(E e) { return m.floorKey(e);} // 返回小于/等于e的最大元素
public E ceiling(E e) { return m.ceilingKey(e);} // 返回大于/等于e的最小元素
public E higher(E e) { return m.higherKey(e);} // 返回大于e的最小元素
private void readObject(java.io.ObjectInputStream s)   // 通过流读数据
private void writeObject(java.io.ObjectOutputStream s)  // 通过流写数据
/**
 * 添加元素  
 * 实质是Map的put 方法，key为add的元素，value 为 null
 */
public boolean add(E e) {    
    return m.put(e, PRESENT)==null;
}
// 获取第一个元素，并删除
public E pollFirst() {
    Map.Entry<E,?> e = m.pollFirstEntry();
    return (e == null) ? null : e.getKey();
}
// 获取最后一个元素，并删除
public E pollLast() {
    Map.Entry<E,?> e = m.pollLastEntry();
    return (e == null) ? null : e.getKey();
}
//返回一个子集合  from ----- to 区间的子集，是否包含输入值，通过其它两个参数控制
public NavigableSet<E> subSet(E fromElement, boolean fromInclusive, E toElement,   boolean toInclusive) {
    return new TreeSet<>(m.subMap(fromElement, fromInclusive, toElement,   toInclusive));
}
//返回set的头部区间， 范围：[first——toElement)  是否包含toElement，由参数inclusive 决定
public NavigableSet<E> headSet(E toElement, boolean inclusive) {
    return new TreeSet<>(m.headMap(toElement, inclusive));
}

//返回set的尾部区间， 范围：(fromElement——last]  是否包含fromElement，由参数inclusive 决定
public NavigableSet<E> tailSet(E fromElement, boolean inclusive) {
    return new TreeSet<>(m.tailMap(fromElement, inclusive));
}
```
总结：因为TreeSet底层数据结构是红黑书，因为红黑树在操作节点的时候会存在节点颜色的变换和旋转操作，所以在效率方面可能会弱一点，如果不需要保证顺序的前提下，建议优先使用HashSet。
### EnumSet
EnumSet 是为枚举类型设计的Set，是一个抽象类

EnumSet 在某些场景下可以简化开发和易于维护某些功能，在很多场景下会对枚举常量进行操作，可以通过EnumSet转换成对Set的操作，简单的使用实例：
```java
//（一个穷屌丝的一生）
public class PoorBoy {
		// 一生应该有的家当
    enum TreasureEnum{
        HOUSE, CAR, WIFE, MONEY
    }
    public static void main(String[] args) {
        // 创建一个包含所有Enum元素的 Set
        // （含着金钥匙出生）
        // EnumSet<TreasureEnum> all = EnumSet.allOf(TreasureEnum.class);
        // System.out.println(all);  // [HOUSE, CAR, WIFE, MONEY]
        
        // 创建一个空的EnumSet
        // （一个穷屌丝的一生）
        EnumSet<TreasureEnum> poorBoy = EnumSet.noneOf(TreasureEnum.class);
        // 结婚前要先购置房子，拥有了房子
        poorBoy.add(TreasureEnum.HOUSE);
        // 可以结婚了,拥有了妻子
        poorBoy.add(TreasureEnum.WIFE);
        // 一起挣钱
        poorBoy.add(TreasureEnum.MONEY);
        // 挣钱了买了一辆车子
        poorBoy.add(TreasureEnum.CAR);
        System.out.println(poorBoy);   // [HOUSE, CAR, WIFE, MONEY]
        // 买车 花光了所有的钱
        poorBoy.remove(TreasureEnum.MONEY);
        System.out.println(poorBoy);   // [HOUSE, CAR, WIFE]
        // 检查还有钱吗
        System.out.println(poorBoy.contains(TreasureEnum.MONEY));  // false
        // 检查Wife还在吗
        System.out.println(poorBoy.contains(TreasureEnum.WIFE));   // true
    }
}
```
原理分析：

1、EnumSet的主要方法
```java
// 主要方法
// allOf 和 noneOf 后面有介绍
// 初始集合包括枚举值中指定范围的元素
<E extends Enum<E>> EnumSet<E> range(E from, E to)
// 初始集合包括指定集合的补集
<E extends Enum<E>> EnumSet<E> complementOf(EnumSet<E> s)
// 提供了一系列的初始集合包括参数中元素的方法（参数不同） 
<E extends Enum<E>> EnumSet<E> of(E e1, E e2)
<E extends Enum<E>> EnumSet<E> of(E first, E... rest)
```
2、两个实现：RegularEnumSet 和 JumboEnumSet

它有两个实现：RegularEnumSet和JumboEnumSet，具体选择哪个是根据实例化的枚举中常量的数量决定的
```java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
    // 获取枚举
		Enum<?>[] universe = getUniverse(elementType);
    if (universe == null)
        throw new ClassCastException(elementType + " not an enum");
    if (universe.length <= 64)
        return new RegularEnumSet<>(elementType, universe);
    else
        return new JumboEnumSet<>(elementType, universe);
}

// 对比 noneOf 方法 还有一个 allOf
// allOf 就是先通过noneOf 产生一个EnumSet，然后把所有枚举再添加进去
public static <E extends Enum<E>> EnumSet<E> allOf(Class<E> elementType) {
    EnumSet<E> result = noneOf(elementType);
    result.addAll();
    return result;
}
```
为什么要以64为界限？ ———— 后面 RegularEnumSet 和 JumboEnumSet源码介绍

RegularEnumSet和JumboEnumSet 是用protected 修饰的，因为具体选择哪个是内部根据数量来决定的，用户不需要显示的控制和选择具体使用哪一个

3、原理（位向量）

由于EnumSet内部采用了位向量的结构，使得它在表示枚举常量的时候非常紧凑且有效

EnumSet的实现类就是将每一个枚举值映射到一个long类型的变量上（这也就是上面提到的为什么要以64为界限），然后对Enum的操作就是对这个long类型的变量进行操作，这样既省空间，效率也搞。

可以参考：https://blog.csdn.net/tugangkai/article/details/89631886

### ConcurrentSkipListSet
ConcurrentSkipListSet是线程安全的有序的集合，适用于高并发的场景

和前面TreeSet不同的是，虽然TreeSet也是有序集合，但是TreeSet是非线程安全的，而ConcurrentSkipListSet是线程安全的，适合线程安全条件下的并发场景（有序集）

TreeSet的底层实现是通过TreeMap实现的，而ConcurrentSkipListSet的底层实现是ConcurrentSkipListMap，ConcurrentSkipListMap中的元素是K-V，而ConcurrentSkipList是List，所以它只用到了ConcurrentSkipListMap中的Key。

函数列表：
```java
// 构造一个新的空 set，该 set 按照元素的自然顺序对其进行排序。
ConcurrentSkipListSet()
// 构造一个包含指定 collection 中元素的新 set，这个新 set 按照元素的自然顺序对其进行排序。
ConcurrentSkipListSet(Collection<? extends E> c)
// 构造一个新的空 set，该 set 按照指定的比较器对其元素进行排序。
ConcurrentSkipListSet(Comparator<? super E> comparator)
// 构造一个新 set，该 set 所包含的元素与指定的有序 set 包含的元素相同，使用的顺序也相同。
ConcurrentSkipListSet(SortedSet<E> s)
// 如果此 set 中不包含指定元素，则添加指定元素。
boolean add(E e)
// 返回此 set 中大于等于给定元素的最小元素；如果不存在这样的元素，则返回 null。
E ceiling(E e)
// 从此 set 中移除所有元素。
void clear()
// 返回此 ConcurrentSkipListSet 实例的浅表副本。
ConcurrentSkipListSet<E> clone()
// 返回对此 set 中的元素进行排序的比较器；如果此 set 使用其元素的自然顺序，则返回 null。
Comparator<? super E> comparator()
// 如果此 set 包含指定的元素，则返回 true。
boolean contains(Object o)
// 返回在此 set 的元素上以降序进行迭代的迭代器。
Iterator<E> descendingIterator()
// 返回此 set 中所包含元素的逆序视图。
NavigableSet<E> descendingSet()
// 比较指定对象与此 set 的相等性。
boolean equals(Object o)
// 返回此 set 中当前第一个（最低）元素。
E first()
// 返回此 set 中小于等于给定元素的最大元素；如果不存在这样的元素，则返回 null。
E floor(E e)
// 返回此 set 的部分视图，其元素严格小于 toElement。
NavigableSet<E> headSet(E toElement)
// 返回此 set 的部分视图，其元素小于（或等于，如果 inclusive 为 true）toElement。
NavigableSet<E> headSet(E toElement, boolean inclusive)
// 返回此 set 中严格大于给定元素的最小元素；如果不存在这样的元素，则返回 null。
E higher(E e)
// 如果此 set 不包含任何元素，则返回 true。
boolean isEmpty()
// 返回在此 set 的元素上以升序进行迭代的迭代器。
Iterator<E> iterator()
// 返回此 set 中当前最后一个（最高）元素。
E last()
// 返回此 set 中严格小于给定元素的最大元素；如果不存在这样的元素，则返回 null。
E lower(E e)
// 获取并移除第一个（最低）元素；如果此 set 为空，则返回 null。
E pollFirst()
// 获取并移除最后一个（最高）元素；如果此 set 为空，则返回 null。
E pollLast()
// 如果此 set 中存在指定的元素，则将其移除。
boolean remove(Object o)
// 从此 set 中移除包含在指定 collection 中的所有元素。
boolean removeAll(Collection<?> c)
// 返回此 set 中的元素数目。
int size()
// 返回此 set 的部分视图，其元素范围从 fromElement 到 toElement。
NavigableSet<E> subSet(E fromElement, boolean fromInclusive, E toElement, boolean toInclusive)
// 返回此 set 的部分视图，其元素从 fromElement（包括）到 toElement（不包括）。
NavigableSet<E> subSet(E fromElement, E toElement)
// 返回此 set 的部分视图，其元素大于等于 fromElement。
NavigableSet<E> tailSet(E fromElement)
// 返回此 set 的部分视图，其元素大于（或等于，如果 inclusive 为 true）fromElement。
NavigableSet<E> tailSet(E fromElement, boolean inclusive)
```
ConcurrentSkipListSet是通过ConcurrentSkipListMap实现的，它的接口基本上都是通过调用ConcurrentSkipListMap接口来实现的，具体可以参考后面的ConcurrentSkipListMap的介绍
## Map
### HashMap
关于HashMap，在之前的文章分析的还是比较详细的：

请翻阅之前的HashMap 专栏

HashMap 是非线程安全的，Collections中增加了 synchronizedMap 转换方法，其本质也是 synchronized ，只是Collections 中的的同步转换方法的琐粒度比synchronized方法要小一点。所以后来又增加了ConcurrentHashMap

### Hashtable
Hashtable是最初jdk1.0设计的容器，自带锁，其实在很多业务场景中是不需要锁的，所以后来增加了HashMap

Hashtable与HashMap极其相似，两者的区别主要如下：

1、HashMap非同步、线程不安全，而Hashtable的读写操作都是synchronized修饰的，线程安全

2、HashMap中key允许一个null，value可以多个null，Hashtable中的key和value都不能为null，如果设置为null时，编译可以通过，运行时会报空指针。

3、迭代器机制不同，HashMap迭代器为Iterator，支持fail-fast，Hashtable迭代器为enumerator，fail-safe机制

fail-fast：快速失败迭代器，如果集合在遍历的过程中进行了写操作，会抛出异常，因为集合在进行写操作的时候会维护一个修改次数的变量，在迭代之前会保存这个值，如果在修改的过程中修改了，则两个值不一致，这时就会抛异常，终止遍历

fail-safe：安全失败机制，对于集合的遍历操作不是在原集合上操作的，在进行遍历的时候，会Copy一这个集合，然后再新的集合对象下进行遍历操作，这样就避免了fail-fast机制下修改集合抛异常的场景，缺点就是对原来集合内的写操作一无所知。

原理分析（主要部分）：

Hashtable底层是通过数组加链表来实现的，大部分的实现逻辑和HashMap的很相似，可以参考HahsMap.

Hashtabale中有两个影响其性能的参数，分别是初始容量和加载因子

所以Hashtable 提供了不同参数类型的构造方法，如下为默认构造：

```java
public Hashtable() {  
    this(11, 0.75f);//默认容量11，加载因子0.75  
}
```
put方法：

```java
// hashtable 中所有涉及到线程安全的方法都有synchronized修饰
// 所以在效率上可能会相对低一点
public synchronized V put(K key, V value) {
    if (value == null) {
        throw new NullPointerException();   // hashtable 在运行时检测到为null的时候会报空指针异常
    }
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;    // hashtable中计算index的算法和hashMap中也不一致，保证是一个正数之后再取余操作
    @SuppressWarnings("unchecked")
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    for(; entry != null ; entry = entry.next) {     // 确保键不在哈希表中。
        if ((entry.hash == hash) && entry.key.equals(key)) {
            V old = entry.value;
            entry.value = value;
            return old;
        }
    }
    addEntry(hash, key, value, index);  // 将Entity 添加到hashtable
    return null;
}
// 添加节点
private void addEntry(int hash, K key, V value, int index) {
    modCount++;  // 修改次数++
    Entry<?,?> tab[] = table;
    if (count >= threshold) {   // 容量大于阈值，进行扩容操作
        rehash();  // 重新根据容量计算hash（如果事先知道容量大小，尽量按照实际需求初始化，rehash操作非常损耗性能），后面会介绍
        tab = table;
        hash = key.hashCode();
        index = (hash & 0x7FFFFFFF) % tab.length;
    }
    // 添加节点到链表首部
    Entry<K,V> e = (Entry<K,V>) tab[index];  // 获取原来链表节点的头节点
    tab[index] = new Entry<>(hash, key, value, e);  // 设置新节点为头节点，原来的头节点e设置为新节点的next 节点
    count++;
}
```

rehash方法

```java
protected void rehash() {
    int oldCapacity = table.length;
    Entry<?,?>[] oldMap = table;
    int newCapacity = (oldCapacity << 1) + 1;  // 扩容为原来的2倍+1
    if (newCapacity - MAX_ARRAY_SIZE > 0) {   // 扩容后数量最大值校验
        if (oldCapacity == MAX_ARRAY_SIZE)
            return;
        newCapacity = MAX_ARRAY_SIZE;
    }
    Entry<?,?>[] newMap = new Entry<?,?>[newCapacity];  // 创建新容量的数组
    modCount++; // 操作次数
    threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);  // 重新计算阈值
    table = newMap; // 引用复制，通过如下代码将原来数据添加到新的数组中
    for (int i = oldCapacity ; i-- > 0 ;) {
        for (Entry<K,V> old = (Entry<K,V>)oldMap[i] ; old != null ; ) {
            Entry<K,V> e = old;
            old = old.next;
            int index = (e.hash & 0x7FFFFFFF) % newCapacity;  // 计算index
            e.next = (Entry<K,V>)newMap[index];
            newMap[index] = e;
        }
    }
}
```
其它方法比较简单，可以参考HashMap的实现方式

由于Hashtable在内部是通过synchronized方法的方式实现线程安全，所以在效率上不是很高，一般的开发中对于hashtable的使用已经被弃用了，可以通过Collections下的synchronized方法（本质还是synchronized，只是锁粒度变小），或者比较高性能的ConcurrentHashMap（推荐使用，后面有介绍）。
### LinkedHashMap
基于jdk1.8分析

从源码中可以看到，LinkedHashMap是继承自HashMap的，LinkedhashMap是基于HashMap在访问遍历顺序上面做了一些增强。理解LinkedHashMap的前置条件是熟悉HashMap的数据结构极原理信息（可以参考前面部分的HashMap）。

从LinkedHashMap的put方法可以得到该put方法实际是复用的父类HashMap方法，所以很容易得到，LinkedHashMap的底层数据结构就是HashMap存储数据的结构（数组+链表/红黑书）

待完善… …

### ConcurrentHashMap
前面介绍了HashMap和Hashtable，为什么要有ConcurrentHashMap呢？

因为HashMap线程不安全但高效、Hashtable线程安全但效率低，响应的特性之前也有介绍

而ConcurrentHashMap结合了这两个Map的特点，它在保证线程安全的前提下尽可能保证高效。

在JDK1.7之前(含)和1.8之后(含)的实现原理是不一样的

JDK1.7 前底层的是分段锁的机制

分段锁机制：即容器中会存在很多把锁，每一段数据分配一把锁，这样多线程在访问不同段的数据时就不存在数据安全问题了，因为每一段锁锁的数据没有冲突。

ConcurrentHashMap在1.7之前是通过Segment数组和HashEntry数组结构组成，Segment是一种可重入锁ReentrantLock，HashEntry则用于存储键值对数据。一个ConcurrentHashMap里包含一个Segment数组，Segment的结构和HashMap类似，是一种数组和链表结构， 一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素， 每个Segment守护者一个HashEntry数组里的元素,当对HashEntry数组的数据进行修改时，必须首先获得它对应的Segment锁。

```java
public class ConcurrentHashMap<K, V> extends AbstractMap<K, V> implements ConcurrentMap<K, V>, Serializable {
    // 每个segment都是一个锁；与hashtable相比，这么设计的目的是对于put, remove等操作，可以减少并发冲突
    // 对不属于同一个片段的节点可以并发操作，大大提高了性能
    final Segment<K,V>[] segments;
    // 本质上Segment类就是一个小的hashmap，里面table数组存储了各个节点的数据，继承了ReentrantLock, 可以作为互拆锁使用
    static final class Segment<K,V> extends ReentrantLock implements Serializable {
        transient volatile HashEntry<K,V>[] table;
        transient int count;
    }
    // 节点，存储Key、value
    static final class HashEntry<K,V> {
        final int hash;
        final K key;
        volatile V value;
        volatile HashEntry<K,V> next;
    }
}
```
JDK1.8 后底层是通过CAS+Synchronized的方式实现的，抛弃了Segment分段锁的锁机制。

优于1.7的地方时，1.8的实现降低了锁的粒度，1.7的锁粒度是基于Segment的，包含多个HashEntry，而JDK1.8锁的粒度是首节点HashEntry

1.8使用红黑树来优化链表，提升遍历效率

ConcurrentHashMap的数据存储结构和基本属性和HashMap中的基本一致，可以参考HashMap的介绍，这里主要介绍put和get操作。

ConcurrentHashMap的put操作：

ConcurrentHashMap在put的过程中如果没有发生冲突，则采用CAS进行无锁更新，只有在put的过程中出现hash碰撞的情况下，会锁住链表的header或数的根节点并添加或者更新结点到链表或者树节点上。

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
	  // 获取hash
		int hash = spread(key.hashCode());
    int binCount = 0; // 记录几点个数，方便扩容等操作
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
					 // 首次put的时候进行初始化操作 
           tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {  // 找到位置
						// 如果这个位置没有元素的情况下，通过cas的方式直接尝试添加元素，这种情况下是没有锁的
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        // hash值为MOVED的时候，说明当前正在进行扩容操作，先协助扩容再进行更新操作
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
						// 如果该位置有元素，则采用synchronized进行加锁，并遍历，如果存在则更新，否则添加
						// 剩下的操作就和HashMap的操作很类似
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key, value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) { // 红黑树
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key, value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);  // 达到阈值，扩张数组或者转化结构
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
		// 计数操作
    addCount(1L, binCount);
    return null;
}
```
ConcurrentHashMap的get操作：

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    int h = spread(key.hashCode()); // 计算hash
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
				// 确定元素位置，然后遍历结点，返回
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
				// 遇到扩容的情况，会调用标志正在扩容节点ForwardingNode的find方法，查找该节点，匹配就返回
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        // 以上都不符合的话，就往下遍历节点，匹配就返回，否则最后就返回null
				while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```
### ConcurrentSkipListMap
ConcurrentSkipListMap是线程安全的有序的哈希表

如下是一个贴子上摘抄的和ConcurrentHashMap的一个测试结果：
```text
在4线程1.6万数据的条件下，ConcurrentHashMap 存取速度是ConcurrentSkipListMap 的4倍左右。
但ConcurrentSkipListMap有几个ConcurrentHashMap 不能比拟的优点：
1、ConcurrentSkipListMap 的key是有序的。
2、ConcurrentSkipListMap 支持更高的并发。ConcurrentSkipListMap 的存取时间是log（N），和线程数几乎无关。也就是说在数据量一定的情况下，并发的线程越多，ConcurrentSkipListMap越能体现出他的优势。
在非多线程的情况下，应当尽量使用TreeMap。此外对于并发性相对较低的并行程序可以使用Collections.synchronizedSortedMap将TreeMap进行包装，也可以提供较好的效率。对于高并发程序，应当使用ConcurrentSkipListMap，能够提供更高的并发度。
所以在多线程程序中，如果需要对Map的键值进行排序时，请尽量使用ConcurrentSkipListMap，可能得到更好的并发度。
注意，调用ConcurrentSkipListMap的size时，由于多个线程可以同时对映射表进行操作，所以映射表需要遍历整个链表才能返回元素个数，这个操作是个O(log(n))的操作
```
ConcurrentSkipListMap的底层是通过跳表结构实现的（跳表是一个通过多层的链表结构，牺牲空间换效率的一种数据结构）

如下是跳表数据结构的一个简单示例和查找结点的路径示例图
![](1.png)

不难看出，红色路径找到红色元素的效率明显优于最底层通过链表的方式找到红色元素的路径。

在跳表结构中，每一层通过维护一个Index来记录对应关系
```java
static class Index<K,V> {
	final Node<K,V>node;
    final Index<K,V>down;
    volatile Index<K,V>right;
		//...
}
```
当实例化ConcurrentSkipListMap的时候，会调用内部的 initialize 方法，如下。
```java
private void initialize() {
    keySet = null;
    entrySet = null;
    values = null;
    descendingMap = null;
		// 初始化元素，包括head ，以及每一层的分层索引down（为null），默认层级level 为1
    head = new HeadIndex<K,V>(new Node<K,V>(null, BASE_HEADER, null), null, null, 1);
}
```

在我们进行put的操作的时候，内部调用了doPut 方法，

进行get操作的时候内部调用了doGet方法，

put和get的具体信息可以参考这篇博客，写的很详细 https://my.oschina.net/u/3768341/blog/3135659
### TreeMap
**更新中.......**
### EnumMap
### WeakHashMap
### IdentifyHahsMap
### IdentityHashtable


## Queue

### Deque
#### ArrayDeque
#### BlockingDeque
#### LinkedBlockingDeque

### BlockingQueue
#### ArrayBlockingQueue
#### PriorityBlockingQueue
#### LinkedBlockingQueue
#### TransferQueue
#### LinkedTransferQueue
#### SynchronousQueue

### PriorityQueue
### ConcurrentLinkedQueue
### DelayQueue

