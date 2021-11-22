---
title: Android.mk
date: 2020-02-06 08:35:05
tags: 
categories: 车机



---



>Android源码编译不同的版本,略微有不同
如下基于9.0

编写app之前,确认编译环境使用的sdk版本,在build.gradle中配置成一样的
删掉不需要用的东西,也许某个虚线下面在编译时就是一个炸弹

一个apk构依赖部分
系统属性/自己项目的module/第三方jar/第三方aar/so库
+ #### 系统预编译好的属性
如AndroidX系列,
implementation 'androidx.constraintlayout:constraintlayout:1.1.0
可在编译目录中通过查找对应 name属性
```find prebuilts/sdk/ -name Android.bp|xargs grep "name.*constraintlayout"```

mk如下:
```
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)
LOCAL_USE_AAPT2 := true

...
LOCAL_STATIC_ANDROID_LIBRARIES := \ androidx-constraintlayout_constraintlayout

include $(BUILD_PACKAGE)
```
+ #### 依赖自己项目中Module
1.只依赖java文件,即常用的jar包
module中配置Android.bp
```
java_library {
    name: "constantproxy",

    srcs: [
        "src/main/java/**/*.java",
    ],

    exclude_srcs: [
    ],

    libs: [
    ],

    static_libs: [
        "androidx.media_media", //依赖的系统属性
    ],

    required: [
    ],

    dxflags: [
    ],

}
```
app中Android.mk
```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_USE_AAPT2 := true

...
LOCAL_STATIC_JAVA_LIBRARIES := \
							   constantproxy\

```
2.既依赖java又依赖res
此种module不需要写mk
在app中引入即可
>? 理论上是不是每个module都可以这么写
```
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)
LOCAL_USE_AAPT2 := true

LOCAL_MODULE_TAGS := optional

LOCAL_SRC_FILES := $(call all-java-files-under, src)
LOCAL_SRC_FILES += $(call all-java-files-under, yourmodule/src)
LOCAL_RESOURCE_DIR += $(LOCAL_PATH)/src/main/res
LOCAL_RESOURCE_DIR += $(LOCAL_PATH)/yourmodule/src/main/res
#注意yourmodule路径的准确性

LOCAL_AAPT_FLAGS := --auto-add-overlay \
                                      --extra-packages yourmodule包名 \
...

include $(BUILD_PACKAGE)
```
+ #### 第三方jar
在jar的同级目录下新建Androd.bp 如libs/Android.bp
```
java_import {
    name: "my_jar",
    jars: ["annotation-1.1.0.jar"],
}
```
如果依赖第三方jar的是库,则在库里面的Android.bp中
```
java_library {
    name: "my_lib_core",

    srcs: [
        "src/main/java/**/*.java",
    ],

    exclude_srcs: [
    ],

    libs: [
    ],

    static_libs: [
        "androidx.appcompat_appcompat","my_jar"
    ],

    required: [
    ],

    dxflags: [
    ],

}
```
如果是自己的项目,则在app中Android.mk
```
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)
LOCAL_USE_AAPT2 := true
LOCAL_STATIC_JAVA_LIBRARIES := \
                my_jar \
...

LOCAL_PRIVATE_PLATFORM_APIS := true
include $(BUILD_PACKAGE)
```
+ #### 依赖第三方aar
在aar同级目录Android.bp中
```
android_library_import {
    name: "navigation-fragment-nodeps",
    aars: ["libs/navigation-fragment-2.3.0-alpha01.aar"],
    sdk_version: "current",
    static_libs: [
		"androidx.fragment_fragment", //navigation-fragment依赖androidx.fragment_fragment
    ],
}

android_library {
    name: "androidx.navigation_navigation-fragment",
    sdk_version: "current",
    manifest: "androidx.navigation.fragment/AndroidManifest.xml",
    static_libs: [
        "navigation-fragment-nodeps",
		"androidx.fragment_fragment",
    ],
    java_version: "1.8",
}
```
在Android.mk中
```
LOCAL_STATIC_ANDROID_LIBRARIES := \
                    androidx.navigation_navigation-fragment
```
+ #### so库
>此so库指的是NDK编译出来的so库,libs下直接可用
需要用到so库的mk里面

