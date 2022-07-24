---
title: Java JDK序列化 （Serializable和Externalizable）
index_img: /img/cover/06.jpg
categories:
  - Java基础
tags:
  - Serializable
  - Externalizable
  - 序列化
abbrlink: a516237a
date: 2019-10-14 14:47:20
---
  
### 概念
```text
序列化：将结构对象转为字节序列的过程。
反序列化：将字节序列恢复为结构对象的过程。
```

#### 为什么要序列化：
简单的说序列化是用来通信的，为了跨进程传递格式化数据（byte流）。
当两个进程在进行远程通信时，彼此可以发送各种类型的数据。无论是何种类型的数据，都会以二进制序列的形式在网络上传送。发送方需要把这个对象转换为字节序列，才能在网络上传送；接收方则需要把字节序列再恢复为对象。
这样客户端和服务端就可以通过序列化的数据在网络上、不同机器上甚至是在同一个进程中通信 （跨平台）。同时通过序列化后的的数据在传输过程中安全性也会相对高一点。
通过序列化来持久化存储。数据在通过序列化操作后将二进制保存到本地的同时还要对这些数据做一些其它处理，比如筛选数据，防止重复存储数据等。
#### 序列化原理：
序列化和反序列化的处理是通过ObjectInputStream和ObjectOutputStream实现的，验证如下：

定义一个Account类如下,怕篇幅过长删除了不必要的空格
```java
public class Account implements Serializable {
    private static final long serialVersionUID = -547309094426427798L;
    private Long id;
    private String account;
    // transient 作用是在序列化的时候忽略当前属性的值，可以屏蔽掉一些敏感或者没用的数据
    // 可以从后面的验证结果输出的内容看到具体效果
    transient private String password;
    private String email;
    //-----------省略get set 方法-----------
    @Override
    public String toString() {
        return "Account{" +
                "id=" + id +
                ", account='" + account + '\'' +
                ", password='" + password + '\'' +
                ", email='" + email + '\'' +
                '}';
    }
}
```
定义序列化和反序列化的工具类
```java
public class SerializableUtil {
    /**
     * 序列化方法
     * @param obj
     * @param fileName
     * @throws IOException
     */
    public static void serialize(Object obj,String fileName) throws IOException {
        FileOutputStream fos = new FileOutputStream(fileName);
        ObjectOutputStream oos = new ObjectOutputStream(fos);
        oos.writeObject(obj);
        oos.close();
        fos.close();
    }
    /**
     * 反序列化方法
     * @param fileName
     * @return
     * @throws Exception
     */
    public static Object deserialize(String fileName) throws Exception {
        FileInputStream fis = new FileInputStream(fileName);
        ObjectInputStream ois = new ObjectInputStream(fis);
        Object object = ois.readObject();
        ois.close();
        fis.close();
        return object;
    }
    public static void main(String[] args) throws Exception {
        Account account = new Account();
        account.setId(1L);
        account.setAccount("chase");
        account.setEmail("wfp_chase@163.com");
        account.setPassword("********");
        serialize(account,"account");
        Account result = (Account) deserialize("account");
        System.out.println(account);
        System.out.println(result);
    }
}
```
验证结果如下：
```text
在实现Serializable  接口的情况下
	Account{id=1, account='chase', password='********', email='wfp_chase@163.com'}
	Account{id=1, account='chase', password='null', email='wfp_chase@163.com'}
	
如果删除 Account中的  implements Serializable  运行结果则会抛异常
	Exception in thread "main" java.io.NotSerializableException: test.serializable.Account
		at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1184)
		at java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:348)
		at test.serializable.SerializableUtil.serialize(SerializableUtil.java:20)
		at test.serializable.SerializableUtil.main(SerializableUtil.java:44)
```
在没有实现Serializable的情况报错的原因如下：

因为是在序列化的过程中报错的，可以从oos.writeObject(obj); 方法中断点找原因

在源码中可以看到 obj 不属于 以上条件的任何一种，所以只能抛出NotSerializableException异常。
![](1.png)

### serialVersionUID
#### 作用：
在序列化中是通过类的serialVersionUID来验证版本一致的

字节流中的serialVersionUID于本地相应实体类的serialVersionUID进行比较，如果一致，则可以反序列化，否则会报InvalidCastException异常。所以在创建类时如果需要进行序列化操作，最好是显示设置serialVersionUID的值。
#### 过程：
在进行序列化时，如果serialVersionUID在类中不显示设置，会根据类的诸多相关的信息（属性、名称、方法等等）自动计算生成一个。由于计算规则是固定的，所以在编译后的class文件没有发生变化时，serialVersionUID 的值始终是一样的
#### 验证：
如果将Account中的serialVersionUID注释掉，再将原来的accout反序列化时就会报如下错误
```text
Exception in thread "main" java.io.InvalidClassException: test.serializable.Account; local class incompatible: stream classdesc serialVersionUID = 7640500986923754437, local class serialVersionUID = -547309094426427798
```
#### 部分源码
```java
// ObjectStreamClass.java
/**
 * 获取serialVersionUID 的方法
 * 其中 suid 表示类的serialVersionUID（如果尚未计算，则为null）
 */
public long getSerialVersionUID() {
        // REMIND: synchronize instead of relying on volatile?
        if (suid == null) {
            suid = AccessController.doPrivileged(
                new PrivilegedAction<Long>() {
                    public Long run() {
						/**
						 * 计算给定类的默认串行版本UID值。
						 */
                        return computeDefaultSUID(cl);
                    }
                }
            );
        }
        return suid.longValue();
    }
```
### 序列化中继承和静态变量序列化
#### 继承问题
+ 如果父类实现序列化，子类自动实现序列化，不需要显式实现Serializable接口。
+ 如果一个子类实现了 Serializable 接口，其父类都没有实现 Serializable 接口，要想将父类对象也序列化，就需要让父类也实现Serializable 接口。

原因：
如果父类不实现 Serializable接口的话，就需要有默认的无参的构造函数。这是因为一个 Java 对象的构造必须先有父对象，才有子对象，反序列化也不例外。在反序列化时，为了构造父对象，只能调用父类的无参构造函数作为默认的父对象。因此当我们取父对象的变量值时，它的值是调用父类无参构造函数后的值。在这种情况下，在序列化时根据需要在父类无参构造函数中对变量进行初始化，否则的话，父类变量值都是默认声明的值，如 int 型的默认是 0，string 型的默认是 null。
#### 静态变量序列化
序列化保存的是对象的状态，静态变量属于类的状态，因此 序列化并不保存静态变量。
### Java序列化之Externalizable
使用Externalizable时，需要实现如下两个方法，用于控制序列化和反序列化的类中属性字段，需要注意的是 transient 关键字将不会在这种情况下起作用。

void writeExternal(ObjectOutput out) throws IOException;

void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;

Externalizable 是继承了 Serializable 所以在序列化时本质和实现是一样的都是通过ObjectOutputSteam和ObjectInputStream处理的。

```java
public interface Externalizable extends java.io.Serializable {
    void writeExternal(ObjectOutput out) throws IOException;
    void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
}
```
![](2.png)


