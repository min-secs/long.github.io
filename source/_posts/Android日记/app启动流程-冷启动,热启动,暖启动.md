---
title: app启动流程
date: 2020-02-06 08:35:05
tags: 
categories: Android日记






---



>前言:项目中被诟病多媒体开机第一次启动很慢,大约3s的黑屏,后续点开启动正常,其中什么原理呢?这就涉及到系统的三种启动模型

### 1.Cold Start,冷启动
system--->
loding and launching the app
                  displaying a blank window

createing the app process
---
process--->
creating the application
launching the main thread
creating the main activity
inflating views
laying out the screen
perfoming the initial draw
main activity place blank window
---
### 2.Hot Start,热启动
前台activity因内存不足,被系统销毁重建的过程(重建流程同冷启动)

### 3.Warm Start

* 用户点击返回,并重新运行
* activity不在前台,被系统销毁,由用户主动运行
可通过onSaveInstance保存状态
---
### 4.APP启动慢常见问题
* Application.onCreate中执行了过重的操作,如I/O操作,频繁创建对象等
* Activity.onCreate
布局过于复杂
Loading and decoding bitmaps
blocking screen drawing on disk or network I/O
Rasterinzing vectordrawable objects
initialztion of other subsystem of the activity
