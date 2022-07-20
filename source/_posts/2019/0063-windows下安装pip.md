---
title: windows下安装pip
index_img: /img/cover/03.jpg
categories:
  - 工具类
tags:
  - pip
abbrlink: 73a1ea38
date: 2019-09-03 10:05:28
---
### 前提：
python 环境正常
### 下载并解压pip压缩包 
https://pypi.python.org/pypi/pip#downloads
![](1.png)
### Dos窗口进入解压目录执行 python setup.py install ，最后出现Finished processing dependencies for pip == 版本号即标识安装成功
![](2.png)
### 配置环境变量
安装完成后，会输出pip的安装路径，找到lib的同级目录Scripts，复制该目录并添加到环境变量
![](3.png)
![](4.png)
### 验证：pip list
![](5.png)
### 应用：安装 PyMySQL
![](6.png)