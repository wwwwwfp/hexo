---
title: Java常用JUC并发包的简单使用和AQS原理分析
index_img: /img/cover/26.jpg
categories:
  - Java基础
tags:
  - JUC
  - AQS
abbrlink: 8d24c38b
date: 2020-09-20 19:24:15
---

## 常见并发包

### Reentrant Lock
实现锁的功能，类似Synchronized，相较于Synchronized，Lock提供了更灵活的API，更容易扩展，以及更灵活的使用场景。
```java
/** 
 * 基本使用
 */
Lock lock  = new ReentrantLock();
// ReentrantLock lock = new ReentrantLock(true); // 公平锁
lock.lock();
try{
	//可能会出现线程安全的操作
	// lock.tryLock(2, TimeUnit.SECONDS);
}catch(Exception e){
	// catch sth
}finally{
	//一定在finally中释放锁
	//也不能把获取锁在try中进行，因为有可能在获取锁的时候抛出异常
	lock.ublock();
}

/**
 * 模拟消费者生产者
 */
public class Produce_Customer_Container2<T> {
    final private LinkedList<T> lists = new LinkedList<T>();
    final private int MAX = 10; // 最多10个元素
    private int count = 0;  // 记录容量
    private Lock lock = new ReentrantLock();
    private Condition producer = lock.newCondition();
    private Condition consumer = lock.newCondition();
    public int getCount(){
        return this.count;
    }
    // 生产
    public  void put(T t){
        try {
            lock.lock();
            while (lists.size() == MAX) {
                producer.await();
            }
            System.out.println("生产==="+t);
            lists.add(t);
            count++;
            consumer.signalAll(); // 通知消费者 消费
        } catch (Exception e) {
        }finally {
            lock.unlock();
        }
    }
    // 消费
    public  T get(){
        T result = null;
        try {
            lock.lock();
            while (lists.size() == 0) {
                consumer.await();
            }
            result = lists.removeFirst();
            count --;
            producer.signalAll(); // 通知消费者生产
        } catch (Exception e) {
        }finally {
            lock.unlock();
        }
        return result;
    }
    public static void main(String[] args) {
        Produce_Customer_Container2 container = new Produce_Customer_Container2();
        for (int i = 1; i <=100; i++) {
            new Thread(()->{
                System.out.println("消费---"+container.get());
            }).start();
        }
        for (int i = 1; i <=2; i++) {
            int temp = i;
            new Thread(()->{
                for (int j=0;j<50;j++){
                    container.put(temp);
                }
            }).start();
        }
    }
}
```
### ReadWriteLock
ReadWriteLock管理了一组锁，读锁 readLock 和 写锁 writeLock，

读锁是共享锁，写锁为排他锁（互斥锁），当一些逻辑上了读锁的时候，其它的读锁也是可以使用这个读锁的。读之间共享，读写、系写 互斥。

使用场景：解决脏读数据的一类问题。

示例：
```java
public class Test_ReadWriteLock {
    // 写操作
    public static void read(Lock lock){
        try {
            lock.lock();
            Thread.sleep(5000);
            System.out.println("read........."+System.currentTimeMillis()/1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
    // 读操作
    public static void write(Lock lock){
        try {
            lock.lock();
            Thread.sleep(5000);
            System.out.println("write........."+System.currentTimeMillis()/1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
        // 读锁
        Lock readLock = readWriteLock.readLock();
        // 写锁
        Lock writeLock = readWriteLock.writeLock();
        Runnable read = ()->read(readLock);
        Runnable write = ()->write(writeLock);
        for (int i=1;i<=8;i++) {
            new Thread(read,"读："+i).start();
            if (i % 4 ==0) {
                new Thread(write,"写："+i).start();
            }
        }
    }
}
// 运行结果 ——  读之间共享，读写、系写  互斥
// read.........1597231126
// read.........1597231126
// read.........1597231126
// read.........1597231126
// read.........1597231126
// write.........1597231131
// read.........1597231136
// read.........1597231136
// read.........1597231136
// write.........1597231141
```
### CountDownLatch
CountDownLatch其实可以把它看作一个计数器,非常典型的应用场景是：有一个任务想要往下执行，但必须要等到其他的任务执行完毕后才可以继续往下执行。当 countDown 到 0 的时候，才会执行latch.await() 后面的逻辑。

