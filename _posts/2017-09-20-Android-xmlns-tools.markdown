---
layout:     post
title:      "xmlns:tools全解析"
subtitle:   "布局文件中的xmlns:tools作用以及用法"
date:       2017-06-20
author:     "李洋彪"
header-img: "img/post-bg-js-module.jpg"
tags:
    - Android
---

写布局的时候经常出现如下：

	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.example.abc.progressbar_lu.MainActivity"

总是多了tools这栏，然后也没有在意，都是直接删掉，囧~

某天心血来潮，好奇地特地查了，看看这家伙什么作用，才有点发现新大陆的感觉。

官方文档有这么一句话：

These are attributes which are used when the layout is rendered inthe tool, but have no impact on the runtime. This is useful if you for examplewant to put sample data in your textfields for when you are editing the layout,but you don't want those attributes to affect your running app.

也就是说，开发人员在设计Android Layout布局时，总会伴随着一些乱七八槽的困扰。比如，为了更加逼真的真实数据预览效果，我们在开发时会将TextView的text属性写上一些假数据，而当运行到模拟器或真机上时这些假数据就成了造成体验上甚至测试BUG的脏数据，又需要一一清除。再比如，我们想在XML的预览界面上就看到ListView的Item内容，而不是只有通过编译运行时才能查看。都可以通过Tools Attributes得以解决。

###作用
这里将Tools Attributes按照功能分为两种类别，一种是去除Lint提示的，一种是展示布局预览的，下面一一介绍相关属性的使用。

###Lint 提示
这个属性用于告诉Android Lint忽视某些xml警告，基本上用的少，就不提了(又不是error，关我什么事~)。

###布局预览

####替换标准的android命名空间的控件固有属性

举个栗子：

	<ImageView
        android:id="@+id/meizi_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:adjustViewBounds="true"
        android:scaleType="fitXY"
        tools:src="@mipmap/ic_launcher"/>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        tools:text="lalalala"
        />
这样达到窗口预览时可以看到TextView的text内容和imageview的src内容，而运行时则会被忽略。

####tools:context
这个属性用在layout文件的根元素上，指明与当前layout相关联的Activity，从而在预览时使用Activity的主题（theme一般定义在Manifest文件中并且与activities联系在一起，而非layout）。可以使用Activity的全名，也可以利用manifest中定义的包名作为前缀：

举个栗子：

	<android.support.design.widget.CoordinatorLayout
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:app="http://schemas.android.com/apk/res-auto"
	    xmlns:tools="http://schemas.android.com/tools"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    tools:context="com.fighter.superframe.ui.activity.MainActivity">

####tools:layout
这个属性主要用于标签中，指定预览时用的layout布局文件内容：

举个栗子：

	<fragment
		android:layout_width="match_parent"
		android:layout_height="match_parent"
		tools:layout="@layout/fragment_content"/>

####tools:listitem
RecyclerView预览item效果

	<android.support.v7.widget.RecyclerView
            android:id="@+id/list"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            tools:listitem="@layout/item_meizhi"
            android:paddingLeft="2.5dp"
            android:paddingRight="2.5dp"/>


####tools:showIn
这个属性用在标签所包含的layout的根元素上，指定一个所在的布局文件，这样在被嵌套的layout的design视图中便可以预览到外层的layout内容。比如在activity_main.xml中使用<include>标签嵌套一个include_content.xml文件，那么在include_content.xml文件的根元素中就可以使用tools:showIn属性指定Outer layout，达到在include_content.xml文件中预览activity_main.xml内容的效果：

比如：

	<android.support.v7.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
                                    xmlns:app="http://schemas.android.com/apk/res-auto"
                                    xmlns:tools="http://schemas.android.com/tools"
                                    android:id="@+id/meizhi_card"
                                    android:layout_width="match_parent"
                                    android:layout_height="wrap_content"
                                    tools:showIn="@layout/fragment_meizi"
                                    android:foreground="?android:attr/selectableItemBackground"
                                    app:cardCornerRadius="8dp"
                                    app:cardElevation="8dp">

这样就能在recycleview的item中预览recycleview整体的效果了。

####tools:menu
这个属性用在layout根元素中，指定layout预览时ActionBar展示的menu内容。使用tools:context属性时，ActionBar会自动查找Activity中的onCreateOptionsMenu()方法预览menu，tools:menu属性的设置会覆盖tools:context属性的效果。tools:menu的值只需要使用menu文件的名字即可，多个menu使用逗号隔开，如

	<?xml version="1.0" encoding="utf-8"?>
		<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:tools="http://schemas.android.com/tools"
	    tools:menu="menu1,menu2"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent">
	</RelativeLayout>

参考：
[官方文档](http://tools.android.com/tech-docs/tools-attributes)