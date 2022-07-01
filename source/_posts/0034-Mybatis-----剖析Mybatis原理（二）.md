---
title: Mybatis ----- 剖析Mybatis原理（二）
index_img: /img/cover/04.jpg
categories:
  - Mybatis
tags:
  - Mybatis
abbrlink: c7610fd2
date: 2018-09-20 19:51:04
---
### SqlSession 创建过程
我们接下来要看看 SqlSession 的创建过程和运行过程，首先调用了 sqlSessionFactory.openSession() 方法。该方法默认实现类是 DefaultSqlSessionFactory ，我们看看该方法如何被重写的
![](1.png)
调用了自身的 openSessionFromDataSource 方法，注意，参数中 configuration 获取了默认的执行器 “SIMPLE”，自动提交我们没有配置，默认是false，我们进入到 openSessionFromDataSource 方法查看：
![](2.png)
该方法以下几个步骤：

1、获取配置文件中的环境，也就是我们配置的 标签，并根据环境获取事务工厂，事务工厂会创建一个事务对象，而 configurationye 则会根据事务对象和执行器类型创建一个执行器。最后返回一个默认的 DefaultSqlSession 对象。

2、可以说，这段代码，就是根据配置文件创建 SqlSession 的核心地带。我们一步步看代码，首先从配置文件中取出刚刚解析的环境对象。
![](3.png)
然后根据环境对象获取事务工厂，如果配置文件中没有配置，则创建一个 ManagedTransactionFactory 对象直接返回。否则调用环境对象的 getTransactionFactory 方法，该方法和我们配置的一样返回了一个 JdbcTransactionFactory，而实际上，TransactionFactory 只有2个实现类，一个是 ManagedTransactionFactory ，一个是 JdbcTransactionFactory。

我们回到 openSessionFromDataSource 方法，获取了 JdbcTransactionFactory 后，调用 JdbcTransactionFactory 的 newTransaction 方法创建一个事务对象，参数是数据源，level 是null， 自动提交还是false。newTransaction 创建了一个 JdbcTransaction 对象，我们看看该类的构造：
![](4.png)
可以看到，该类都是有关连接和事务的方法，比如commit，openConnection，rollback，和JDBC 的connection 功能很相似。而我们刚刚看到的level是什么呢?在源码中我们看到了答案：
![](5.png)
就是 “事务的隔离级别”。并且该事务对象还包含了JDBC 的Connection 对象和 DataSource 数据源对象，好亲切啊，可见这个事务对象就是JDBC的事务的封装。

继续回到 openSessionFromDataSource 方，法此时已经创建好事务对象。接下来将事务对象执行器作为参数执行 configuration 的 newExecutor 方法来获取一个 执行器类。

我们看看该方法实现：
![](6.png)
首先，该方法判断给定的执行类型是否为null，如果为null，则使用默认的执行器， 也就是 ExecutorType.SIMPLE，然后根据执行的类型来创建不同的执行器，默认是 SimpleExecutor 执行器，这里楼主需要解释以下执行器：

Mybatis有三种基本的Executor执行器，SimpleExecutor、ReuseExecutor、BatchExecutor。

1、SimpleExecutor：每执行一次update或select，就开启一个Statement对象，用完立刻关闭Statement对象。

2、ReuseExecutor：执行update或select，以sql作为key查找Statement对象，存在就使用，不存在就创建，用完后，不关闭Statement对象，而是放置于Map<String, Statement>内，供下一次使用。简言之，就是重复使用Statement对象。

3、BatchExecutor：执行update（没有select，JDBC批处理不支持select），将所有sql都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个Statement对象，每个Statement对象都是addBatch()完毕后，等待逐一执行executeBatch()批处理。与JDBC批处理相同。

作用范围：Executor的这些特点，都严格限制在SqlSession生命周期范围内。

我们再看看默认执行器的构造方法，2个参数，一个是 Configuration， 一个是事务对象。该构造器调用了父类 BaseExecutor 的构造器，我们看看该方法实现：
![](7.png)
该类包装了事务对象，延迟加载的队列，本地缓存，永久缓存，配置对象，还包装了自己。

