---
title: 告别USB数据线,使用wifi调试
date: 2021-02-06 08:35:05
tags: 
categories: 车机



---



>前言:目前在车机开发多媒体的U盘播放功能,每次插U盘就不能调试,调试就不能插U盘很不方便.有了远程调试程序,终于方便多了,唯一的缺点是wifi不稳定容易连接超时
##### 步骤一
使用USB数据线成功连接(抱一下大腿先)程序.打开cmd,输入
```adb tcpip 5555```
看到![tcp_success](http://upload-images.jianshu.io/upload_images/2226681-a657793cd1ff5b96.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
即表示成功.
##### 步骤二
打开手机wifi查看当前IP
```adb connect IP:5555```
此时就算拔掉USB,可以看到Eclipse还是连接着设备
#### 后记
使用adb的时候容易遇到一个错误
```ADB server didn't ACK```
一般情况下是端口号被360手机助手等等软件占用了
* 解决办法
在cmd窗口中
```adb nodaemon server```
查看一下哪个端口被占用了(比如我的 5037)
然后
```netstat -ano | findstr "5037"```
找到被占用的端口,在任务管理器中将其结束掉
