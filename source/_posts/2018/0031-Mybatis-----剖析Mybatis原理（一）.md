---
title: Mybatis ----- 剖析Mybatis原理（一）
index_img: /img/cover/01.jpg
categories:
  - Mybatis
tags:
  - Mybatis
abbrlink: f777e30a
date: 2018-09-13 14:34:41
---
### 从 Mybatis 的一个 Demo 案例开始

从 github 上 clone 了mybatis 的源码，过程比Spring源码顺利，主要注意一点：在 IDEA 编辑器中（Eclipse 楼主不知道），需要排除 src/test/java/org/apache/ibatis/submitted 包，防止编译错误

在源码中写了一个Demo，给大家看一下目录结构
![](1.png)
![](2.png)

图片中的红框部分是楼主自己新增的，然后看看代码：
![](3.png)
![](4.png)

再看看 mybatis-config.xml 配置文件：
```xml
<?xml version="1.0" encoding="UTF-8" ?><!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd"><configuration>
 
  <properties><!--定义属性值-->
    <property name="driver" value="com.microsoft.sqlserver.jdbc.SQLServerDriver"/>
    <property name="url" value="jdbc:sqlserver://192.168.0.122:1433;DatabaseName=test"/>
    <property name="username" value="sa"/>
    <property name="password" value="434343"/>
  </properties>
 
  <settings>
    <setting name="cacheEnabled" value="true"/>
  </settings>
 
  <!-- 类型别名 -->
  <typeAliases>
    <typeAlias alias="userInfo" type="org.apache.ibatis.mybatis.UserInfo"/>
  </typeAliases>
 
  <!--环境-->
  <environments default="development">
    <environment id="development"><!--采用jdbc 的事务管理模式-->
      <transactionManager type="JDBC">
        <property name="..." value="..."/>
      </transactionManager>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
 
  <!--映射器  告诉 MyBatis 到哪里去找到这些语句-->
  <mappers>
    <mapper resource="UserInfoMapper.xml"/>
  </mappers>
</configuration>
```

UserInfoMapper.xml 配置文件
```xml
<?xml version="1.0" encoding="UTF-8" ?><!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd" ><mapper namespace="org.apache.ibatis.mybatis.UserInfoMapper">
 
  <select id="selectById" parameterType="int" resultType="org.apache.ibatis.mybatis.UserInfo">
    SELECT * FROM user_info  WHERE  id = #{id}  </select>
</mapper>
```

运行一下测试类：
![](5.png)

结果正确，打印了2次，因为我们使用了两种不同的方式来执行SQL。

那么，我们就从这个简单的例子来看看 Mybatis 是如何运行的

### 深入源码之前的理论知识
再深入源码之前，楼主想先来一波理论知识，避免因进入源码的汪洋大海导致迷失方向。

首先, Mybatis 的运行可以分为2个部分,第一部分是读取配置文件创建 Configuration 对象, 用以创建 SqlSessionFactroy, 第二部分是 SQLSession 的执行过程。

我们再来看看我们的测试代码:
![](6.png)
这是一个和我们平时使用不同的方式, 但如果细心观察,会发现, 实际上在 Spring 和 Mybatis 整合的框架中也是这么使用的, 只是 Spring 的 IOC 机制帮助我们屏蔽了创建对象的过程而已. 如果我们忘记创建对象的过程, 这段代码就是我们平时使用的代码。

那么,我们就来看看这段代码, 首先创建了一个流, 用于读取配置文件, 然后使用流作为参数, 使用 SqlSessionaFactoryBuilder 创建了一个 SqlSessionFactory 对象,然后使用该对象获取一个 SqlSession, 调用 SqlSession 的 selectOne 方法 获取了返回值,或者 调用了 SqlSession 的 getMapper 方法获取了一个代理对象, 调用代理对象的 selectById 方法 获取返回值。

在这里, 楼主觉得有必要讲讲这几个类的生命周期:

1、SqlSessionaFactoryBuilder 该类主要用于创建 SqlSessionFactory, 并给与一个流对象, 该类使用了创建者模式, 如果是手动创建该类(这种方式很少了,除非像楼主这种测试代码), 那么建议在创建完毕之后立即销毁。

2、SqlSessionFactory 该类的作用了创建 SqlSession, 从名字上我们也能看出, 该类使用了工厂模式, 每次应用程序访问数据库, 我们就要通过 SqlSessionFactory 创建 SqlSession, 所以SqlSessionFactory 和整个 Mybatis 的生命周期是相同的。

这也告诉我们不同创建多个同一个数据的 SqlSessionFactory, 如果创建多个, 会消耗尽数据库的连接资源, 导致服务器夯机. 应当使用单例模式. 避免过多的连接被消耗, 也方便管理.

3、SqlSession 那么是什么 SqlSession 呢? SqlSession 相当于一个会话, 就像 HTTP 请求中的会话一样, 每次访问数据库都需要这样一个会话, 大家可能会想起了 JDBC 中的 Connection, 很类似,但还是有区别的, 何况现在几乎所有的连接都是使用的连接池技术, 用完后直接归还而不会像 Session 一样销毁。

注意:他是一个线程不安全的对象, 在设计多线程的时候我们需要特别的当心, 操作数据库需要注意其隔离级别, 数据库锁等高级特性。