回到 newExecutor 方法，判断是否使用缓存，默认是true， 则将刚刚的执行器包装到新的 CachingExecutor 缓存执行器中。最后将执行器添加到所有的拦截器中（如果配置了话），我们这里没有配置。
![](8.png)
现在，我们回到 openSessionFromDataSource 方法，我们已经有了执行器，此时创建 DefaultSqlSession 对象，携带 configuration, executor, autoCommit 三个参数，该构造器就是简单的赋值过程。我们有必要看看该类的结构：

![](9.png)
该类包含了常用的所有方法，包括事务方法，可以说，该类封装了执行器和事务类。而执行器才是具体的执行工作人员。至此，我们已经完成了 SqlSession 的创建过程。

接下来，就要看看他的执行过程。
### SqlSession 执行过程
![](10.png)
![](11.png)
如何扩展呢？如果返回值是null，则抛出异常，JDK中HashMap 可是不抛出异常的，如果 value是 Ambiguity 类型，也抛出异常，说明 key 值不够清晰。

那么 buildAllStatements 方法做了什么呢？
![](12.png)
注意看注释（大意）：解析缓存中所有未处理的语句节点。当所有的映射器都被添加时，建议调用这个方法，因为它提供了快速失败语句验证。意思是如果链表中任何一个不为空，则抛出异常，是一种快速失败的机制。那么这些是什么时候添加进链表的呢？答案是catch的时候，看代码：
![](13.png)
该方法首先判断是否是集合类型，如果是，则创建一个自定义Map，key是collection，value是集合，如果不是，并且还是数组，则key为array，都不满足则直接返回该对象。那么我们该进入 query 一探究竟：
![](14.png)
进入 CachingExecutor 的query 方法，首先根据参数获取 BoundSql 对象，最终会调用 StaticSqlSource 的 getBoundSql 方法，该方法会构造一个 BoundSql 对象，构造过程是什么样子的呢？
![](15.png)
会有5个属性被赋值，sql语句，参数，参数是我们刚刚传递的，那么SQL 是怎么来的呢，答案是在 XMLConfigBuilder 的 parseConfiguration 方法中，通过层层调用。

最终执行 StaticSqlSource 的构造方法，将mapper 文件中的Sql解析到该类中，最后会将XML 中的 #{id} 构造成一个ParameterMapping 对象

并将配置对象赋值给该类。

回到 BoundSql 的构造器，首先赋值SQL， 参数映射对象数组，参数对象，默认的额外参数，还有一个元数据参数。

回到我们的 getBoundSql 方法：
![](16.png)
我们已经有了参数绑定对象，该对象中有SQL语句，参数。继续向下执行，从该对象获取参数映射集合，如果为空，则再次创建一个 BoundSql 对象。

接着循环参数，先获取 resultMap id，如果有，则从配置对下中获取resultMap 对象，如果不为null，则修改 hasNestedResultMaps 为 true。最后返回 BoundSql 对象。

我们回到 CachingExecutor 的 query 方法， 我们已经有了sql绑定对象， 接下来创建一个缓存key，根据sql绑定对象，方法声明对象，参数对象，分页对象。

注意：mybatis 一级缓存默认为true，二级缓存默认false。创建缓存的过程很简单，就是将所有的参数的key或者id构造该 CacheKey 对象，使该对象唯一。最后执行query方法：
![](17.png)
该方法步骤：

1、获取缓存，如果没u偶，则执行代理执行器的query方法，如果有，且需要清空了，则清空缓存（也就是Map）。

2、如果该方法声明使用缓存并且结果处理器为null，则校验参数，如果方法声明使存储过程，且所有参数有任意一个不是输入类型，则抛出异常。意思是当为存储过程时，确保不能有输出参数。

3、调用 TransactionalCacheManager 事务缓存处理器执行 getObject 方法，如果返回值时null，则调用代理执行器的query方法，最后添加进事务缓存处理器。

我们重点关注代理执行器的query方法，也就是我们 SimpleExecutor 执行器。该方法如下：
![](18.png)
1、首先判断执行器状态是否关闭。

2、判断是否需要清除缓存。

3、判断结果处理器是否为null，如果不是null，则返回null，如果不是，则从本地缓存中取出。

4、如果返回的list不是null，则处理缓存和参数。否则调用queryFromDatabase 方法从数据库查询。

