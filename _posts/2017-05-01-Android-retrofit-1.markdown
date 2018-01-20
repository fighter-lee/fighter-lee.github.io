---
layout:     post
title:      "Retrofit2+Rxjava2七日谈"
subtitle:   "第一日：Retrofit 初来乍到"
date:       2017-05-1 12:00:00
author:     "李洋彪"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Android
---

###1、Retrofit2入门
>Retrofit 其实相当简单，简单到源码只有30多个文件，其中20多个文件是注解还都和HTTP有关，真正暴露给用户的类并不多,所以我看了一遍 官方教程 大多数情景就可以无障碍使用，如果你还没有看过，可以先去看看，附上连接：[retrofit](http://square.github.io/retrofit/)

会用是一方面，但是首先我们得知道为什么要这么用，先一起学学http的基础知识。

####1. HTTP请求报文格式
HTTP 的请求报文分为三个部分 请求行、请求头和请求体，格式如图：
![](http://i.imgur.com/oXbwuLo.png)
直接上实例图：
![](http://i.imgur.com/ziOkCEa.png)