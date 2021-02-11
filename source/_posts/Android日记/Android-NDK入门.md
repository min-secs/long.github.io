---
title: NDK入门
date: 2021-02-06 08:35:05
tags:
categories: Android日记


---



>最近需要打包C++文件成so库,按照网上教程同步的方式,build/intermediates下始终没有ndk文件中的so库

当前环境
windows 10
Android Studio3.5.3
NDK r21
* 配置NDK环境变量
我的电脑 右键-->属性-->高级-->环境变量 path中添加ndk-bundle路径
* 新建jni文件
src/main/java同级目录下src/main/jni
将c++文件和头文件全部放入
* jni目录下新建Android.mk文件
你的so库名称对应java类中System.loadLibrary("so库名称")
```java
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)

LOCAL_MODULE := 你的so库名称 
LOCAL_SRC_FILES := 你的.cpp
include $(BUILD_SHARED_LIBRARY)
```
* jni目录下新建Application.mk文件
```APP_ABI := all```
打包所有支持cpu架构
* 在jni目录下打开power shell
输入ndk-build
![ndk-build.png](https://upload-images.jianshu.io/upload_images/2226681-69efe3b9b940f9e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
jni同级目录下会有libs/* 各平台so库文件
* app build.gradle中配置so库路径
```java
android{
 //***
  sourceSets {
        main() {
            jniLibs.srcDirs = ['src/main/libs']
            jni.srcDirs = [] 
        }
    }
}

```

---

