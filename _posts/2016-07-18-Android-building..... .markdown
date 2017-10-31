---
layout:     post
title:      "AndroidStudio building时间过长"
subtitle:   "完美解决as秒加载新项目"
date:       2016-07-18
author:     "李洋彪"
header-img: "img/post-bg-js-module.jpg"
tags:
    - Android
    - AndroidStudio
---

### 问题描述
每一个用过as的Android攻城狮都会有这样的问题，在使用别人的项目或者新建项目时一直卡在building阶段，就如下图：
![](https://i.imgur.com/Bt9GEiy.png)

### 问题分析
为什么as会卡在这个地方呢，因为他在下载缺少的Gradle版本，大概几十M，没有梯子的话下载速度特别慢。

### 问题解决
首先我们要知道as正在下载的是哪个版本，先将as强行停止掉，去需要构建的项目目录下，找到gradle-wrapper.properties文件，用文本打开：

![](https://i.imgur.com/VQCsder.png)

文件内容如下,即下载的gradle版本是gradle-2.14.1-all

	#Mon Dec 28 10:00:20 PST 2015
	distributionBase=GRADLE_USER_HOME
	distributionPath=wrapper/dists
	zipStoreBase=GRADLE_USER_HOME
	zipStorePath=wrapper/dists
	distributionUrl=https\://services.gradle.org/distributions/gradle-2.14.1-all.zip

然后点击如下链接中去找对应的gradle版本：

[点击](http://services.gradle.org/distributions/) http://services.gradle.org/distributions/

找到自己需要下载的gradle版本，我的是2.14，然后鼠标右键，复制下载链接，用迅雷下载，速度杠杠的。

![](https://i.imgur.com/QgBdCfL.png)


下载后，复制到自己用户目录下的如下文件夹中（C:\Users\fighter_lee\.gradle\wrapper\dists），复制到对应版本的文件夹下，重新打开as，会发现以下就编译好了。

![](https://i.imgur.com/aK9OwF2.png)

