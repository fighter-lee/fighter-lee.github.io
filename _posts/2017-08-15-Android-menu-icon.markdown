---
layout:     post
title:      "NavigationView 侧滑栏menu实现右侧自定义布局"
subtitle:   ""
date:       2017-08-15
author:     "李洋彪"
header-img: "img/post-bg-js-module.jpg"
tags:
    - Android
---

效果图：
![](http://i.imgur.com/1U5HI1w.png)

布局：
	
	<android.support.v4.widget.DrawerLayout
        android:id="@+id/drawer_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:openDrawer="start">

        <include
            layout="@layout/drawer_content_layout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"/>

        <android.support.design.widget.NavigationView
            android:id="@+id/nav_view"
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:layout_gravity="start"
            android:fitsSystemWindows="true"
            app:headerLayout="@layout/drawer_menu_layout"
            app:menu="@menu/activity_main_drawer"/>

    </android.support.v4.widget.DrawerLayout>

menu（需要在item下添加相对应的layout）：

	<?xml version="1.0" encoding="utf-8"?>
	<menu xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto">

    <group android:checkableBehavior="single">

        <item
            android:id="@+id/deleteFile"
            android:icon="@mipmap/round"
            android:title="@string/delete_package"/>
        <item
            android:id="@+id/local_upgrade"
            android:icon="@mipmap/round"
            android:title="@string/local_update"/>
        <item
            android:id="@+id/ota_login_logout"
            android:icon="@mipmap/round"
            android:title="@string/login"
            app:actionLayout="@layout/online_layout"/>

    </group>

</menu>

获取menu及menu对应的布局

	Menu menu = navView.getMenu();
    menu_login_logout = menu.findItem(R.id.ota_login_logout);
    LinearLayout ll_action_view = (LinearLayout) menu_login_logout.getActionView();
    mIv_online = (ImageView) ll_action_view.findViewById(R.id.iv_online);