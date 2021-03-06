---
layout:     post
title:      "AndroidStudio构建变体指南"
subtitle:   ""
date:       2018-02-04
author:     "李洋彪"
header-img: "img/post-bg-love1.jpg"
tags:
    - Android
---

## 了解
要熟练构建变体，先需要了解Androidstudio是如何来自动化执行和管理构建流程。

### 构建流程
![](https://i.imgur.com/rpJMlyP.png)

如图所示，典型 Android 应用模块的构建流程通常依循下列步骤：

1. 编译器将您的源代码转换成 DEX（Dalvik Executable) 文件（其中包括运行在 Android 设备上的字节码），将所有其他内容转换成已编译资源。
2. APK 打包器将 DEX 文件和已编译资源合并成单个 APK。不过，必须先签署 APK，才能将应用安装并部署到 Android 设备上。
3. APK 打包器使用调试或发布密钥库签署您的 APK：
	1. 如果您构建的是调试版本的应用（即专用于测试和分析的应用），打包器会使用调试密钥库签署您的应用。Android Studio 自动使用调试密钥库配置新项目。
	2. 如果您构建的是打算向外发布的发布版本应用，打包器会使用发布密钥库签署您的应用。要创建发布密钥库，请阅读在 Android Studio 中签署您的应用。
4. 在生成最终 APK 之前，打包器会使用 zipalign 工具对应用进行优化，减少其在设备上运行时的内存占用。

构建流程结束时，您将获得可用来进行部署、测试的调试 APK，或者可用来发布给外部用户的发布 APK。

### 自定义构建配置
通过Androidstudio我们可以自动构建哪些内容呢？

* 构建类型
* 产品风味
* 构建变体
* 清单条目
* 依赖项

下面详细介绍如何去构建。

## 配置构建类型(buildTypes)
可以在模块级 build.gradle 文件的 android {} 代码块内部创建和配置构建类型，默认的类型为release和debug。

我们可以在realease项中很方便的对APK进行签名，也可以对applicationId根据类型添加后缀，例如：
	
	//读取签名文件信息
	def keystorePropertiesFile = rootProject.file("keystore.properties");
	def keystoreProperties = new Properties()
	keystoreProperties.load(new FileInputStream(keystorePropertiesFile))

	android {
	    compileSdkVersion rootProject.ext.compileSdkVersion
	    buildToolsVersion rootProject.ext.buildToolsVersion
	    defaultConfig {
	        applicationId "com.abupdate.fota_demo_iot"
	        minSdkVersion rootProject.ext.minSdkVersion
	        targetSdkVersion rootProject.ext.targetSdkVersion
	        versionCode 1
	        versionName verName
	    }
	
		//签名配置信息
	    signingConfigs {
	        config {
	            storeFile file(keystoreProperties['ReleaseStoreFile'])
	            storePassword keystoreProperties['ReleaseStorePassword']
	            keyAlias keystoreProperties['ReleaseKeyAlias']
	            keyPassword keystoreProperties['ReleaseKeyPassword']
	        }
	    }
	
	    buildTypes {
	        release {
	            minifyEnabled true
	            shrinkResources true
	            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
				//签名
	            signingConfig signingConfigs.config
	        }
	        debug {
	            minifyEnabled false
	            shrinkResources false
				//对applicationId根据类型添加后缀
				applicationIdSuffix ".debug"
	        }
	    }
	}

修改后记得点一下工具栏中的同步。

## 配置产品风味(productFlavors)
创建productFlavor与创建构建类型类似：只需将它们添加到 productFlavors {} 代码块并配置您想要的设置。productFlavor支持与 defaultConfig 相同的属性，这是因为 defaultConfig 实际上属于 ProductFlavor 类。这意味着，您可以在 defaultConfig {} 代码块中提供所有productFlavor的基本配置，每种productFlavor均可更改任何这些默认值，例如 applicationId。

比如，现在我们有两种产品风味，一种为车机产品，一种为手机产品，那么可以这么配置：

	productFlavors{
        Car{
			//根据不同的产品提添加不同的字段，以便在代码中进行判断
            buildConfigField("String", "APP_LAUNCH_ACTIVITY", APP_LAUNCH_ACTIVITY_CAR)
            buildConfigField("String", "APP_TYPE", APP_TYPE_CAR)
            applicationIdSuffix ".car"
            versionNameSuffix "-car
        }
        Phone{
            buildConfigField("String", "APP_LAUNCH_ACTIVITY", APP_LAUNCH_ACTIVITY_IOT)
            buildConfigField("String", "APP_TYPE", APP_TYPE_IOT)
            applicationIdSuffix ".phone"
            versionNameSuffix "-phone
        }
	}

