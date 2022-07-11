---
title: >-
  Springcloud zuul上传文件 zuul GENERAL 错误 The field file exceeds its maximum
  permitted size of bytes.
index_img: /img/cover/16.jpg
categories:
  - Spring Cloud
tags:
  - zuul
  - spring cloud
abbrlink: d6560e4
date: 2018-05-21 17:31:48
---
springboot 中tomcat对文件的大小做了限制，文档中有说明

springboot对properties或yml的配置在不同版本是有差异的。

maxFileSize：单个文件最大限制

maxRequestSize：单次请求最大限制

如果不做限制都传-1

#### 1. 1.4版本之前配置方式：
```
multipart.maxFileSize=2000Mb

multipart.maxRequestSize=2500Mb
```
#### 2. 1.4版本后修改为:
``` 
spring.http.multipart.maxFileSize=2000Mb

spring.http.multipart.maxRequestSize=2500Mb
```
#### 3. 2.0版本之后修改该为:
```
spring.http.multipart.max-file-size=2000Mb

spring.http.multipart.max-request-size=2500Mb
```
#### 4. 1.4之前部分版本需要添加一个配置类

```java
public class FileUploadConfig {     

     @Bean
    public MultipartConfigElement multipartConfigElement(
                            @Value("${multipart.maxFileSize}") String maxFileSize,
                            @Value("${multipart.maxRequestSize}") String maxRequestSize) {
        MultipartConfigFactory factory = new MultipartConfigFactory();
        factory.setMaxFileSize(maxFileSize);
        factory.setMaxRequestSize(maxRequestSize);
        return factory.createMultipartConfig();      

    }    

}
```