示例：t1中当list 的大小到5 的时候，执行t2 await 后面的逻辑
```java
public class Test_CountDownLatch {
	public static void main(String[] args) {
		List<Integer> list = new ArrayList<>();
		CountDownLatch latch = new CountDownLatch(5);
		new Thread(() -> {
			System.out.println("t2 —— start");
			try {
				latch.await();
				System.out.println("list 大小："+list.size());
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println("t2 —— end");
		}, "t2").start();
		new Thread(() -> {
			System.out.println("t1 —— start");
			for (int i = 0; i < 10; i++) {
				list.add(i);
				latch.countDown();
				try {
					TimeUnit.SECONDS.sleep(1);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			System.out.println("t1 —— end");
		}, "t1").start();
	}
}
// 执行结果
// t2 —— start
// t1 —— start
// list 大小：5
// t2 —— end
// t1 —— end
```
### Semaphore
Semaphore用于限制可以访问某些资源（物理或逻辑的）的线程数目，他维护了一个许可证集合，有多少资源需要限制就维护多少许可证集合，假如这里有N个资源，那就对应于N个许可证，同一时刻也只能有N个线程访问。一个线程获取许可证就调用acquire方法，用完了释放资源就调用release方法。
```java
public class Test_Semaphore {
    public static void main(String[] args) {
        // 许可   数量
        Integer permits = 1;
        // true 公平，线程先排队到队列 ，再排队拿许可
        Semaphore semaphore = new Semaphore(permits,true);
        //Semaphore semaphore = new Semaphore(permits);  //默认非公平
        for (int i = 1;i<=3;i++) {
            final int temp = i;
            new Thread(()->{
                try {
                    // 阻塞方法 acquire 每获 取一次  许可permits 会-1
                    // 只有拿到许可后才会执行 acquire 后面的逻辑
                    // 许可数量为0 时，阻塞
                    System.out.println("--------"+temp);
                    semaphore.acquire();
                    TimeUnit.SECONDS.sleep(1);
                    System.out.println("========"+temp);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    //释放资源，将当前许可数量恢复   permits + 1
                    semaphore.release();
                }
            }).start();
        }
    }
}
//  执行结果
// --------1
// --------2
// --------3
// ========2
// ========3
// ========1
```
### CyclicBarrier
从字面上的意思可以知道，这个类的中文意思是“循环栅栏”。大概的意思就是一个可循环利用的屏障。它的作用就是会让所有线程都等待完成后才会继续下一步行动。

简单示例
```java
public class Test_CyclicBarrier {
    public static void main(String[] args) {
        /**
         *  让指定数量的线程都等待完成后才会进入下一步行动
         *  parties  参与线程的限制个数
         *  barrierMethod ：当线程数达到parties 数量后 await 放行，并执行barrierMethod方法
         */
        Integer parties = 2;
        CyclicBarrier barrier = new CyclicBarrier(parties,()-> barrierMethod());
        for (int i=1;i<=6;i++) {
            final int a = i;
            new Thread(()->{
                try {
                    System.out.println("wait  前："+a);
                    barrier.await();   // 阻塞线程，直到线程数量堆满barrier
                    System.out.println("wait  后："+a);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
    public static void barrierMethod(){
        System.out.println("放行=======");
    }
}
// 运行结果
// wait  前：1
// wait  前：2
// 放行=======
// wait  后：1
// wait  后：2
// wait  前：3
// wait  前：4
// 放行=======
// wait  后：4
// wait  后：3
// wait  前：5
// wait  前：6
// 放行=======
// wait  后：6
// wait  后：5
```
### Phaser
Phaser是JDK 7新增的一个同步辅助类，在功能上跟CyclicBarrier和CountDownLatch差不多，但支持更丰富的用法。

