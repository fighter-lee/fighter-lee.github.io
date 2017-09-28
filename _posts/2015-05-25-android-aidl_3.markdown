---
layout:     post
title:      "Android aidl 系列"
subtitle:   "深入浅出（三）--自定义Parcelable"
date:       2015-06-2
author:     "李洋彪"
header-img: "img/post-bg-js-module.jpg"
tags:
    - Android
---

如果现在需要传递下载url，文件大小，md5值等数据时，用基本数据类型去传递的话会很麻烦，那么我们用基本数据类型去传递会很容易实现。

现在看一下项目结构：（<font color = red>请注意Info类的位置，包名要和Info.aidl文件包名一致</font>）

![](https://i.imgur.com/MxWPQi7.png)

![](https://i.imgur.com/1nDjojO.png)

### IService.aidl：

注意void start()参数中的in

	package com.example.aidlservice;

	import com.example.aidlservice.IServiceListener;
	import com.example.aidlservice.Info;
	
	interface IService {

	   void test();// 测试直接调用aidl方法
	   void start(in Info info,IServiceListener iServiceListener);// 方法里面携带监听回调对象
	}

### Info.aidl

	package com.example.aidlservice;

	parcelable Info;

### Info.java

	public class Info implements Parcelable {
	
	    public static final Creator<Info> CREATOR = new Creator<Info>() {
	        @Override
	        public Info createFromParcel(Parcel in) {
	            return new Info(in);
	        }
	
	        @Override
	        public Info[] newArray(int size) {
	            return new Info[size];
	        }
	    };
	
	    public String getUrl() {
	        return url;
	    }
	
	    public void setUrl(String url) {
	        this.url = url;
	    }
	
	    public long getFileSize() {
	        return fileSize;
	    }
	
	    public void setFileSize(long fileSize) {
	        this.fileSize = fileSize;
	    }
	
	    private String url;
	    private long fileSize;
	
	    public Info() {
	
	    }
	
	    protected Info(Parcel in) {
	        url = in.readString();
	        fileSize = in.readLong();
	    }
	
	    @Override
	    public int describeContents() {
	        return 0;
	    }
	
	    @Override
	    public void writeToParcel(Parcel dest, int flags) {
	        dest.writeString(url);
	        dest.writeLong(fileSize);
	    }
	}

其他用法同上两篇。