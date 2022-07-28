---
title: CAS (CompareAndSwap) 底层基本原理分析和 ABA问题
index_img: /img/cover/25.jpg
categories:
  - Java基础
tags:
  - CAS
  - ABA
abbrlink: 44916e5f
date: 2020-09-07 16:07:43
---

### CAS机制
CAS——CompareAndSwap：比较并替换

作用是进行计算的时候判断当前值是否满足预期，如果满足则更新为新值，保证整个过程具备原子性。通过内存中的值，逻辑值和要更改的值进行比较替换，通过自旋的方式在操作内存的值的时候通过内存的值和逻辑值进行比较，如果一致，则替换更改（这一步原子操作）。

### 代码分析
JDK中为了方便开发正操作，已经实现了很多原子性操作的类，这些类底层就是通过CAS控制原子操作的，比如AtomicInteger，通过AtomicInteger 提供的API就可以验证CAS的简单执行原理。在AtomicInteger API中可以看到具体实现是通过sun.misc包中的Unsafe类实现的。后面会简单的说明，如下代用来验证CAS的简单执行过程和思想。
```java
public class Test {
    public static void main(String[] args) {
        AtomicInteger test = new AtomicInteger();
        System.out.println(test.get());   // 初始化的 值为 0 
        /*
         //该方法就是根据预期来更新值得，底层是调用了Unsafe类的 compareAndSwapInt
         //expect 预期值， update 更新的值
         public final boolean compareAndSet(int expect, int update) {
         	 // 最后调用的是Unsafe类中的compareAndSwap 方法
		     return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
		 }
         */
        boolean b = test.compareAndSet(2, 1);  // 传入一个预期值2  但实际值是0
        System.out.println(b);   // false   实际值0 不满足预期值2 即设置不成功
        System.out.println(test.get());  // 0  
        b = test.compareAndSet(0, 10);   // 预期值0   实际值0 
        System.out.println(b);        // true
        System.out.println(test.get());  // 10  满足预期
    }
}
```
### Unsafe 类
Unsafe是存在于sun.misc包中的一个类，该类的作用是使Java拥有想C语言一样直接操作内存空间的能力，该类使用了单利模式，需要通过一个静态方法来获取，但是又做了限制，如果是普通调用，会抛出一个SecurityException的异常，只允许jdk 加载核心类的类加载器(Bootstrap)加载的类才可以调用。该类提供了很多直接操作内存的方法。

```java
// Unsafe提供了许多可以直接操作内存的一些API
// 如分配指定内存大小，释放内存，给指定内存设值等。
public native long allocateMemory(long bytes);
public native long reallocateMemory(long address, long bytes);
public native void freeMemory(long address);
public native void setMemory(Object o, long offset, long bytes, byte value);
public native void putAddress(long address, long x);
public native long getAddress(long address);
public native void putLong(long address, long x);
public native long getLong(long address);
public native byte getByte(long address);
public native void putByte(long address, byte x);
public native int pageSize();
```
Unsafe从名称上就可以看到该类是非安全的，JAVA官方也不建议直接只用该类，所以对该类做了很多限制，禁止开发者直接在Java中直接调用Unsafe 相关的代码，包括方法修饰符都是native，以及只允许JDK加载核心类加载器加载的调用。
```java
// Unsafe.class
@CallerSensitive
public static Unsafe getUnsafe() {
    Class var0 = Reflection.getCallerClass();
    if (!VM.isSystemDomainLoader(var0.getClassLoader())) {
        throw new SecurityException("Unsafe");
    } else {
        return theUnsafe;
    }
}
// VM.class
public static boolean isSystemDomainLoader(ClassLoader var0) {
    return var0 == null;
}
```
在自己代码中执行 “Unsafe unsafe = Unsafe.getUnsafe();” 语句会抛异常，这也印证了上面说的。debug如下
![](1.png)
### 简单了解Unsafe底层实现
Unsafe.java 的底层具体实现是通过C++ 实现的，unsafe.cpp 的方法如下：
```text
UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapInt(
  JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x))
  UnsafeWrapper("Unsafe_CompareAndSwapInt");
  oop p = JNIHandles::resolve(obj);
  jint* addr = (jint *) index_oop_from_field_offset_long(p, offset);
  return (jint)(Atomic::cmpxchg(x, addr, e)) == e;
UNSAFE_END
```
不难看出，在这些代码中最有价值的一行代码就是 return (jint)(Atomic::cmpxchg(x, addr, e)) == e;关于该方法具体细节没有细究，这行代码的大概意思就是 (Atomic::cmpxchg(x, addr, e)) 的返回值和 e作比较，相等就返回true。
如下是 底层cmpxchg 的具体方法
```text
inline jint     Atomic::cmpxchg(
    jint     exchange_value, 
    volatile jint*     dest, 
    jint     compare_value) {
  int mp = os::is_MP();
  __asm__ volatile ("cmp $0, " "%4" "; je 1f; lock; 1: " "cmpxchgl %1,(%3)"
                    : "=a" (exchange_value)
                    : "r" (exchange_value), "a" (compare_value), "r" (dest), "r" (mp)
                    : "cc", "memory");
  return exchange_value;
}
```
通过参考别的文章和资料，大概了解到 cmpxchg 方法到CPU层面的具体实现是通过一条 cmpxchg 指令，关于Linux内核的具体实现可以参考https://blog.csdn.net/zdy0_2004/article/details/48013829

