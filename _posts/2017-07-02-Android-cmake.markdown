---
layout:     post
title:      "Cmake--让JNI从入门不再放弃"
subtitle:   ""
date:       2016-07-02
author:     "李洋彪"
header-img: "img/post-bg-js-module.jpg"
tags:
    - Android Cmake JNI
---

>使用Cmake构建jni项目，简直简单到不要不要的。

## 编译so库

### 从创建项目开始

![](https://i.imgur.com/ZRp3MM8.png)

记得将C++ suport 勾选上。

然后在Customize C++ Support界面默认即可。

创建项目后直接运行就能看到结果，

### 从老项目开始

在module的gradle中添加如下：

	android {
		externalNativeBuild {
	        cmake {
	            path 'CMakeLists.txt'
	        }
    	}
	}

其中**CMakeLists.txt**文件位于module根目录下。（复制过去就好啦）

### CMakeLists.txt介绍
首先看一下CMakeLists.txt文件的内容：

	# Sets the minimum version of CMake required to build the native
	# library. You should either keep the default value or only pass a
	# value of 3.4.0 or lower.
	
	cmake_minimum_required(VERSION 3.4.1)
	
	# Creates and names a library, sets it as either STATIC
	# or SHARED, and provides the relative paths to its source code.
	# You can define multiple libraries, and CMake builds it for you.
	# Gradle automatically packages shared libraries with your APK.
	
	add_library( # Sets the name of the library.
	             YOUR-Library_name
	
	             # Sets the library as a shared library.
	             SHARED
	
	             # Provides a relative path to your source file(s).
	             # Associated headers in the same location as their source
	             # file are automatically included.
	             src/main/cpp/native-lib )
	
	# Searches for a specified prebuilt library and stores the path as a
	# variable. Because system libraries are included in the search path by
	# default, you only need to specify the name of the public NDK library
	# you want to add. CMake verifies that the library exists before
	# completing its build.
	
	find_library( # Sets the name of the path variable.
	              log-lib
	
	              # Specifies the name of the NDK library that
	              # you want CMake to locate.
	              log )
	
	# Specifies libraries CMake should link to your target library. You
	# can link multiple libraries, such as libraries you define in the
	# build script, prebuilt third-party libraries, or system libraries.
	
	target_link_libraries( # Specifies the target library.
	                       YOUR-Library_name
	
	                       # Links the target library to the log library
	                       # included in the NDK.
	                       ${log-lib} )

什么意思呢，看下面的介绍：

#### 1、 CMake最小版本使用的是3.4.1。

#### 2、 add_library,配置cmake

**YOUR-Library_name**是引用so库的名称，在项目中，如果需要使用这个so文件，引用的名称就是这个。实际上生成的so文件名称是libYOUR-Library_name。当Run项目或者build项目是，在Module级别的build文件下的intermediates\cmake\debug\debug\obj下会生成相应的so库文件。

SHARED 表示共享so库文件，也就是在Run项目或者build项目时会在目录intermediates\cmake\debug\debug\obj下会生成相应的so库文件。

src/main/cpp/native-lib.cpp 构建so库的源文件。

>STATIC：静态库，是目标文件的归档文件，在链接其它目标的时候使用。
SHARED：动态库，会被动态链接，在运行时被加载。
MODULE：模块库，是不会被链接到其它目标中的插件，但是可能会在运行时使用dlopen-系列的函数动态链接。
更详细的解释请参考这篇文章：C++静态库与动态库

### 3、 find_library

log-lib 这个指定的是在NDK库中每个类型的库会存放一个特定的位置，而log库存放在log-lib中

log 指定使用log库

### 4、target_link_libraries()
>如果你本地的库（native-lib）想要调用log库的方法，那么就需要配置这个属性，意思是把NDK库关联到本地库。

* native-lib
要被关联的库名称

* ${log-lib}
要关联的库名称，要用大括号包裹，前面还要有$符号去引用。

**实际上，真正用到的地方是：YOUR-Library_name，src/main/cpp/native-lib.cpp**

### 5、native-lib 代码

	#include <jni.h>
	#include <string>

	extern "C"
	JNIEXPORT jstring JNICALL
	Java_top_fighter_1lee_jnitest_JniUtils_stringFromJNI(
	        JNIEnv *env,
	        jobject /* this */) {
	    std::string hello = "Hello from C++";
	    return env->NewStringUTF(hello.c_str());
	}

注意Java_top_fighter_1lee_jnitest_JniUtils_stringFromJNI，从包名到到方法，不能有差异。

### 6、JniUtils代码

	
	public class JniUtils {

	    static {
	        System.loadLibrary("native-lib");
	    }
	
	    public static native String stringFromJNI();

	}

## 使用so库
>上述操作在编译后在intermediates\cmake\debug\debug\obj下会生成相应的so库文件。复制到jinLibs下，项目结构如下图：

![](https://i.imgur.com/A2OMUHj.png)

可以将cmake都注释掉，然后运行，发现和之前是一样的结果也，如果你不信，可以将cpp文件删掉后再运行。

