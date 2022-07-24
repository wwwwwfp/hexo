---
title: Spring Service层本类中调用另一个事务方法不生效原因
index_img: /img/cover/12.jpg
categories:
  - Spring
tags:
  - 事务
abbrlink: 719f306
date: 2019-12-24 14:47:04
---

原文链接：
https://blog.csdn.net/dapinxiaohuo/article/details/52092447

![](1.jpg)

首先调用的是AOP代理对象而不是目标对象，首先执行事务切面，事务切面内部通过TransactionInterceptor环绕增强进行事务的增强，即进入目标方法之前开启事务，退出目标方法时提交/回滚事务