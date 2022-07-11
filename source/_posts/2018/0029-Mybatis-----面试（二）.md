---
title: Mybatis ----- 面试（二）
index_img: /img/cover/29.jpg
categories:
  - Mybatis
tags:
  - Mybatis
abbrlink: 9963c2d9
date: 2018-09-13 14:23:39
---
### Mybatis的Xml映射文件中，不同的Xml映射文件，id是否可以重？
不同的Xml映射文件，如果配置了namespace，那么id可以重复；如果没有配置namespace，那么id不能重复；毕竟namespace不是必须的，只是最佳实践而已。

原因就是namespace+id是作为Map<String, MappedStatement>的key使用的，如果没有namespace，就剩下id，那么，id重复会导致数据互相覆盖。

有了namespace，自然id就可以重复，namespace不同，namespace+id自然也就不同。

### 为什么说Mybatis是半自动ORM映射工具？它与全自动的区别在哪里？
Hibernate属于全自动ORM映射工具，使用Hibernate查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的。而Mybatis在查询关联对象或关联集合对象时，需要手动编写sql来完成，所以，称之为半自动ORM映射工具。

### 一对一、一对多的关联查询 ？
```xml
<mapper namespace="com.lcb.mapping.userMapper">  
 
   <!--association  一对一关联查询 -->  
   <select id="getClass" parameterType="int" resultMap="ClassesResultMap">  
       select * from class c,teacher t where c.teacher_id=t.t_id and c.c_id=#{id}  
   </select>  
 
   <resultMap type="com.lcb.user.Classes" id="ClassesResultMap">  
       
       <!-- 实体类的字段名和数据表的字段名映射 -->  
       <id property="id" column="c_id"/>  
       <result property="name" column="c_name"/>  
       <association property="teacher" javaType="com.lcb.user.Teacher">  
           <id property="id" column="t_id"/>  
           <result property="name" column="t_name"/>  
       </association>  
   </resultMap>  
 
 
 
   <!--collection  一对多关联查询 -->  
 
   <select id="getClass2" parameterType="int" resultMap="ClassesResultMap2">  
       select * from class c,teacher t,student s where c.teacher_id=t.t_id and c.c_id=s.class_id and c.c_id=#{id}  
 
   </select>  
 
   <resultMap type="com.lcb.user.Classes" id="ClassesResultMap2">  
 
       <id property="id" column="c_id"/>  
       <result property="name" column="c_name"/>  
 
       <association property="teacher" javaType="com.lcb.user.Teacher">  
           <id property="id" column="t_id"/>  
           <result property="name" column="t_name"/>  
       </association>  
       
       <collection property="student" ofType="com.lcb.user.Student">  
           <id property="id" column="s_id"/>  
           <result property="name" column="s_name"/>  
       </collection>  
   </resultMap>  
</mapper>
```

### Xml映射文件中，除了常见的select|insert|updae|delete标签之外，还有哪些标签？

答：还有很多其他的标签，<resultMap>、<parameterMap>、<sql>、<include>、<selectKey>，加上动态sql的9个标签，trim|where|set|foreach|if|choose|when|otherwise|bind等，其中<sql>为sql片段标签，通过<include>标签引入sql片段，<selectKey>为不支持自增的主键生成策略标签。

### Mybatis是如何进行分页的？分页插件的原理是什么？
答：Mybatis使用RowBounds对象进行分页，它是针对ResultSet结果集执行的内存分页，而非物理分页，可以在sql内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页。

分页插件的基本原理是使用Mybatis提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的sql，然后重写sql，根据dialect方言，添加对应的物理分页语句和物理分页参数。

举例：select * from student，拦截sql后重写为：select t.* from （select * from student）t limit 0，10

### 简述Mybatis的插件运行原理，以及如何编写一个插件。
Mybatis仅可以编写针对ParameterHandler、ResultSetHandler、StatementHandler、Executor这4种接口的插件，Mybatis使用JDK的动态代理，为需要拦截的接口生成代理对象以实现接口方法拦截功能，每当执行这4种接口对象的方法时，就会进入拦截方法，具体就是InvocationHandler的invoke()方法，当然，只会拦截那些你指定需要拦截的方法。

实现Mybatis的Interceptor接口并复写intercept()方法，然后在给插件编写注解，指定要拦截哪一个接口的哪些方法即可，记住，别忘了在配置文件中配置你编写的插件。

### Mybatis执行批量插入，能返回数据库主键列表吗？
能，JDBC都能，Mybatis当然也能。

### Mybatis能执行一对一、一对多的关联查询吗？都有哪些实现方式，以及它们之间的区别。
能，Mybatis不仅可以执行一对一、一对多的关联查询，还可以执行多对一，多对多的关联查询，多对一查询，其实就是一对一查询，只需要把selectOne()修改为selectList()即可；

多对多查询，其实就是一对多查询，只需要把selectOne()修改为selectList()即可。

关联对象查询，有两种实现方式，一种是单独发送一个sql去查询关联对象，赋给主对象，然后返回主对象。

另一种是使用嵌套查询，嵌套查询的含义为使用join查询，一部分列是A对象的属性值，另外一部分列是关联对象B的属性值，好处是只发一个sql查询，就可以把主对象和其关联对象查出来。

那么问题来了，join查询出来100条记录，如何确定主对象是5个，而不是100个？

其去重复的原理是<resultMap>标签内的<id>子标签，指定了唯一确定一条记录的id列，Mybatis根据<id>列值来完成100条记录的去重复功能，<id>可以有多个，代表了联合主键的语意。

同样主对象的关联对象，也是根据这个原理去重复的，尽管一般情况下，只有主对象会有重复记录，关联对象一般不会重复。

      |       t_id | t_name       |     s_id|
      |          1 | teacher      |      38 |
      |          1 | teacher      |      39 |
      |          1 | teacher      |      40 |
      |          1 | teacher      |      41 |
      |          1 | teacher      |      42 |
      |          1 | teacher      |      43 |

### Mybatis是否支持延迟加载？如果支持，它的实现原理是什么？
Mybatis仅支持association关联对象和collection关联集合对象的延迟加载，association指的就是一对一，collection指的就是一对多查询。

在Mybatis配置文件中，可以配置是否启用延迟加载lazyLoadingEnabled=true|false。

它的原理是，使用CGLIB创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，比如调用a.getB().getName()，拦截器invoke()方法发现a.getB()是null值，那么就会单独发送事先保存好的查询关联B对象的sql，把B查询上来，然后调用a.setB(b)，于是a的对象b属性就有值了，接着完成a.getB().getName()方法的调用。这就是延迟加载的基本原理。

当然了，不光是Mybatis，几乎所有的包括Hibernate，支持延迟加载的原理都是一样的。

### Mybatis的Xml映射文件中，不同的Xml映射文件，id是否可以重复？
不同的Xml映射文件，如果配置了namespace，那么id可以重复；如果没有配置namespace，那么id不能重复；毕竟namespace不是必须的，只是最佳实践而已。

原因就是namespace+id是作为Map<String, MappedStatement>的key使用的，如果没有namespace，就剩下id，那么，id重复会导致数据互相覆盖。有了namespace，自然id就可以重复，namespace不同，namespace+id自然也就不同。