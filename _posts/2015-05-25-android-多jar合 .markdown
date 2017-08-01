---
layout:     post
title:      "多jar包合并"
subtitle:   ""
date:       2015-05-25
author:     "李洋彪"
header-img: "img/post-bg-js-module.jpg"
tags:
    - Android
---


    D:
    
    cd document/jar1
    
    jar -xvf http_libs.jar
    
    jar -xvf iot_download_libs.jar
    
    jar -xvf iot_libs.jar
    
    jar -xvf mqtt_libs.jar
    
    jar -xvf trace.jar
    
    del /F *.jar
    
    jar –cvfM iport.jar .


jar -xvf xx.jar 为将jar拆分成class文件
jar –cvfM iport.jar . 为将class文件合并  记得别忘记jar后面的.