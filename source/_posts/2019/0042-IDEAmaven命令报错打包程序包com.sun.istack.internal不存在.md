---
title: IDEA maven命令报错：打包程序包com.sun.istack.internal不存在
index_img: /img/cover/12.jpg
categories:
  - Maven
tags:
  - maven
abbrlink: 919dda13
date: 2019-03-12 11:08:40
---

+ **错误信息**

    ![](20190311194252146.png)

+ **错误原因:**

  因使用到 @NotNull注解导致，引入相应的包即可
+ **解决方案：pom文件中加入如下配置**
  ```xml
  <build>
          <plugins>
              <plugin>
                  <groupId>org.apache.maven.plugins</groupId>
                  <artifactId>maven-compiler-plugin</artifactId>
                  <configuration>
                      <source>${java.version}</source>
                      <target>${java.version}</target>
                      <compilerArgs>
                          <!-- 过期方法警告 -->
                          <arg>-Xlint:deprecation</arg>
                      </compilerArgs>
                      <compilerArguments>
                          <!-- 解决maven命令编译报错，因为rt.jar 和jce.jar在jre的lib下面，不在jdk的lib下面，
                          导致maven找不到（java7以后会出现这个问题）-->
                          <bootclasspath>${java.home}\lib\rt.jar;${java.home}\lib\jce.jar</bootclasspath>
                      </compilerArguments>
                  </configuration>
              </plugin>
          </plugins>
      </build>
  
  ```
+ 修改后key值显示正常

  参考：https://blog.csdn.net/xiaolyuh123/article/details/78682200