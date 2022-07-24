---
title: 创建Mysql数据库 Cant create database XXX 问题完美解决
index_img: /img/cover/13.jpg
categories:
  - Mysql
tags:
  - Mysql
abbrlink: '67338697'
date: 2020-01-08 14:36:10
---

情况：

创建数据库报错：Can’t create database XXX (errno: 15938145)

排除权限问题

偶然看了一篇文章，说重启系统会自动恢复部分故障的

重启后，启动mysql报错

Can’t connect to local MySQL server through socket ‘/var/lib/mysql/mysql.sock’

翻到入下博客完美解决问题

https://blog.csdn.net/CCCrunner/article/details/97515760