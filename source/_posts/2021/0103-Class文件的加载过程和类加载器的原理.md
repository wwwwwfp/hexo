---
title: Class文件的加载过程和类加载器的原理
index_img: /img/cover/13.jpg
categories:
  - Java基础
tags:
  - class
  - 类加载器
  - 双亲委派
abbrlink: d16df73a
date: 2021-07-02 11:04:52
---

### Class 文件是怎样被放在内存的
![](1.png)

#### 硬盘中的Class被加载的过程
1. loading

    把一个class 加载到内存,懒加载，需要的时候再加载
2. linking
   1. verification
   
       校验class，是否满足class的格式
   
   2. preparation
   
        把class中静态变量设置成默认值 int类型 0
   3. resolution
   
       解析 loadeClass方法中的第二个参数 true 为解析 false 不解析
   class中常量池用到的符号引用转换成可以直接访问内存的值（直接能访问到的内容）
   即 将类、方法、属性等符号引用解析为直接引用
   常量池中的各种符号引用解析为指针、偏移量等内存地址的直接引用
3. initializing

   静态变量的赋值，赋值为初始值，调用静态变量块

   在类初始化代码,给静态成员变量赋初始值

    Class：load 默认值 初始化
   
    new：申请内存 默认值 初始值

### 类加载器（ClassLoader）

jvm有一个类加载器的层次，分别加载不同的class

jvm中所有的class 都是被类加载器加载到内存的

一个class 被 load 到内存后， 内存中创建了两块内容

一块是将class对应的二进制内容扔进 内存中 ，同时也生成了一个class类的对象并指向二进制文件，这个对象保存在mataspace中

别的对象访问这个类的时候就是通过访问生成的这个Class类对象，然后这个class类对象再去访问二进制文件

即访问过程是 通过访问class文件对应创建的class类，然后class类从指向的二进制中获取对应二进制并解析成汇编指令

如下是获取类对应加载器的部分示例

```java
public class T002_ClassLoaderLevel {
    public static void main(String[] args) {
		// null---Bootstrap ClassLoader  
		// 返回null的原因：Bootstrap ClassLoader 是由C++实现的，java中没有对应的class 所以返回空
		// 可以理解成c++实现的一个模块
        System.out.println(String.class.getClassLoader());  
		//null---Bootstrap ClassLoader
        System.out.println(sun.awt.HKSCS.class.getClassLoader());  
		//sun.misc.Launcher$ExtClassLoader@6d6f6e28 --- Extension ClassLoader
        System.out.println(sun.net.spi.nameservice.dns.DNSNameService.class.getClassLoader());  
		//sun.misc.Launcher$AppClassLoader@18b4aac2 --- App ClassLoader
        System.out.println(T002_ClassLoaderLevel.class.getClassLoader());
		// null   父加载器不是 加载器的加载器     所以   classLoader 的 classloader 都是null
        System.out.println(sun.net.spi.nameservice.dns.DNSNameService.class.getClassLoader().getClass().getClassLoader());
		// null  同上   ClassLoader 本身就是c++ 写的
        System.out.println(T002_ClassLoaderLevel.class.getClassLoader().getClass().getClassLoader());
		// sun.misc.Launcher$ExtClassLoader@6d6f6e28 --- T002_ClassLoaderLevel 的类加载器是APP APP的父加载器是 Ext加载器
        System.out.println(new T002_ClassLoaderLevel().getParent());
        System.out.println(ClassLoader.getSystemClassLoader());
    }
}
```

#### 加载器分类
![](2.png)

Bootstrap： jdk 加载核心类的类加载器

Extension： 是加载扩展包中的的

App： 加载classpath 指定的类的，即加载我们自己写的类

Custom： 自定义的类加载器

#### Class 类加载器加载过程
**双亲委派：** 从子到父的过程和从父到子的过程

**过程：** class首先会从 Custom加载器（缓存 ——每个类加载器 维护一个list 用来管理已经加载的对象引用）中找，找到返回结果，找不到会从上层加载器APP中找，如果还是找不到再到Ext加载器中招，一直到Bootstrap加载器，(前面的find 都是从缓存中找)，如果缓存中没有，则find class 并加载，如果 bootstrap没有加载到，则交给下一层加载器 ext find class 并加载 ，直到custom 。如果custom 没有找到class,则报错 ClassNotFoundException异常

![](3.png)
源码中 每一个加载器维护了一个 属性名师parent的ClassLoader，通过这个属性找对应关系

#### 双亲委派注意的问题：
+ 父加载器

   父加载器不是“类加载器的加载器”，也不是“类加载器的父类加载器”
+ 双亲委派是一个孩子向父亲方向，然后父亲向孩子方向的双亲委派过程

#### 为啥要搞双亲委派

+ 主要是为了安全

   假如没有双亲委派，我自己定义一个 java.lang.String ，这样自定classloader 则会加载到的是自己写的，不是oracle的，如果是双亲委派，则find cache 的时候 找到的jdk的String

+ 其次是效率问题

#### 如何打破双亲委派机？ 即不想用双亲委派机制
由于委培机制是在 ClassLoader 类中的 loadClass方法中完成的

通过parent.loadClass 进行向上查找 findClass 向下加载 完成

所以如果要想打破这个机制，则重写 loadClass 方法即可

