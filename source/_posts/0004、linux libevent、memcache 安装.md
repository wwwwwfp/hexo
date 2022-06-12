---
title: linux libevent、memcache 安装
index_img: /img/cover/04.jpg
categories:
  - 缓存管理
tags:
  - libevent
  - memcache
abbrlink: 13af38ca
date: 2017-12-21 11:09:33
---
**1. libevent安装：**

**1）到http://monkey.org/~provos/libevent/下找到对应的安装包**

![img.png](source/_posts/img/0004/img.png)

**2）下载**

**#wget[https://github.com/libevent/libevent/releases/download/release-2.1.8-stable/libevent-2.1.8-stable.tar.gz](https://github.com/libevent/libevent/releases/download/release-2.1.8-stable/libevent-2.1.8-stable.tar.gz)**

![img_1.png](source/_posts/img/0004/img_1.png)

**3) 解压配置编译**

**\# tar -zxvf libevent-2.1.8-stable.tar.gz**

**\# ./configure -prefix=/usr/local/libevent**

**#make && make install**

**完成界面**

![img_2.png](source/_posts/img/0004/img_2.png)

**2 、安装memcache**

**1）、**下载参考 memcache   地址：http://memcached.org/    （略）****

**2）解压配置安装**

**\[root@iZ94kr6cqbrZ local\]#  tar -zxvf memcached-1.5.3.tar.gz**

**\[root@iZ94kr6cqbrZ local\]# cd memcached-1.5.3**

**\[root@iZ94kr6cqbrZ memcached-1.5.3\]# ./configure --prefix=/usr/local/memcache/ --with-libevent=/usr/local/libevent**

**\[root@iZ94kr6cqbrZ memcached-1.5.3\]# make && make install**

![img_3.png](source/_posts/img/0004/img_3.png)

**启用测试连接**

**\[root@iZ94kr6cqbrZ /\]# /usr/local/memcache/bin/memcached -d -l 120.24.99.18 -p 11211 -m 2048 -u root**

**\[root@iZ94kr6cqbrZ /\]# telnet 120.24.99.18 11211**

![img_4.png](source/_posts/img/0004/img_4.png)