使用场景是按照不同的阶段来对线程进行执行的场景，如一批学员要想拿到证书必须要通过如下步骤，全部报名完成后，一起去考试，考试通过后，部分学员会拿到证书这个场景。
```java
public class Test_Phaser {
    // 注册线程数
    static Integer parties = 5;
    static Random random = new Random();
    static ExamPhaser  phaser = new ExamPhaser();

    static class Student implements Runnable{
        String name;  // 姓名
        Integer score;  // 分数
        public Student(String name) {
            this.name = name;
        }
        // 报名注册
        public void register(){
            System.out.println(this.name+" 已报名");
            phaser.arriveAndAwaitAdvance();
        }
        // 去考试
        public void goExam(){
            System.out.println(this.name+" 去考试");
            phaser.arriveAndAwaitAdvance();
        }
        // 考试
        public void exam(){
            this.score = random.nextInt(10);
            System.out.println(this.name+" 已参加考试，成绩为："+this.score);
            phaser.arriveAndAwaitAdvance();

        }
        // 拿证书
        public void certificate(){
            if (this.score >= 6) {
                System.out.println(this.name+" 成绩为:"+this.score+",通过考试，可以拿证");
                phaser.arriveAndAwaitAdvance();
            }else {
                System.out.println(this.name+" 成绩为:"+this.score+",垃圾，滚回去继续深造");
                // 将当前线程从当前阶段剔除，只有考试通过的学院才可以执行
                // onAdvance 中证书这个阶段的逻辑
                phaser.arriveAndDeregister();
                //phaser.register();  注册
            }
        }
        @Override
        public void run() {
            register(); // 报名
            goExam(); //去考试
            exam();  //考试
            certificate();// 获取证书
        }
    }
    // 自定义 Phaser   屏障事件
    static class ExamPhaser extends Phaser{
        @Override
        protected boolean onAdvance(int phase, int registeredParties) {
            // return ture  阶段 phase后面的阶段将不执行
            switch (phase) {
                case 0:
                    System.out.println("======"+registeredParties+"(人)报名完成======");
                    break;
                case 1:
                    System.out.println("======"+registeredParties+"(人)去考试======");
                    break;
                case 2:
                    System.out.println("======"+registeredParties+"(人)考试完成======");
                    break;
                case 3:
                    System.out.println("======"+registeredParties+"(人)获取证书======");
                    break;
                default:
                    break;
            }
            return super.onAdvance(phase,registeredParties);
        }
    }
    public static void main(String[] args) {
        phaser.bulkRegister(parties);
        for (int i=1;i<=5;i++) {
            new Thread(new Student("学员 "+i )).start();
        }
    }
}
// 运行结果
学员 1 已报名
学员 2 已报名
学员 3 已报名
学员 4 已报名
学员 5 已报名
======5(人)报名完成======
学员 5 去考试
学员 4 去考试
学员 3 去考试
学员 2 去考试
学员 1 去考试
======5(人)去考试======
学员 1 已参加考试，成绩为：9
学员 2 已参加考试，成绩为：6
学员 4 已参加考试，成绩为：4
学员 3 已参加考试，成绩为：5
学员 5 已参加考试，成绩为：2
======5(人)考试完成======
学员 5 成绩为:2,垃圾，滚回去继续深造
学员 3 成绩为:5,垃圾，滚回去继续深造
学员 2 成绩为:6,通过考试，可以拿证
学员 4 成绩为:4,垃圾，滚回去继续深造
学员 1 成绩为:9,通过考试，可以拿证
======2(人)获取证书======
```
### Exchanger
交换器，两个线程之间的事情，a 线程 exchange 的时候 会阻塞 ，b 线程 exchange 的时候 也会阻塞，等两个线程交换完值以后再继续执行 exchange 后面的逻辑。

使用场景为双发进行等价交易等情况、或交换数据的情况，简单示例如下：
```java
public class Test_Exchanger {
    public static void main(String[] args) {
        Exchanger<Integer> exchanger = new Exchanger<>();
        for (int i = 1;i<=2;i++) {
            int temp = i;
            new Thread(()->{
                Integer exchange = null;
                try {
										// 扔进 Exchanger 容器中 等待和 容器中第二交换
                    exchange = exchanger.exchange(temp);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(temp+"     "+exchange);
            }).start();
        }
    }
}
// 执行结果
2     1
1     2
```