#### 不同类加载器 中加载的路径是在Launcher中 已经设置好的
```java
/**
 *  可以通过该程序找到 不同加载器对应的目录
 */
public class T003_ClassLoaderScope {
    public static void main(String[] args) {
        String pathBoot = System.getProperty("sun.boot.class.path");
        System.out.println(pathBoot.replaceAll(";", System.lineSeparator()));

        System.out.println("--------------------");
        String pathExt = System.getProperty("java.ext.dirs");
        System.out.println(pathExt.replaceAll(";", System.lineSeparator()));

        System.out.println("--------------------");
        String pathApp = System.getProperty("java.class.path");
        System.out.println(pathApp.replaceAll(";", System.lineSeparator()));
    }
}
```

**手动load一个类**

```java
public class T005_LoadClassByHand {
    public static void main(String[] args) throws ClassNotFoundException {
		// 拿到当前类的ClassLoader  即 APP ClassLoader   然后调用 loadClass ('类的全名')
        Class clazz = T005_LoadClassByHand.class.getClassLoader().loadClass("com.chase.jvm.T002_ClassLoaderLevel");
        System.out.println(clazz.getName());
				
        //利用类加载器加载资源
        //T005_LoadClassByHand.class.getClassLoader().getResourceAsStream("");
    }
}
```

**ClassLoader loadClass 源码**

```java
public abstract class ClassLoader {
	private final ClassLoader parent;
	protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException{
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);   
            if (c == null) {    // 当前类没有load 到
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);   // 如果当前没有load 到 则从父加载器load  递归过程
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                }

                if (c == null) {                  
                    long t1 = System.nanoTime();
                    c = findClass(name);         // 如果所有的加载器都没有缓存到，则自己加载    递归  找到直接返回 
                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
}

//   findClass 的源码
protected Class<?> findClass(String name) throws ClassNotFoundException {
    throw new ClassNotFoundException(name);
}

// 默认没有加载到 class 时会报错
// 所以自定义类加载器的时候只需要重写这个方法就好
//  这个模式  就是模板方法模式----------设计模式（模板方法模式）---逻辑都定义好了  部分需要自己实现

// 面试 模板方法的应用场景    ClassLoader 的loadClass 过程 --- 自定义加载器的时候
```

#### 自定义类加载器

```java
// 一般用于加载指定位置的Class的情况   是class文件  不是java文件
// 继承 ClassLoader
public class T006_MSBClassLoader extends ClassLoader {
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        File f = new File("c:/test/", name.replace(".", "/").concat(".class"));  // 读到Class 文件
        try {
            FileInputStream fis = new FileInputStream(f);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int b = 0;

            while ((b=fis.read()) !=0) {
                baos.write(b);
            }
            byte[] bytes = baos.toByteArray();
            baos.close();
            fis.close();//可以写的更加严谨
						
			// 将 字节数组转换成 class 的类对象   并返回该对象
            return defineClass(name, bytes, 0, bytes.length);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return super.findClass(name); //throws ClassNotFoundException   抛异常
    }
    public static void main(String[] args) throws Exception {
        ClassLoader l = new T006_MSBClassLoader();
        Class clazz = l.loadClass("com.mashibing.jvm.Hello");
        Class clazz1 = l.loadClass("com.mashibing.jvm.Hello");
        System.out.println(clazz == clazz1);
        Hello h = (Hello)clazz.newInstance();
        h.m();
        System.out.println(l.getClass().getClassLoader());
        System.out.println(l.getParent());
        System.out.println(getSystemClassLoader());
    }
}
```
**自定义类加载器的时候，parent 怎么指定？**
```java
public class T010_Parent {
    private static T006_MSBClassLoader parent = new T006_MSBClassLoader();
    private static class MyLoader extends ClassLoader {
		// 重写构造方法，传入指定的parent
        public MyLoader() {
            super(parent);
        }
    }
}
```

**自定义的ClassLoader 默认的classloader是那个？**
```java
// 调用父类的无参构造
protected ClassLoader() {
    this(checkCreateClassLoader(), getSystemClassLoader());
}
// getSystemClassLoader()   通过该行代码 可以获得系统的ClassLoader
```

#### 如果要打破双亲委派机制怎么办？—-不用系统的委派机制
由于委派机制是在loadClass中的 super.loadClass 中进行的

所以删除 super这个逻辑就行，即重写 loadClass方法

自定义加载器：重写findClass ——只是修改了加载class的逻辑

打破双亲委派机制：重写loadClass——不进行委派机制 删除super

打破双亲委派的示例
```java
public class T012_ClassReloading2 {
    private static class MyLoader extends ClassLoader {
        @Override
        public Class<?> loadClass(String name) throws ClassNotFoundException {
            File f = new File("C:/JVM/" + name.replace(".", "/").concat(".class"));
            if(!f.exists()) return super.loadClass(name);
            try {
                InputStream is = new FileInputStream(f);
                byte[] b = new byte[is.available()];
                is.read(b);
                return defineClass(name, b, 0, b.length);
            } catch (IOException e) {
                e.printStackTrace();
            }
            return super.loadClass(name);
        }
    }

    public static void main(String[] args) throws Exception {
        MyLoader m = new MyLoader();
        Class clazz = m.loadClass("com.chase.jvm.Hello");
        m = new MyLoader();
        Class clazzNew = m.loadClass("com.chase.jvm.Hello");
        System.out.println(clazz == clazzNew);
    }
}
```