```
include $(BUILD_PACKAGE)

libs_dir := $(TARGET_OUT)/app/yourapkname/lib/armeabi
$(shell mkdir -p $(libs_dir))
src_files := $(shell ls $(LOCAL_PATH)/libs/armeabi/)
$(foreach file, $(src_files),\
			    $(shell cp $(LOCAL_PATH)/libs/armeabi/$(file) $(libs_dir)/$(file)))

```
---
Android9.0源码中的mk例子
```
LOCAL_PATH:= $(call my-dir)
 
include $(CLEAR_VARS)
 
LOCAL_USE_AAPT2 := true
LOCAL_PACKAGE_NAME := APK名字
LOCAL_OVERRIDES_PACKAGES := 
LOCAL_MODULE_TAGS := optional
 
# can't use LOCAL_SDK_VERSION, otherwise the error will occur like this:
# "(java:sdk) should not link to"
#LOCAL_SDK_VERSION := current
LOCAL_PRIVATE_PLATFORM_APIS := true
LOCAL_PRIVILEGED_MODULE := true
LOCAL_JAVACFLAGS := -Xlint:deprecation -Xlint:unchecked
 
#LOCAL_PROGUARD_FLAG_FILES := proguard.flags
#LOCAL_JARJAR_RULES := $(LOCAL_PATH)/jarjar-rules.txt
 
###################################################
# src, res, AndroidManifest.xml ...
###################################################
LOCAL_MANIFEST_FILE := src/main/AndroidManifest.xml
LOCAL_SRC_FILES := $(call all-java-files-under, src/main/java)
LOCAL_RESOURCE_DIR := $(LOCAL_PATH)/src/main/res
LOCAL_ASSET_DIR := $(LOCAL_PATH)/src/main/assets
 
###################################################
# AS dependencies from androidx or android-support
###################################################
#implementation 'androidx.appcompat:appcompat:1.0.0'
#LOCAL_STATIC_ANDROID_LIBRARIES += androidx.appcompat_appcompat
 
#implementation 'androidx.constraintlayout:constraintlayout:1.1.0'
#LOCAL_STATIC_ANDROID_LIBRARIES += androidx-constraintlayout_constraintlayout
 
#implementation 'androidx.recyclerview:recyclerview:1.0.0'
#LOCAL_STATIC_ANDROID_LIBRARIES += androidx.recyclerview_recyclerview
 
#implementation 'com.google.android.material:material:1.0.0'
#LOCAL_STATIC_ANDROID_LIBRARIES += com.google.android.material_material
 
###################################################
# custom aar library
###################################################
#LOCAL_STATIC_JAVA_AAR_LIBRARIES += 第三方AAR的别名，如：Demo_xxx-1.0.0
#LOCAL_AAPT_FLAGS += --extra-packages 第三方AAR里面AndroidManifest.xml定义的包名
#LOCAL_RESOURCE_DIR += 如果第三方AAR里面有res编不过的时候可以将res解压出来在此引用，如：$(LOCAL_PATH)/libs/xxx/res
LOCAL_AAPT_FLAGS += --auto-add-overlay
 
###################################################
# custom jar library
###################################################
#LOCAL_STATIC_JAVA_LIBRARIES += 自定义Module或第三方JAR的别名
#LOCAL_JAVA_LIBRARIES +=
 
###################################################
# custom jni library
###################################################
LOCAL_JNI_SHARED_LIBRARIES := 
 
include $(BUILD_PACKAGE)
 
 
##############################################################
# Pre-built dependency jars,aars,...
##############################################################
include $(CLEAR_VARS)
 
#LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES += AAR或JAR别名与文件路径对应关系，如：Demo_xxx-1.0.0:libs/xxx/xxx.aar
 
include $(BUILD_MULTI_PREBUILT)
 
 
##############################################################
# Pre-built dependency jars,aars,...
##############################################################
#prebuilts += xxx:libs/xxx/xxx.aar
#
#define define-prebuilt
#  $(eval tw := $(subst :, ,$(strip $(1)))) \
#  $(eval include $(CLEAR_VARS)) \
#  $(eval LOCAL_MODULE := $(word 1,$(tw))) \
#  $(eval LOCAL_MODULE_TAGS := optional) \
#  $(eval LOCAL_MODULE_CLASS := JAVA_LIBRARIES) \
#  $(eval LOCAL_SRC_FILES := $(word 2,$(tw))) \
#  $(eval LOCAL_UNINSTALLABLE_MODULE := true) \
#  $(eval LOCAL_SDK_VERSION := system_current) \
#  $(eval include $(BUILD_PREBUILT))
#endef
#
#$(foreach p,$(prebuilts),\
#  $(call define-prebuilt,$(p)))
#
#prebuilts :=
 
 
##############################################################
# find other Android.mk 
##############################################################
include $(call all-makefiles-under, $(LOCAL_PATH))
```

---
上面的方式都不行的话,直接打包apk编译
xx.apk同级目录Android.mk
```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)

LOCAL_MODULE := weixin
LOCAL_MODULE_CLASS := APPS
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)
LOCAL_SRC_FILES := $(LOCAL_MODULE).apk
LOCAL_MODULE_PATH := $(TARGET_OUT_APP)
LOCAL_CERTIFICATE := platform
include $(BUILD_PREBUILT)
include $(call all-makefiles-under,$(LOCAL_PATH))
```

### 指定if else

```mk
LOCAL_SRC_FILES := $(call all-java-files-under, src/main)
ifeq "$(CA_PRODUCT_VEHICLE_TYPE)" "CD569"
LOCAL_SRC_FILES += $(call all-java-files-under, src/nfore)
#for GaoTong
else ifeq "$(CA_PRODUCT_VEHICLE_TYPE)" "8155"
LOCAL_SRC_FILES += $(call all-java-files-under, src/gt)
$(warning "No define CA Product Vehicle Type")
endif

```

