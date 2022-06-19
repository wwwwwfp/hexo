---
title: Java中使用sfntly的sfnttool-jar简化裁剪ttf字体大小
index_img: /img/cover/09.jpg
categories:
  - 图片文字处理
tags:
  - sfntly
  - sfnttool
  - ttf
abbrlink: b751d45e
date: 2018-02-27 18:39:54
---
#### 1、sfnttool.jar 下载地址：
http://download.csdn.net/download/qq_31482599/10260762

#### 2、（终端）生成命令
    java -jar sfnttool.jar文件路径 -s '字体内容' 原始ttf文件路径 裁剪后ttf文件路径
注：字体内容 单引号表示不允许有空格字符 双引号表示可以有空格字符

#### 3、java代码生成
```java
String content = fontUtil.getContent();
content = content.replaceAll("\\s*",""); //文字内容去除空格字符
Process exec = Runtime.getRuntime().exec(
    "java -jar conf"+File.separator+"sfnttool.jar -s '"+content
    +"' conf"+File.separator+"font"+File.separator+fontFamily+".ttf "+ttf_target);
int i = exec.waitFor(); //当前线程等待 0表示正常终止
```
ttf_target：目标文件

效果图：
![](1.jpg)

#### 4、前端
```html
<html>  
<head>  
<meta charset="utf-8">   
<style type="text/css">  
@font-face {
    font-family: "pomo";
    src: url("D:\\000\\pomo_sub.ttf") format("truetype");
    font-style: normal;
    font-weight: normal;
}
.test {  
    font-family: 'pomo';  
}  
</style>  
</head>  
<body>  
<h1 class="test">测试 English TeST 英语</h1>  
</body>
</html> 
```
示例：
![](2.jpg)




注：某些网络字体ttf裁剪后是看不到效果的！！！！

多试几个就知道了。。。。。。。
