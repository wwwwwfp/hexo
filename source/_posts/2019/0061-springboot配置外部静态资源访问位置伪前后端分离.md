---
title: springboot 配置外部静态资源访问位置、将所有静态资源分离到项目外部完成伪前后端分离（springboot+thymeleaf）
index_img: /img/cover/01.jpg
categories:
  - SpringBoot
tags:
  - thymeleaf
abbrlink: 6ed10394
date: 2019-08-23 17:43:08
---

```text
通过配置将项目中的静态资源和后台代码分离出来，达到伪前后端分离的目的。
在springboot中对静态资源访问提供了很好的支持

将一个已经成型的小项目中的所有静态资源分离到项目外部
```
**如下为项目原来的目录结构**
![](1.png)
**默认访问项目是正常的**
![](2.png)

+ 注：在Springboot中默认的静态资源路径有：classpath:/META-INF/resources/，classpath:/resources/，classpath:/static/，classpath:/public/，从这里可以看出这里的静态资源路径都是在classpath中（也就是在项目路径下指定的这几个文件夹）
+ 在springboot 只需要做如下两步就可以将静态资源中的

1. 修改themeleaf中的资源配置
    ![](3.png)
2. 新增配置类，配置静态资源的位置到 D 盘中
    ```java
    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        @Override
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
            registry.addResourceHandler("/**").addResourceLocations("file:D:/aaaa/static/");
        }
    }
    ```
**效果图**

![](4.gif)