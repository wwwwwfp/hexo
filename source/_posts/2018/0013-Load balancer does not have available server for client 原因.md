---
title: >-
  springcloud zuul Forwarding error 错误 Load balancer does not have available
  server for client 原因
index_img: /img/cover/13.jpg
categories:
  - Spring Cloud
tags:
  - spring cloud
  - zuul
abbrlink: 63a9df37
date: 2018-05-19 16:22:06
---
异样错误：
  
```java
exception.ZuulException: Forwarding error
com.netflix.client.ClientException: Load balancer does not have available server for client
```

  刚开始用Api gateway   zuul组建，错误如上，

  网上查了资料，主要错误原因如下：

1. 缺 eureka 依赖，因为@EnableZuulProxy 是一个组合注解

![](1.jpg)

2. 第二个原因也是我遇到的，同时到访问服务中配置了spring.application.name 和 eureka.instance.appname , 注册到eureka的application name 为 eureka.instance.appname。 我再访问的时候直接复制的eureka服务中的application，然后就报以上错误。 最后把eureka.instance.appname的配置去掉后，访问正常。 具体原因不清楚，哪路大神如果清楚的话欢迎骚扰，谢谢！！！