此外, 每次创建的 SqlSession 都必须及时关闭它, 它长期存在就会使数据库连接池的活动资源减少,对系统性能的影响很大, 我们一般在 finally 块中将其关闭. 还有, SqlSession 存活于一个应用的请求和操作,可以执行多条 Sql, 保证事务的一致性。

Mapper 映射器， 正如我们编写的那样, Mapper 是一个接口, 没有任何实现类, 他的作用是发送 SQL, 然后返回我们需要的结果。

或者执行 SQL 从而更改数据库的数据, 因此它应该在 SqlSession 的事务方法之内, 在 Spring 管理的 Bean 中, Mapper 是单例的。

大家应该还看见了另一种方式， 就是上面的我们不常见到的方式，其实， 这个方法更贴近Mybatis底层原理，只是该方法还是不够面向对象， 使用字符串当key的方式也不易于IDE 检查错误。我们常用的还是getMapper方法。

### 开始深入源码
我们一行一行看。

首先根据maven的classes目录下的配置文件并创建流，然后创建 SqlSessionFactoryBuilder 对象，该类结构如下：
![](7.png)
可以看到该类只有一个方法并且被重载了9次，而且没有任何属性，可见该类唯一的功能就是通过配置文件创建 SqlSessionFactory。那我们就紧跟来看看他的build方法：
![](8.png)
该方法，默认环境为null， 属性也为null，调用了自己的另一个重载build方法，我们看看该方法。
```java
/**
   * 构建SqlSession 工厂
   *
   * @param inputStream xml 配置文件
   * @param environment 默认null
   * @param properties 默认null
   * @return 工厂
   */
  public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
    try {
      // 创建XML解析器
      XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
      // 创建 session 工厂
      return build(parser.parse());
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
      ErrorContext.instance().reset();
      try {
        inputStream.close();
      } catch (IOException e) {
        // Intentionally ignore. Prefer previous error.
      }
    }
  }
```
可以看到该方法只有2个步骤，第一，根据给定的参数创建一个 XMLConfigBuilder XML配置对象，第二，调用重载的 build 方法。

并将上一行返回的 Configuration 对象作为参数。我们首先看看创建 XMLConfigBuilder 的过程。
![](9.png)

首先还是调用了自己的构造方法，参数是 XPathParser 对象， 环境（默认是null），Properties （默认是null），然后调用了父类的构造方法并传入 Configuration 对象，注意，Configuration 的构造器做了很多的工作，或者说他的默认构造器做了很多的工作。

我们看看他的默认构造器：
![](10.png)
该构造器主要是注册别名，并放入到一个HashMap中，这些别名在解析XML配置文件的时候会用到。如果平时注意mybatis配置文件的话，这些别名应该都非常的熟悉了。

我们回到 XMLConfigBuilderd 的构造方法中，也就是他的父类 BaseBuilder 构造方法，该方法如下：
![](11.png)
主要是一些赋值过程，主要将刚刚创建的 Configuration 对象和他的属性赋值到 XMLConfigBuilder 对象中。

我们回到 SqlSessionFactoryBuilder 的 build 方法中，此时已经创建了 XMLConfigBuilder 对象，并调用该对象的 parse 方法，我们看看该方法实现：
![](12.png)
首先判断了最多只能解析一次，然后调用 XPathParser 的 evalNode 方法，该方法返回了 XNode 对象 ，而XNode 对象就和我们平时使用的 Dom4j 的 node 对象差不多，我们就不深究了，总之是解析XML 配置文件，加载 DOM 树，返回 DOM 节点对象。然后调用 parseConfiguration 方法。

我们看看该方法：
![](13.png)
该方法的作用是解析刚刚的DOM节点，可以看到我们熟悉的一些标签。

比如：properties，settings，objectWrapperFactory，mappers。我们重点看看最后一行 mapperElement 方法，其余的方法，大家如果又兴趣自己也可以看看。

mapperElement 方法如下：
![](14.png)
该方法循环了 mapper 元素，如果有 “package” 标签，则获取value值，并添加进映射器集合Map中，该Map如何保存呢，找到包所有class，并将Class对象作为key，MapperProxyFactory 对象作为 value 保存。

MapperProxyFactory 类中有2个属性，一个是 Class mapperInterface ，也就是接口的类名，一个 Map<Method, MapperMethod> methodCache 方法缓存。

我们回到 XMLConfigBuilder 的 mapperElement 方法中， 如果没有 “package” 属性，则尝试获取 “resource”， “url”，“class”属性，并一个个判断。

最后都会和 “package”方法一样，调用 configuration.addMapper 方法。将 namespace 属性和配置文件关联。

在执行完 parseConfiguration 方法后，也就完成了 XMLConfigBuilder 对象的 parse 方法，调用重载方法 build ：

![](15.png)

返回了一个默认的 DefaultSqlSessionFactory 对象。

至此，解析配置文件的工作就结束了，此时创建了 SqlSessionFactory 对象和 Configuration 对象，这两个对象都是单例的，且他们的声明周期和 Mybatis 是一致的。

Configuration 对象中包含了 Mybatis 配置文件中的所有信息，在后面大有用处，SqlSessionFactory 将创建后面所有的SqlSession对象，可见其重要性。

可以看到，创建 SqlSessionFactory 对象是比较简单的，然后，SqlSession 的执行过程就不那么简单了。

好了由于篇幅过长，下一届继续讲解SqlSession 创建过程和SqlSession 执行过程。