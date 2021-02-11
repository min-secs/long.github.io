---
title: Android-Op
date: 2021-02-06 08:35:05
tags:
categories: Android日记



---



>
>
>Android 优化篇

##### 布局优化/绘制优化

- 原则
避免嵌套过多,可采用约束布局
 ```java
implementation 'com.android.support.constraint:constraint-layout:1.1.3'
 ```
- 工具
Android Studio在tools菜单栏选择layout工具查看
![Layout Inspector](https://upload-images.jianshu.io/upload_images/2226681-d0ff0e0407c64127.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
---
##### 内存优化

- 原则
  避免创建过多不必要的对象,尤其是在循环中
  比如不要在onDraw中new Paint

- 工具
  查询内存泄漏
  LeakCanary

  

  ##### cpu优化

- 原则
  避免多次执行同一个耗时方法

- 工具
TraceView 可找出单次执行耗时方法和多次调用的方法
Android5.0 AS3.0之后可使用![Profiler](https://upload-images.jianshu.io/upload_images/2226681-b8b2cc90602d1eb5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 网络优化
车机项目中使用本地较多,暂略
##### 电量优化
昝略

##### apk体积优化

- 原则
  1. 去除不用的资源
  2. 开启混淆
  3. 新型构建工具Bundle
- 工具
Lint
静态代码检测工具
![Lint](https://upload-images.jianshu.io/upload_images/2226681-492f842ffa340630.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

双击Shift开启搜索,输入![去除多余资源](https://upload-images.jianshu.io/upload_images/2226681-bb2565cc1c32eed2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
