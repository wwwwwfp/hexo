---
title: Java spymemcached 简单使用
index_img: /img/cover/03.jpg
categories:
  - Java基础
tags:
  - spymemcached
abbrlink: 7af52029
date: 2017-12-20 16:42:01
---

1. 下载并安装memcached  百度一大把

2. 添加依赖  sbt 或mvn
    
   sbt
    ```java     
    libraryDependencies += "net.spy" % "spymemcached" % "2.12
    ```
   maven
    ```xml
    <dependency>
        <groupId>net.spy</groupId>
        <artifactId>spymemcached</artifactId>
        <version>2.12.3</version>
    </dependency>
    ```
3. 上代码
    ```java
    public class MemcachedUtil {
        // 日志对象
        private static final play.Logger.ALogger logger = play.Logger.of(MemcachedUtil.class);
    
        static final String servers = "127.0.0.1:11211";
     
        public static MemcachedClient mc = null;
     
        public static MemcachedClient getMemcachedClient(){
            if (mc != null) {
                return mc;
            }
            try{
                mc = new MemcachedClient(new ConnectionFactoryBuilder()
                        .setProtocol(ConnectionFactoryBuilder.Protocol.BINARY).build(), AddrUtil.getAddresses(servers));
            }catch(Exception ex){
                logger.error(ex.getMessage());
            }
     
            return mc;
   }
    ```
4. 设值
   ```java
    MemcachedUtil.getMemcachedClient().set(key, 秒, value);
    MemcachedUtil.getMemcachedClient().get(key)
    ```

  
