---
layout:     post
title:      "Android aidl 系列"
subtitle:   "深入浅出（二）--回调"
date:       2015-05-30
author:     "李洋彪"
header-img: "img/post-bg-js-module.jpg"
tags:
    - Android
---

上一节讲了基本使用，但是如果有这么一个需求：本应用控制另一个应用下载吗，并且需要知道进度，所以还需要使用到回调。

先看看项目结构：

![](https://i.imgur.com/6r0WhOt.png)


IService的代码：

	package com.example.aidlservice;

	import com.example.aidlservice.IServiceListener;
	
	interface IService {
	   void test();// 测试直接调用aidl方法
	   void start(IServiceListener iServiceListener);// 方法里面携带监听回调对象
	}

IServiceListener代码：

	package com.example.aidlservice;
	
	interface IServiceListener {
	    void download(int progress);
	    void cancel();
	}

需要注意包名是否正确。

### client控制server端下载并回调进度：

1.绑定service，同上一节：

	private ServiceConnection conn = new ServiceConnection() {

        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            iService = IService.Stub.asInterface(service);
            Log.d(TAG, "onServiceConnected: ");
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            iService = null;
        }
    };

2.开始控制下载：

	iService.start(new IServiceListener.Stub() {
                        @Override
                        public void download(int progress) throws RemoteException {
                            Log.d(TAG, "download: "+progress);
                        }

                        @Override
                        public void cancel() throws RemoteException {
                            Log.d(TAG, "cancel: ");
                        }
                    });

需要注意：回调函数如果让as自动生成是这样的，多了个asBinder，**需要去掉**，不然会出错：

	iService.start(new IServiceListener() {
                        @Override
                        public void download(int progress) throws RemoteException {
                            
                        }

                        @Override
                        public void cancel() throws RemoteException {

                        }

                        @Override
                        public IBinder asBinder() {
                            return null;
                        }
                    });

### server端下载

下面模拟回调：

	private void startCallback(IServiceListener iServiceListener) {
        for (int i = 0; i < 100; i++) {
            SystemClock.sleep(100);
            try {
                iServiceListener.download(i);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        try {
            iServiceListener.cancel();
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }

搞定~