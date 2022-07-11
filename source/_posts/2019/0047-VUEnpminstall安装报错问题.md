---
title: VUE npm install 安装报错问题 （记录）
index_img: /img/cover/17.jpg
categories:
  - - 前端
  - - VUE
tags:
  - VUE
abbrlink: 26c3d5f8
date: 2019-04-01 23:55:15
---

**错误信息：**
```text
npm ERR!
npm ERR! If you're sure you want to delete the entire cache, rerun this command with --force.

npm ERR! A complete log of this run can be found in:
npm ERR!     C:\Users\newb2\AppData\Roaming\npm-cache\_logs\2019-03-28T15_07_13_255Z-debug.log
```

使用如下命令清除npm编译缓存即可

npm cache clean --froce

![](1.png)

然后重新执行安装依赖命令即可
npm install

安装过程中又报如下错误：
```text
npm ERR! the command again as root/Administrator (though this is not recommended)
```

切换到 root权限账号 执行install命令即可

还有一种情况就是网络的原因，一直下载失败，需要安装淘宝镜像

```text
>npm install -g cnpm --registry=https://registry.npm.taobao.org
# 安装完成后 
>cnpm install
```
![](2.png)

执行成功结果：

![](3.png)
