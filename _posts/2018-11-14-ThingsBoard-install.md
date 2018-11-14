---
layout:     post
title:      "ThingsBoard部署安装"
subtitle:   "大半夜写博客也是醉了，但是有位大佬说要写博客，真的很有用"
date:       2018-11-14 23:38:21
author:     "felix"
header-img: "img/in-post/thingsboard/head.jpg"
catalog:    true
tags:
    - ThingsBoard
    - IOT
    - SpringBoot
    - MQTT
---
# 下载

## 准备环境

官方文档已经很详细了，有Linux和Windows各种版本的。

需要准备**Java环境**，**数据库环境**

注意Jdk版本需要高于1.8

# 安装

我现在最常用的就是这几个命令了，由于在机器上总是更新代码于是乎一遍遍的重置，下面列出了在Centos系统上的安装命令：

> ps：为什么用dos2unix 因为我源码编译是Windows机器，打包之后配置文件都是dos格式的，会有各种离奇的错误。。。。详见后面的博客

## ThingsBoard


```
rpm -ivh thingsboard.rpm

find /usr/share/thingsboard/ | xargs dos2unix
find /etc/thingsboard/ | xargs dos2unix
```

## Tb-Gateway

```
rpm -ivh tb-gateway.rpm

find /usr/share/tb-gateway/ | xargs dos2unix
find /etc/tb-gateway/ | xargs dos2unix
```

# 启动

## ThingsBoard

```
service thingsboard start
```

## Tb-Gateway

```
service tb-gateway start
```

# 卸载

卸载前请备份相应配置文件。配置文件路径为

```
cp /etc/thingsboard/conf/thingsboard.yml ~
cp /etc/tb-gateway/conf/tb-gateway.yml ~
```

## ThingsBoard

```
service thingsboard stop
rpm -e --nodeps thingsboard
rm -rf /usr/share/thingsboard/
rm -rf /var/log/thingsboard/
rm -rf /etc/thingsboard/
```

## Tb-Gateway

```
service tb-gateway stop
rpm -e --nodeps tb-gateway
rm -rf /usr/share/tb-gateway/
rm -rf /var/log/tb-gateway/
rm -rf /etc/tb-gateway/
```

---

> 这段时间准备把tb和gw的一些心得记下来。其实已经有很多东西了，只是没时间发博客啦。抽时间整理一下放到这里。