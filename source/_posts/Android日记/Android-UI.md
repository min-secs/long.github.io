---
title: Android-UI
date: 2021-02-06 08:35:05
tags:
categories: Android日记
toc: true

---



>Android知识体系,笼统分为UI,Data,interact

#### UI
1.屏幕适配

[郭霖微信公众号:屏幕适配](https://mp.weixin.qq.com/s?__biz=MzA5MzI3NjE2MA==&mid=2650243915&idx=1&sn=7054d165d06caf917b6a2b57800316a2&chksm=88637224bf14fb320b689948ebb3fd0d783bd816c7184644abea51e26be6c2ec50b54e61b46c&scene=38#wechat_redirect)
---
adb修改分辨率和屏幕密度
adb shell wm size 1280x720 //小写的x
adb shell wm density 240
adb shell wm size reset //还原默认
adb shell wm density reset
---
2.自定义view
[扔物线大神](https://hencoder.com/)
---
3.优化布局
- 减少嵌套

- 合理使用<include/><merge/>

- 延迟加载<ViewStub>

  <!--more-->
---
4.优化应用启动时间
- 从Application到Activity,Fragment中的生命周期方法避免重操作,100ms的差距往往影响甚大
可以通过handler延迟初始化一些资源,显示加载框,初始化完成后在隐藏(亲测延迟加载Fragment,界面快了2.5S)
- 给Activity或者Application添加背景图片
```java
//MediaTheme
 <style name="MediaTheme" parent="@android:style/Theme.NoTitleBar">
        <item name="android:windowBackground">@drawable/yourpic</item>
 </style>

//Manifest
 <application
        android:name=".MediaApplication"
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/MediaTheme">
```
