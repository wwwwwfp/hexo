---
title: Redis key 乱码问题（springboot）
index_img: /img/cover/11.jpg
categories:
  - Redis
tags:
  - 乱码
abbrlink: 6661450d
date: 2019-01-31 16:24:23
---

+ **保存到redis中的key  前半段会出现乱码问题**

    ![](1.png)

+ **原来配置：**
    ```javascript
    @Configuration
    @EnableCaching
    public class RedisCacheConfig {
        @Bean
        public CacheManager cacheManager(RedisTemplate<?, ?> redisTemplate) {
        CacheManager cacheManager = new RedisCacheManager(redisTemplate);
        return cacheManager;
        }
     
        @SuppressWarnings("rawtypes")
        @Bean
        public RedisTemplate redisTemplate(RedisConnectionFactory factory){
        RedisTemplate redisTemplate = new RedisTemplate();
            RedisSerializer stringSerializer = new StringRedisSerializer();
            redisTemplate.setConnectionFactory(factory);
            return redisTemplate;
        }
    }
    ```
+ **到  redisTemplate方法中添加如下代码段**
    ```java
    @SuppressWarnings("rawtypes")
    @Bean
    public RedisTemplate redisTemplate(RedisConnectionFactory factory){
    RedisTemplate redisTemplate = new RedisTemplate();
        RedisSerializer stringSerializer = new StringRedisSerializer();
        redisTemplate.setConnectionFactory(factory);
        redisTemplate.setKeySerializer(stringSerializer);
        redisTemplate.setValueSerializer(stringSerializer);
        redisTemplate.setHashKeySerializer(stringSerializer);
        redisTemplate.setHashValueSerializer(stringSerializer);
        return redisTemplate;
    }
    ```
+ 修改后key值显示正常

  参考：https://blog.csdn.net/xiaolyuh123/article/details/78682200