https://blog.csdn.net/isea533/article/details/80301535
### CAS ABA问题
#### 什么是ABA？
由于CAS的整个过程是在操作值得时候检查时不时和原来的有出入，但是这样很难避免一个问题，就是调用开始到具体操作这个过程中有没有变化过，就比如如果一个值原来是A，在开始操作之前由原来的A变成B，再变回A的过程。就是说在具体操作的时候，这个A已经不是原来的A了，这样就不具备原则性。

网上看到过这样一个例子，虽然很俗，但是很形象，说你和你的女朋友分手了，分手后和别的男人好了，然后和那个男人分手后又来找你，这个时候你女朋友还是原来的女朋友么/呲牙=，=。
#### 如何解决这个问题？
解决思路：操作的时候通过加版本号来避免ABA问题。

从JDK5开始，提供了一个AtomicStampedReference 来解决这个问题，通过加一个版本号来解决这个问题，操作一次，版本号更新一次。

示例代码：
```java
public class ABADemo {
    private static AtomicInteger atomicInteger = new AtomicInteger(1);
    private static AtomicStampedReference atomicStampedReference = new AtomicStampedReference(1, 0);

    // 常规原子操作-----不考虑ABA问题
    public static void testAtomicInteger(){

        Thread t1 = new Thread(()->{
            atomicInteger.compareAndSet(1, 2);
            atomicInteger.compareAndSet(2, 1);
        });
        Thread t2 = new Thread(()->{
            try {
                t1.join();
            } catch (Exception e) {
            }
            boolean success = atomicInteger.compareAndSet(1, 2);
            System.out.println(success);
        });
        t1.start();
        t2.start();
    }

    // 考虑ABA问题----通过Reference 解决
    public static void testAtomicStampedReference(){
        Thread t1 = new Thread(()->{
            atomicStampedReference.compareAndSet(1, 2, atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
            atomicStampedReference.compareAndSet(2, 1, atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
        });
        Thread t2 = new Thread(()->{
            try {
                t1.join();
            } catch (InterruptedException e) {
            }
            int stamp = atomicStampedReference.getStamp();
            System.out.println(stamp);    // 2
            System.out.println(atomicStampedReference.getReference());  //1
            Integer expectStemp = stamp;   // 2
            // expectStemp = 1;
            boolean success = atomicStampedReference.compareAndSet(1, 2, expectStemp, stamp+1 );
            System.out.println(success);
        });
        t1.start();
        t2.start();

    }

    public static void main(String[] args) throws Exception {
        testAtomicInteger();    // true
        // 如果将上面expectStemp 的值设置成2的，则输出true，设置成1 则输出false
        // 这说明通过版本号可以解决ABA问题
        testAtomicStampedReference();
    }
}
```