5、如果需要延迟加载，则开始加载，最后清空加载队列。

6、如果配置文件中的缓存范围是声明范围，则清空本地缓存。

7、最后返回list。

可以看出，我们重点要关注的是 queryFromDatabase 方法，其余的方法都是和缓存相关，但如果没有从数据库取出来，缓存也没什么用。进入该方法查看：
![](19.png)
![](20.png)
该方法创建了一个声明处理器，然后调用了 prepareStatement 方法，最后调用了声明处理器的query方法，注意，这个声明处理器有必要说一下：

mybatis 的SqlSession 有4大对象：

1、Executor代表执行器，由它调度StatementHandler、ParameterHandler、ResultSetHandler等来执行对应的SQL。其中StatementHandler是最重要的。

2、StatementHandler的作用是使用数据库的Statement（PreparedStatement）执行操作，它是四大对象的核心，起到承上启下的作用，许多重要的插件都是通过拦截它来实现的。

3、ParamentHandler是用来处理SQL参数的。

4、ResultSetHandler是进行数据集的封装返回处理的，它相当复杂，好在我们不常用它。

好，我们继续查看 configuration 是如何创建 StatementHandler 对象的。我们看看他的 newStatementHandler 方法

首先根据方法声明类型创建一个声明处理器，有最简单的，有预编译的，有存储过程的，在我们这个方法中，创建了一个预编译的方法声明对象，这个对象的构造器对 configuration 等很多参数进行的赋值。我们还是看看吧：
![](21.png)
我们看到了刚刚提到了parameterHandler和resultSetHandler。

回到 newStatementHandler 方法，需要执行下面的拦截器链的pluginAll方法，由于我们这里没有配置拦截器，该方法也就结束了。

拦截器就是实现了Interceptor接口的类，国内著名的分页插件pagehelper就是这个原理，在mybais 源码里，有一个插件使用的例子，我们可以随便看看：
![](22.png)
这里就是动态代理的知识了，获取目标类的接口，最后执行拦截器的invoke方法。有机会和大家再一起探讨如何编写拦截器插件。这里由于篇幅原因就不展开了。

我们回到 newStatementHandler 方法，此时，如果我们有拦截器，返回的应该是被层层包装的代理类，但今天我们没有。返回了一个普通的方法声明器。

执行 prepareStatement 方法，携带方法声明器，日志对象

从事务管理器中获取连接器（该方法中还需要设置是否自动提交，隔离级别）。如果我们的事务日志是debug级别，则创建一个日志代理对象，代理Connection。

回到 prepareStatement 方法，看第二行，开始让预编译处理器预编译sql（也就是让connection预编译），我看看看是如何执行的。注意，我们没有配置timeout。因此返回null。

进入 RoutingStatementHandler 的 prepare 方法，调用了代理类的 PreparedStatementHandler 的prepare方法，该方法实现入下：



该方法以下几个步骤：

实例化SQL，也就是调用connection 启动 prepareStatement 方法。我们熟悉的JDBC方法。

设置超时时间。

设置fetchSize ，作用是，执行查询时，一次从服务器端拿多少行的数据到本地jdbc客户端这里来。

最后返回映射声明处理器。

我们主要看看第一步：



有没有很亲切，我们看到我们在刚开始回忆JDBC编程的 connection.prepareStatement 代码，由此证明mybatis 就是封装了 JDBC。

首先判断是否含有返回主键的功能，如果有，则看 keyColumnNames 是否存在，如果不存在，取第一个列为主键。最后执行else 语句，开始预编译。

注意：此connection 已经被动态代理封装过了，因此会调用 invoke 方法打印日志。最后返回声明处理器对象。

我们回到 SimpleExecutor 的 prepareStatement 方法， 执行第三行 handler.parameterize(stmt)，该方法其实也是委托了 PreparedStatementHandler 来执行，而 PreparedStatementHandler 则委托了 DefaultParameterHandler 执行 setParameters 方法，我们看看该方法：



首先获取参数映射集合，然后从配置对象创建一个元数据对象，最后从元数据对象取出参数值。再从参数映射对象中取出类型处理器，最后将类型处理器和参数处理器关联。

我们看看最后一行代码：



