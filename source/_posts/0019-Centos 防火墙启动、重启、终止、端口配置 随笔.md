---
title: Centos 防火墙启动、重启、终止、端口配置--随笔
index_img: /img/cover/19.jpg
categories:
  - Linux
tags:
  - 防火墙
abbrlink: bfd95a20
date: 2018-06-27 15:13:30
---
```
# firewall-cmd --zone=public --add-port=8080/tcp --permanent        #添加8080端口

# firewall-cmd --reload                      #重新加载策略配置，以使新配置生效
```
注：禁止firewall开机启动为：systemctldisable firewalld.service

运行、停止、禁用firewalld

    启动：# systemctl start  firewalld

    查看状态：# systemctl status firewalld 或者 firewall-cmd --state

    停止：# systemctl disable firewalld

    禁用：# systemctl stop firewalld

关闭防火墙
    
    systemctl stop firewalld.service             #停止firewall
    systemctl disable firewalld.service        #禁止firewall开机启动

开启端口
    
    firewall-cmd --zone=public --add-port=80/tcp --permanent

命令含义

    --zone #作用域
    --add-port=80/tcp #添加端口，格式为：端口/通讯协议
    --permanent #永久生效，没有此参数重启后失效

重启防火墙

    firewall-cmd --reload

常用命令介绍

    firewall-cmd --state                           ##查看防火墙状态，是否是running
    firewall-cmd --reload                          ##重新载入配置，比如添加规则之后，需要执行此命令
    firewall-cmd --get-zones                       ##列出支持的zone
    firewall-cmd --get-services                    ##列出支持的服务，在列表中的服务是放行的
    firewall-cmd --query-service ftp               ##查看ftp服务是否支持，返回yes或者no
    firewall-cmd --add-service=ftp                 ##临时开放ftp服务
    firewall-cmd --add-service=ftp --permanent     ##永久开放ftp服务
    firewall-cmd --remove-service=ftp --permanent  ##永久移除ftp服务
    firewall-cmd --add-port=80/tcp --permanent     ##永久添加80端口
    iptables -L -n                                 ##查看规则，这个命令是和iptables的相同的
    man firewall-cmd                               ##查看帮助

更多命令，使用  firewall-cmd --help 查看帮助文件

CentOS 7.0默认使用的是firewall作为防火墙，使用iptables必须重新设置一下

直接关闭防火墙

    systemctl stop firewalld.service           #停止firewall
    systemctl disable firewalld.service     #禁止firewall开机启动