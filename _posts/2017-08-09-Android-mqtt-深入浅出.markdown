---
layout:     post
title:      "mqtt深入浅出"
subtitle:   ""
date:       2017-08-15
author:     "李洋彪"
header-img: "img/post-bg-js-module.jpg"
tags:
    - Android
---

如何使用请参考此篇博客，写的通俗易懂。

[Android APP必备高级功能，消息推送之MQTT](http://blog.csdn.net/qq_17250009/article/details/52774472)

## 源码分析

### 获取源码文件

github上是没有mqtt源码的，虽然我们能从远程依赖上拿到jar包，但是因为java文件编译后成class文件后注释、常量、以及部分方法都发生了一些变化，可读性很差，那我们如何拿源码呢，当然是jcenter了，如图：

在jcenter官网上搜索关键字：eclipse.paho.client.mqttv3

点进去后，找一下以下文件，如图的那个就是java文件（下载下来后改成zip格式，然后解压就好啦）

![](https://i.imgur.com/UZIdDrm.png)

### connect