还是JDBC。而这个下标的顺序则就是参数映射的数组下标。

终于，在准备了那么多之后，我们回到 doQuery 方法，有了预编译好的声明处理器，接下来就是执行了。当然还是调用了PreparedStatementHandler 的query方法。



可以看到，直接执行JDBC 的 execute 方法，注意，该对象也被日志对象代理了，做打印日志工作，和清除工作。如果方法名称是 “executeQuery” 则返回 ResultSet 并代理该对象。 否则直接执行。

我们继续看看DefaultResultSetHandler 的 handleResultSets 是如何执行的：



首先调用 getFirstResultSet 方法获取包装过的 ResultSet ，然后从映射器中获取 resultMap 和resultSet，如果不为null，则调用 handleResultSet 方法，将返回值和resultMaps处理添加进multipleResults list中 ，然后做一些清除工作。

最后调用 collapseSingleResultList 方法，该方法内容如下：



如果返回值长度等于1，返回第一个值，否则返回本身。

至此，终于返回了一个List。不容易啊！！！！最后在返回值的时候执行关闭 Statement 等操作。

我们还需要关注一下 SqlSession 的 close 方法，该方法是事务最后是否生效的关键，当然真正的执行者是executor，在 CachingExecutor 的close 方法中：



该方法决定了到底是commit 还是rollback，最后执行代理执行器的 close 方法，也就是 SimpleExecutor 的close方法，该方法内容入下：



首先执行rollback方法，该方法内部主要是清除缓存，校验是否清除 Statements。然后执行 transaction.close()方法，重置事务（重置事务的autoCommit 属性为true），最后调用 connection.close() 方法，和我们JDBC 一样，关闭连接。

但实际上，该connection 被代理了，被 PooledConnection 连接池代理了，在该代理的invoke方法中，会将该connection从连接池集合删除，在创建一个新的连接放在集合中。

最后回到 SimpleExecurtor 的 close 方法中，在执行完事务的close 方法后，在finally块中将所有应用置为null，等待GC回收。清除工作也就完毕了。

到这里 SqlSession的运行就基本结束了。

最后返回到我们的main方法，打印输出。

我们再看看这行代码，这么一行简单的代码里面 mybatis 为我们封装了无数的调用。可不简单。
### 总结
今天我们从一个小demo开始 debug mybatis 源码，从如何加载配置文件，到如何创建SqlSedssionFactory，再到如何创建 SqlSession，再到 SqlSession 是如何执行的，我们知道了他们的生命周期。

其中创建SqlSessionFactory 和 SqlSession 是比较简单的，执行SQL并封装返回值是比较复杂的，因为还需要配置事务，日志，插件等工作。

还记得我们刚开始说的吗？mybatis 做的什么工作?

根据 JDBC 规范 建立与数据库的连接。

通过反射打通Java对象和数据库参数和返回值之间相互转化的关系。

还有Mybatis 的运行过程？

读取配置文件创建 Configuration 对象, 用以创建 SqlSessionFactroy.

SQLSession 的执行过程.

我们也知道了其实在mybatis 层层封装下，真正做事情的是 StatementHandler，他下面的各个实现类分别代表着不同的SQL声明，我们看看他有哪些属性就知道了：



该类可以说囊括了所有执行SQL的必备属性：配置，对象工厂，类型处理器，结果集处理器，参数处理器，SQL执行器，映射器（保存这个SQL 所有相关属性的地方，比放入SQL语句，参数，返回值类型，配置，id，声明类型等等）， 分页对象， 绑定SQL与参数对象。有了这些东西，还有什么SQL执行不了的呢？

当然，StatementHandler 只是 SqlSession 4 大对象的其中之一，还有Executor 执行器，他负责调度 StatementHandler，ParameterHandler，ResultHandler 等来执行对应的SQL，而 StatementHandler 的作用是使用数据库的 Statement(PreparedStatement ) 执行操作。

它是4大对象的核心，起到承上启下的作用。ParameterHandler 就是封装了对参数的处理，ResultHandler 封装了对结果级别的处理。

到这里，我们这篇文章就结束了，当然，大家肯定还想知道 getMapper 的原理是怎么回事，其实我们开始说过，getMapper 更加的面向对象，但也是对上面的代码的封装。