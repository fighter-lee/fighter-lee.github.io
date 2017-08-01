---
layout:     post
title:      "Android上传Library到Jcenter的坑与解"
subtitle:   ""
date:       2016-05-22
author:     "李洋彪"
header-img: "img/post-bg-js-version.jpg"
tags:
    - Android
---


参考鸿神的博客
[http://blog.csdn.net/lmj623565791/article/details/51148825](http://blog.csdn.net/lmj623565791/article/details/51148825)； 
>鸿神的博客实在太简略了，很多细节多没有说到,我也踩了很多的坑，所以既然你点进来了，那么还是跟着我的节奏来。

![6af89bc8gw1f8qksbvy3wj208c08c74e.jpg](http://upload-images.jianshu.io/upload_images/4126773-1df8371eece935dc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###1.当然是注册
个人免费版注册地址：https://bintray.com/signup/oss
>之前在这就碰到一个坑，很多博客上说的注册地址是 https://bintray.com （企业版），然后我跟着做到最后一步，发现怎么都没有"Add to jcenter"按钮，各种谷歌，才找到问题。。。

###2.根据图解一步步添加
![Q~0]Z%4F5TD$1V4])`B9Q6E.png](http://upload-images.jianshu.io/upload_images/4126773-8b43f0c961d9299f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![DQX61PIX7XJ8{_080SJI14S.png](http://upload-images.jianshu.io/upload_images/4126773-4b96198e38fa76c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![)[6OKL_ELE1]}CT78LPRWXC.png](http://upload-images.jianshu.io/upload_images/4126773-66b90ce77444c51b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![)BLSEEBPP2V(YBOKV_FV11K.png](http://upload-images.jianshu.io/upload_images/4126773-e902191e7d5b8f65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![C9UK`AJY%)@U%WK]~2JMBPE.png](http://upload-images.jianshu.io/upload_images/4126773-a5f848ab8162846d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](UL0]A0H0%O.png](http://upload-images.jianshu.io/upload_images/4126773-3831cb8c79447456.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

做完以上操作后，那么可以上传代码了

###3.配置项目信息
配置module下build.gradle

    apply plugin: 'com.android.library'

    version "1.0.0"

    android {
        compileSdkVersion 23
        buildToolsVersion '25.0.0'

        defaultConfig {
            minSdkVersion 14
            targetSdkVersion 23
            versionCode 1
            versionName "${version}"

        }
        buildTypes {
            release {
                minifyEnabled true
                proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-        rules.pro'
        }
    }
    lintOptions {
        abortOnError false
    }

    }

    dependencies {
        provided 'com.android.support:appcompat-v7:25.0.0'
    }

    //需添加如下内容

    apply plugin: 'com.github.dcendents.android-maven'
    apply plugin: 'com.jfrog.bintray'


    //发布到组织名称名字，必须填写
    group = "com.fighter.maven"
    //发布到JCenter上的项目名字，必须填写
    def libName = "mylibs"
    // 版本号，下次更新是只需要更改版本号即可
    version = "1.0.0"
    /**  上面配置后上传至jcenter后的编译路径是这样的： compile       'com.fighter.maven:mylibs:1.0.0'  **/

    //生成源文件
    task sourcesJar(type: Jar) {
        from android.sourceSets.main.java.srcDirs
        classifier = 'sources'
    }
    //生成文档
    task javadoc(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
    options.encoding "UTF-8"
    options.charSet 'UTF-8'
    options.author true
    options.version true
    failOnError false
    }

    //文档打包成jar
    task javadocJar(type: Jar, dependsOn: javadoc) {
        classifier = 'javadoc'
       from javadoc.destinationDir
    }
    //拷贝javadoc文件
    task copyDoc(type: Copy) {
        from "${buildDir}/docs/"
        into "docs"
    }

    //上传到jcenter所需要的源码文件
    artifacts {
        archives javadocJar
        archives sourcesJar
    }

    // 配置maven库，生成POM.xml文件
    install {
        repositories.mavenInstaller {
            // This generates POM.xml with proper parameters
            pom {
                project {
                    packaging 'aar'
                    name 'This is iot sdk'
                    developers {
                        developer {
                            id 'fighter_lee'
                            name 'liyang'
                            email 'liyang@xxx.com'
                        }
                    }
                }
            }
        }
    }

    //上传到jcenter
    Properties properties = new Properties()
    properties.load(project.rootProject.file('local.properties').newDataInputStream())
    bintray {
        user = properties.getProperty("bintray.user")    //读取 local.properties 文件里面的 bintray.user
        key = properties.getProperty("bintray.apikey")   //读取 local.properties 文件里面的 bintray.apikey
        configurations = ['archives']
        pkg {
            userOrg = "fighter"
            repo = "maven"
            name = libName    //发布到JCenter上的项目名字，必须填写
            desc = 'This is a iot sdk'    //项目描述
            licenses = ["Apache-2.0"]
            publish = true
        }
    }


项目下的build.gradle配置：

    buildscript {
        repositories {
            jcenter()
        }
        dependencies {

            classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.2'
            classpath 'com.github.dcendents:android-maven-gradle-plugin:1.5'

            // NOTE: Do not place your application dependencies here; they belong
            // in the individual module build.gradle files
        }
    }

    allprojects {
        repositories {
            jcenter()
        }

        tasks.withType(Javadoc) {
            options.addStringOption('Xdoclint:none', '-quiet')
            options.addStringOption('encoding', 'UTF-8')
        }
    }

在项目下的local.properties下添加如下信息：

    bintray.user=liyang
    bintray.apikey=xxxxx

其中，apikey的获取位置如图：

![O5KS{CJ6R7F}_LNN4R6Q$_N.png](http://upload-images.jianshu.io/upload_images/4126773-4ff1ef54ad82cdc0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

输入密码后就能看到了


###4.编译上传

执行命令： gradlew install
成功后可以看到如下图的doc文档：

![]9NDDORD(N8}J3N8}3CGO)P.png](http://upload-images.jianshu.io/upload_images/4126773-180137b0c1eb907d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后执行：gradlew bintrayUpload
如果build success，那么则上传成功了，当然基本上很难一次传成功，坑很多，具体的坑请看下文。

然后我们去后台看看我们上传的版本：

![ZUM{6W(GZP1LQTAX@{SCZU5.png](http://upload-images.jianshu.io/upload_images/4126773-f0fbf83c85da242f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![F%Q5@5)DPB7AWS0H5PEW`XT.png](http://upload-images.jianshu.io/upload_images/4126773-b74fd9ef8273546b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击add to jcenter 按钮后，申请将我们的依赖库传入jcenter，一般隔夜能审核通过。

###5.坑与解
1. 坑一：错误: 编码GBK的不可映射字符->请正确配置javadoc编码
   在项目下的build.gradle 下添加如下，将中文注释改成英文，别问我怎么知道的~
    
    tasks.withType(Javadoc) {
           options.addStringOption('Xdoclint:none', '-quiet')
            options.addStringOption('encoding', 'UTF-8')
       }

2. 坑二：Could not create version ‘0.1’: HTTP/1.1 401 Unauthorized [message:This resource requires authentication]
原因：没有正确配置API key

3. 坑三：Could not create package 'fighter/xxx/xxx': HTTP/1.1 404 Not Found [message:Repo 'xxx' was not found]
原因：信息匹配错误，请参考下图对照：

![AUGQCZFD3{9LT36J5)9~$QF.png](http://upload-images.jianshu.io/upload_images/4126773-974706772f97ff6c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://i.imgur.com/4vwfoFj.png)

4 .  坑四：没有Add to JCenter按钮：
注意：在这个地址注册：[https://bintray.com/signup/oss](https://bintray.com/signup/oss)；不是[https://bintray.com/signup](https://bintray.com/signup)；这两个地址不一样的！

5 . 坑5： 今天出现了个问题，403 [message:forbidden]，也谷歌了很久，原因是我讲userOrg字段写错了，所以一定要按我的坑3的匹配顺序来写相应的字段。