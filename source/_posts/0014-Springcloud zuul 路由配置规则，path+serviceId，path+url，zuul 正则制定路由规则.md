---
title: Springcloud zuul 路由配置规则，path+serviceId，path+url，zuul 正则制定路由规则
index_img: /img/cover/14.jpg
categories:
  - Spring Cloud
tags:
  - spring cloud
  - zuul
abbrlink: cb42d89
date: 2018-05-20 01:30:23
---
照搬照抄，记录一下，方便以后查看
原文地址：http://www.bubuko.com/infodetail-2012976.html

zuul路由的几个配置参数

1.静态路由
```
zuul:
routes:
myroute1:
path: /mypath/**
url: http://localhost:8080 (注意这里url要http://开头)
```
2.静态路由+ribbon负载均衡/故障切换
```
zuul:
routes:
myroutes1:
path: /mypath/**
serviceId: myserverId
myserverId:
ribbon:
listOfServers: localhost:8080, localhost:8081
ribbon:
eureka:
enabled: false
```
3.动态路由+ribbon负载均衡/故障切换
```
zuul:
routes:
myroutes1:
path: /mypath/**
serviceId: myserviceId
eureka:
client:
serviceUrl:
defaultZne:xxx
```
4.路由匹配的一些配置
```
stripPrefix=true，转发会过滤掉前缀。
path: /myusers/**，默认时转发到服务的请求是/**，如果stripPrefix=false，转发的请求是/myusers/**
zuul.prefix=/api 会对所有的path增加一个/api前缀

ignoredPatterns: /**/admin/** 过滤掉匹配的url
route:
users: /myusers/** 会匹配所有/myusers/**的url，但由于ignoredPatterns, /myusers/**/admin/**的请求不会被转发，而是直接由zuul里的接口接收

匹配顺序
path:/myusers/**
path:/** 如果是在application.yml中配置的，那么会优先匹配/myusers/**
但如果是applicaiton.properties配置的，那么可能导致/myusers/**被/**覆盖
ignored-Services: ‘*‘ 对于自动发现的services,除了route中明确指定的，其他都会被忽略
```
5.请求头过滤
```
route.sensitiveHeaders: Cookie,Set-Cookie,Authorization
默认就有这三个请求头，意思是不向下游转发请求这几个头
zuul.ignoredHeaders 是一个全局设置，而route.sensitiveHeaders是局部设置
```


zuul过滤器
```
标准的zuul过滤器有4中，分别对应一次路由转发的几个关键点;
pre: 在路由转发之前起作用
routing: 在路由时其作用
post: 在把结果返回给浏览器时起作用
error: 在整个路由阶段，出现异常时起作用
如果要分析前端传来的参数，验证前端身份等对前端参数的操作，显然是用pre过滤器
如果是要对返回给前端的结果进行操作或者分析，显然是用post过滤器
```

编写自定义路由器
```
public class MyFilter extends ZuulFilter{
filterType() 重写，返回这个过滤器的类型
filterOrder() 重写，返回这个过滤器在过滤器链的顺序
shouldFilter() true启动
run() 具体逻辑
} 
```
