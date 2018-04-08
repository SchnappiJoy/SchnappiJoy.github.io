---
layout: post
title: 'Android中的Gradle配置笔记'
subtitle: '转载请注明出处'
date: 2018-04-08
categories: Android  Gradle 
cover: 'http://bpic.588ku.com/back_pic/03/60/56/8357a5ebfd572fd.jpeg'
tags: Android Gradle
---

 
 <center>
 <font color="#4590a3" size="4px">工程目录下新建config.gradle</font>
 </center>
 
 <br>
 
```java
ext {

    android = [
            compileSdkVersion: 26,
            buildToolsVersion: "27.0.3",
            minSdkVersion    : 19,
            targetSdkVersion : 26,

    ]

    dependencies = [
            "appcompatV7": "com.android.support:appcompat-v7:26.1.0",
            "design"      : "com.android.support:design:23.2.1",
            "constraintLayout": "com.android.support.constraint:constraint-layout:1.0.2"
    ]
}
```

 <center>
 <font color="#4590a3" size="4px">根目录Gradle配置</font>
 </center>
 
 <br>
 
 ```java
 // Top-level build file where you can add configuration options common to all sub-projects/modules.
apply from: "config.gradle"
buildscript {
    
    repositories {
        google()
        jcenter()//使用jcenter库
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.1'
        

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
//为所有的工程的repositories配置为jcenters
allprojects {
    repositories {
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

```

 <center>
 <font color="#4590a3" size="4px">模块下Gradle配置</font>
 </center>  
 
<br>


```java
apply plugin: 'com.android.application'

android {
    compileSdkVersion  rootProject.ext.android.compileSdkVersion
    buildToolsVersion  rootProject.ext.android.buildToolsVersion
    defaultConfig {
        applicationId "top.schnappi.mvpdemo"
        minSdkVersion rootProject.ext.android.minSdkVersion
        targetSdkVersion rootProject.ext.android.targetSdkVersion
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
        buildConfigField 'String', 'API_SERVER_URL', '"http://wuxiaolong.me/"'
    }

    signingConfigs {//签名的配置
        release {
            storeFile file('xiaoyu.jks')
            storePassword 'android'
            keyAlias 'android'
            keyPassword 'android'
        }
        debug {
            storeFile file('xiaoyu.jks')
            storePassword 'android'
            keyAlias 'android'
            keyPassword 'android'
        }
    }
    buildTypes {
        release {
            minifyEnabled false//是否启动混淆
            zipAlignEnabled false//是否启动zipAlign
            shrinkResources false // 是否移除无用的resource文件
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release//打包命令行：gradlew assembleRelease
            applicationVariants.all { variant ->
                if (variant.buildType.name == 'release') {
                    variant.outputs.all {
                        def apkName = "gradle4android_v${variant.versionName}"
                        if (!variant.flavorName.isEmpty()) {
                            apkName += "_${variant.flavorName}"
                        }
                        outputFileName = apkName + "_${releaseTime()}.apk"
                    }
                }
            }
            //针对很多渠道
            productFlavors.all { flavor ->
                manifestPlaceholders.put("UMENG_CHANNEL_VALUE", name)
            }
        }
    }
    flavorDimensions 'tier'
    productFlavors {
        //多渠道打包，命令行打包：gradlew assembleRelease
        debugServer {
            dimension 'tier'
            buildConfigField 'String', 'API_SERVER_URL', '"我的公众号：吴小龙同学"'
        }

        releaseServer {
            dimension 'tier'
            buildConfigField 'String', 'API_SERVER_URL', '"http://wuxiaolong.me/"'
        }

        xiaomi {
            dimension 'tier'
            applicationId 'com.wuxiaolong.gradle4android1'
        }

        googlepaly {
            dimension 'tier'
            applicationId 'com.wuxiaolong.gradle4android2'
        }
    }
    lintOptions {
        //设置编译的lint开关，程序在buid的时候，会执行lint检查，有任何的错误或者警告提示，都会终止构建
        abortOnError false
    }

}




dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    testImplementation 'junit:junit:4.12'
    compile rootProject.ext.dependencies.appcompatV7
    compile rootProject.ext.dependencies.design
    compile rootProject.ext.dependencies.constraintLayout

}

def releaseTime() {
    return new Date().format("yyyy-MM-dd", TimeZone.getTimeZone("UTC"))
}

```
 
<br>


>  ##### [Demo下载](https://github.com/SchnappiJoy/GradleWithMvpPluginDemo)

>  ##### [推荐一个很好的博主吴小龙同学可以去看看](http://wuxiaolong.me/)

