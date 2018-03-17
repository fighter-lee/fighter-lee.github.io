---
layout:     keynote
title:      "ADB操作"
subtitle:   "日常记录"
date:       2015-05-10
author:     "李洋彪"
header-img: "img/post-bg-js-version.jpg"
tags:
    - Android
    - Java
---


一个非常全的总结：[click](https://github.com/mzlogin/awesome-adb)

链接如下：

https://github.com/mzlogin/awesome-adb

**通过命令行执行adb shell am broadcast发送广播通知。**

adb shell am broadcast 后面的参数有：

[-a <ACTION>]
[-d <DATA_URI>]
[-t <MIME_TYPE>] 
[-c <CATEGORY> [-c <CATEGORY>] ...] 
[-e|--es <EXTRA_KEY> <EXTRA_STRING_VALUE> ...] 
[--ez <EXTRA_KEY> <EXTRA_BOOLEAN_VALUE> ...] 
[-e|--ei <EXTRA_KEY> <EXTRA_INT_VALUE> ...] 
[-n <COMPONENT>]
[-f <FLAGS>] [<URI>]



例如：

adb shell am broadcast -a com.Android.test --es test_string "this is test string" --ei test_int 100 --ez test_boolean true

--es 前面为双横线

说明：蓝色为key，红色为alue，分别为String类型，int类型，boolean类型

adb 启动应用：
adb shell am start -n 包名/.mainactivity

adb 停止应用
adb shell am force-stop 包名

**启动activity**

adb shell am start -n 包名/启动activity路径

例如：

adb shell am start -n com.fighter-lee.mydemo/com.fighter-lee.mydemo.MainActivity

**启动service**

adb shell am startservice -n com.adups.fota_iot/com.adups.fota_demo_iot.service.OTAEngine

**adb修改时间**

adb shell date -s "20171010.120000"
