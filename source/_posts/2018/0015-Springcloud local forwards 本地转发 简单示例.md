---
title: Springcloud local forwards 本地转发 简单示例
index_img: /img/cover/15.jpg
categories:
  - Spring Cloud
tags:
  - 本地转发
abbrlink: 7dff1b12
date: 2018-05-21 15:23:16
---
迁移现有应用程序或API时的常见模式是“扼杀”旧端点，用不同的实现慢慢替换它们。Zuul代理是一个有用的工具，因为您可以使用它来处理来自旧端点的客户端的所有流量，但将一些请求重定向到新端点。   ----------摘抄官方文档

zuul  yml配置
```yaml
zuul:
  routes:
    userserver: /userserver/**  #代理user服务path
      path: /first/**
      url: http://first.example.com
    a:
      path: /a/**
      url: forward:/b
    legacy:
      path: /**
      url: legacy-path
```
示例代码:
```java
@RestController
@RequestMapping("/a")
public class AController {
    @RequestMapping("/getA")
    public String getA(){
       return "A";
    }
}
@RestController
@RequestMapping("/b")
public class BController {
    @RequestMapping("/getB")
    public String getA(){
       return "B";
    }
}
```
通过a: path: /a/** url: forward:/b 该配置将请求转发给 b

测试结果:
```
http://localhost:8095/a/getB 返回B

http://localhost:8095/b/getB 返回B
```