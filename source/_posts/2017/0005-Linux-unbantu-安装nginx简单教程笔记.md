---
title: Linux-unbantu-安装nginx简单教程笔记
index_img: /img/cover/05.jpg
categories:
  - Nginx
tags:
  - Nginx
abbrlink: 4e2c3806
date: 2017-12-28 20:10:01
---

注：以管理员权限安装  sudo
### 1. 安装gcc、g++依赖库
    apt-get install build-essential
    apt-get  install libtool
注：gcc and g++分别是GNU的c & c++ 的编译器
### 2.安装pere依赖库
    apt-get apt-get install libpcre3 libpcre3-dev
注：prep是为了让 nginx 支持正则表达式。只是准备，并不安装，是为了避免在64位系统中出现错误
### 3.安装zlib依赖库
    apt-get install  zlib1g-dev
注：zlib一个免费、通用、无损数据压缩库
### 4. 安装ssl依赖库
    apt-get install
    openssl
注：openssl用于传输层的一个工具包----点击打开链接
### 5.安装nginx
    cd/usr/local/src  下载目录
    wget http://nginx.org/download/nginx-1.4.2.tar.gz  下载链接
    tar -zxvf nginx-1.4.2.tar.gz  解压
    cd nginx-1.4.2  解压目录
    ./configure --prefix=/usr/local/nginx 生成makefile  为下一步编译做准备，便于管理
    make   编译
    make install  安装
### 6.nginx启动
    cd/usr/local/niginx--安装目录
    ./sbin/nginx
### 7.nginx停止
    cd/usr/local/niginx--安装目录
    ./sbin/nginx -s stop
### 8.nginx重新加载配置
    cd/usr/local/niginx--安装目录
    ./sbin/nginx -s reload
### 9.nginx指定配置文件
    cd/usr/local/niginx--安装目录
    ./sbin/nginx -c /usr/local/nginx/conf/nginx.conf