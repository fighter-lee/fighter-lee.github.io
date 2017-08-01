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


通过命令行执行adb shell am broadcast发送广播通知。

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


说明：蓝色为key，红色为alue，分别为String类型，int类型，boolean类型