## 组合多个产品风味(flavorDimensions)
某些情况下，您可能希望组合多个产品风味中的配置。

你应该经常有这种需求，要构建有启动图标和没有启动图标的两个APP，那么这个productFlavor和上文提到的productFlavors(car和phone)是两个维度的东西，那么我们创建两个维度的productFlavors，示例如下：

	//两个维度：设备类型和图标类型
	flavorDimensions "device_type", "icon_type"
    productFlavors{
        Car{
			//指定该productFlavor是哪个维度
			dimension  'device_type'
			//根据不同的产品提添加不同的字段，以便在代码中进行判断
            buildConfigField("String", "APP_LAUNCH_ACTIVITY", APP_LAUNCH_ACTIVITY_CAR)
            buildConfigField("String", "APP_TYPE", APP_TYPE_CAR)
            applicationIdSuffix ".car"
            versionNameSuffix "-car
        }
        Phone{
			dimension  'device_type'
            buildConfigField("String", "APP_LAUNCH_ACTIVITY", APP_LAUNCH_ACTIVITY_IOT)
            buildConfigField("String", "APP_TYPE", APP_TYPE_Phone)
            applicationIdSuffix ".phone"
            versionNameSuffix "-phone
        }

        with_icon{
            dimension  'icon_type'
        }

        no_icon{
            dimension  'icon_type'
        }
    }

## 选择变体
完成以上步骤后，点击Androidstudio左侧的Build Variant,可以看到如图所示的内容：

![](https://i.imgur.com/4x0WiBu.png)

## 根据变体打包成不同的APP
根据上文选择的变体，点击工具栏中的Build APK可以生成对应的APP。

	android.applicationVariants.all { variant ->
        variant.outputs.all {
            outputFileName = "${variant.name}-${variant.versionName}.apk"

        }
    }

buildAPK后可以生成如图的APP：

![](https://i.imgur.com/aSq2pIX.png)

## 清单文件合并(AndroidManifest)
各个变体肯定需要差异化，不同的APPName、icon、或者是业务逻辑，要做到这些，需要先学习如何合并AndroidManifest。

首先建立对应的源集，默认的方式如图：

![](https://i.imgur.com/4YELSS4.png)

然后建立对应的AndroidManifest文件

![](https://i.imgur.com/veM7Y0g.png)

### 合并优先级
可能你没有合并过AndroidManifest，但是肯定有这样的体验，以前我们使用jar包时需要在AndroidManifest中添加jar包中的组件和权限，但是在使用model或者远程依赖或者aar时就不需要添加，其实编译时就有合并的操作。

合并 3 个清单 文件（从优先级最低的文件（左）合并至优先级最高的文件（右））的流程：
![](https://i.imgur.com/lesmpAH.png)

那我们的变体是如何去合并的呢？

优先级如下（由高到低）（其实是定制程度高低决定）：

1. 构建变体清单（如 src/demoDebug/）
2. 构建类型清单（如 src/debug/）
3. 产品定制清单（如 src/demo/）

### 合并冲突解决
内容较多，可以参考官方指南：https://developer.android.com/studio/build/manifest-merge.html#_3

官方文档介绍比较清楚了，这里不再重述。

## 构建差异

### noIcon变种去掉启动图标

在对应noIcon的变种集下，AndroidManifest代码如下：

	<activity
            android:name=".view.activity.CarActivity"
            android:configChanges="keyboard|screenSize|orientation"
            android:launchMode="singleTask"
            android:theme="@style/AppTheme.NoActionBar"
            tools:node="merge">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>

                <category
                    android:name="android.intent.category.LAUNCHER"
                    tools:node="remove"/>
            </intent-filter>
        </activity>

即采用上文提到的 **合并冲突解决** 中的方式将启动LAUNCHER去掉。

### 不同变种启动的activity不同
可以将main中的启动activity去掉，在变体源集中将对应的activity添加进来，那么，按照合并优先级，变体源集中的activity会被合并进来。

### 不同的变体代码逻辑不同
可以看到上文 **组合多个产品风味** 中，有在产品风味中增加buildConfigField("String", "APP_TYPE", APP_TYPE_CAR)代码。

那么我们可以在代码中拿到这个标识，进行逻辑区分。

	BuildConfig.APP_TYPE;

### 不同的代码和资源文件
在对应的源集下进行编码和存放资源文件即可，但是注意不要类路径和类名不要一样，不然会有冲突。

![](https://i.imgur.com/hOepTKS.png)