### LockSupport
作用：让当前线程阻塞，不是锁

原来阻塞线程和唤醒线程是通过wait 方法 和notify 方法，而wait 方法和notify方法是Object中的方法，所以必须要加一把锁才可以，而park 和 unpark 是 LockSupport 提供的静态方法，可以直接调用,notify 不可以唤醒指定的线程、而 unpark可以唤醒指定的线程。
```java
public class Test_LockSupport {
    public static void main(String[] args) {
        Thread thread = new Thread(()->{
            for (int i=1;i<=10;i++) {
                System.out.println(i+"    当前时间戳："+System.currentTimeMillis()/1000);
                if (i == 5) {
                    LockSupport.park();
                }
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        thread.start();
        // LockSupport.unpark(thread);   // unpark 位置1
        try {
            Thread.sleep(8000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        LockSupport.unpark(thread);   // unpark 位置2

    }
}
```
验证：unpark方法先于park方法调用，会使park方法失效，线程不会阻塞，结果如下：
![](1.png)

## AQS 原理分析
AQS 全称AbstractQueuedSynchronizer，是上面很多JUC并发包下的一个基类。包括上面介绍的ReentrantReadWriteLock，ReentrantLock、Semaphore、CountDownlatch等底层都有用到AbstractQueuedSynchronizer。

AQS底层实际是一个volatile修饰的int类型的state维护了一个FIFO的双向链表，用CAS操作head节点和tail节点的方式替代了锁整条node链表的核心。
![](2.png)

### 以ReentrantLock为例
ReentrantLock的lock方法有两个实现（公平锁和非公平锁），公平锁和非公平的意思是有新线程抢占CPU资源的时候非公平锁的情况是不去检查队列里面是否有线程，直接和队列里面的争抢CPU时间片，公平锁的情况是如果队列中有没有完成的线程，则插入到队列中等待执行
![](3.png)
#### 如下是FairSync 和 NonfairSync 下对lock方法的不同实现
```java
static final class FairSync extends Sync {
    final void lock() {
        acquire(1);
    }
}
// nonfair 的情况比fair的情况下多了异步判断操作，
// 直接CAS 去修改（抢占资源），如果修改成功 将当前线程设置成独占线程
// 如果没有修改成功，说明cpu时间片被别的线程占用，需要入队
// 注：入队线程获取CPU时间片是公平的，先到的先得
static final class NonfairSync extends Sync {
    final void lock() {
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }
}
```
##### acquire 方法的实现
```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
    	// 重点（入队），后面介绍
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
// 下面介绍具体实现
```
##### tryAcquire 的具体实现

调用acquire 的时候内部有调用了 tryAcquire

尝试再一次获得，首先判断state 是否为0（无锁的情况）

其次判断当前线程是否为exclusiveOwnerThread，然后把state++
```java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    // 判断没锁的情况  
    if (c == 0) {  
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
	// 重入再次获得锁的情况
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}

```

##### 重点（入队），创建节点 addWaiter

```java
/**
 * 创建一个Node(Thread) 节点，添加到队列的尾部 
 * 添加队列的时候要加锁，用CAS操作，不需要将原来的整个链表上锁，
 * 只需要关注tail即可，通过观察tail节点是否满足set时候的预期来控制（compareAndSetTail 自旋）
 */
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    Node pred = tail;  // 获取链表 尾节点
    if (pred != null) {
        node.prev = pred;   // 当前node pre节点设置为 原来的尾节点 pred
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);  // 自旋操作
    return node;   // 返回一个node
}
```

##### 在队列里排队获得锁（尝试获得）

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
        	// 获取当前添加节点的前置节点
            final Node p = node.predecessor();   
            // 如果前置节点是头节点，说明快轮到当前线程，就尝试获取锁，如果拿到，则设置当前结点为头节点
            if (p == head && tryAcquire(arg)) { 
                setHead(node);     // 设置为头节点
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

### 总结
所以AQS的核心是通过一个volatile修饰的int类型的state（保证其它线程可见）维护了一个FIFO的双向链表通过操作head节点和tail节点来完成资源的调度

