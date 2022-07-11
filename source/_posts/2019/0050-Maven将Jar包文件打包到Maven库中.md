---
title: Maven 将Jar包文件打包到Maven库中
index_img: /img/cover/20.jpg
categories:
  - Maven
tags:
  - mvn
abbrlink: 24be83cd
date: 2019-05-20 14:04:15
---
  
### mvn命令：
```text
mvn install:install-file -Dfile=sms.jar -DgroupId=com.test.sms -DartifactId=sms -Dversion=1.0.0 -Dpackaging=jar
```
+ -Dfile=sms.jar：文件所在路径

+ -DgroupId=com.test.sms：依赖groupId

+ -DartifactId=sms：artifactId配置

+ -Dversion=1.0.0：版本配置

+ -Dpackaging=jar：打包方式


### 依赖

```xml
<dependency>
    <groupId>com.test.sms</groupId>
    <artifactId>sms</artifactId>
    <version>1.0.0</version>
</dependency> 
```
