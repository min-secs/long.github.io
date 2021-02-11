---
title: 重新理解Activity启动模式,确认返回到哪个界面
date: 2021-02-06 08:35:05
tags:
categories: Android日记
---



>------
>
>
>
>前言：之前开发单个应用，把每个Activity声明为singleTask完事，最近开发车机系统相关的应用，各个应用间通过语音可来回跳转，点击返回时返回的Activity和预期不一致，于是研究了一下如何定位到当前Activity点返回会跳转到哪

车机系统 Android4.4
###  所需概念
####  task&stack
Android对于Activity的管理使用First in,Last out的数据结构,对所有的Activity都通过回退栈的方式来管理

启动activity实际上启动的activity所属的task,最顶部的activity处于onResume状态,这点一定要切记

放入stack里面的顺序不能重排序,只能遵从后入先出的原则(pop&push)

Home在一个Stack里面,其它应用在另一个Stack里面,通过Task ID管理

### 使用命令
adb shell dumpsys activity > E:\stack01.txt
或
adb shell dumpsys activity activities > E:\stack02.txt  //生成的文件更详细

此时打开生成的文件找到*Recent tasks:*
![stack.png](https://upload-images.jianshu.io/upload_images/2226681-d3097c4de9f501b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
即表示当前所有系统管理activity栈,Recent #0是当前界面,点击返回即跳到Recent #1,一直返回到tasks结束或Home为止

"sz=1"表示当前task所拥有的Activity数量,在同一个task的Activity回退完后,才会到另一个task,一般来说一个应用拥有一个独立的task,(Home启动应用会通过Intent设置FLAG_ACTIVITY_NEW_TASK的flag)

### 启动模式与任务栈的关系

1. 默认
   默认情况下每次都启动一个新的Activity实例

2. singleTop
   当前Activity在task顶部时,及当前正在交互的Activity需要重新打开时不会创建实例,而会走到onNewIntent里面

非顶部和默认情况下一致

3. singleTask
   当前task里面有实例则会复用,回调onNewIntent,并将当前Activity上面的Activity弹出销毁

4. singleInstance
   同singleTask类似,不同的是这个task里面只会有唯一一个Activity,启动其它Activity会放到新的一个task里面(可理解为给其它Activity设置FLAG_ACTIVITY_NEW_TASK)
   标识)

有一点需要注意,对于从Home启动的Activity来说,会设置mOnTopOfHome=true,不管和Home是不是同一个Stack都会返回到主页
### 小技巧
没有给定API接口启动其他应用(跳到指定activity)尽量使用

```java
   Intent intent = MyApp.getInstance().getPackageManager()
    .getLaunchIntentForPackage("com.example.otherpackage");

   MyApp.getInstance().startActivity(intent);
```
由系统判断Launcher category启动,如果手动启动需要根据当前方案设置intent的flag
