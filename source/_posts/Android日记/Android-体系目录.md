---
title: Android-体系目录
date: 2021年4月7日 08:57
tags:
categories: Android日记

---

#### 思源码

##### 1.handle

​	framwork层![工作流程](http://gityuan.com/images/android-arch/handler_thread_commun.jpg)

​	native层

​	当消息队列无消息时, native层做的休眠, 发送消息通过往管道写数据唤醒

###### Q1 Looper在什么时候阻塞等待的?

A1 messageQueue 为null,在native层做的等待

###### Q2 enqueueMessage为何用的for(;;)发送消息, 不应该只发送一条吗

A2 头部消息为null时,或者不是延时消息时,调用一次发送, 如果是延时消息需要加入到单链表中, 需要一次挪next的位置

###### Q3 MessageQueue合适调用mQuit?

A3



##### 2.binder

> 2021年6月7日 开始

#####  Service完整传输数据流程

![架构](D:\资料\long\boke\github.io\source\_posts\Android日记\Android-体系目录.assets\ams_ipc.jpg)
//

![startService完整流程](D:\资料\long\boke\github.io\source\_posts\Android日记\Android-体系目录.assets\binder_ipc_process.jpg)
//

![通信协议](D:\资料\long\boke\github.io\source\_posts\Android日记\Android-体系目录.assets\binder_transaction.jpg)

 >= 21的版本, 不准隐式启动服务

###### Q1 如何从java中定义的native方法, 找到实现类

A1 根据命名规则, 一般为(native)__javaName_(native)

###### Q2 服务何时注册到native层ServiceManager中的

A2 首次bind的时候

##### 3. Activity启动流程

![Activity启动流程](D:\资料\long\boke\github.io\source\_posts\Android日记\Android-体系目录.assets\start_activity_process.jpg)

##### 4.PMS

管理应用的安装, 卸载, 具体在installd守护进程中实现

##### 5.WMS



##### 6.签名

##### 7.四大组件

##### 8.Glide数据结构

##### 9.原生扫描逻辑



#### 思架构

##### view体系

##### input体系

##### audio体系

声音卡顿常见解决方案

1. 增大AudioTrack bufferSizeInBytes * 2

2. 设置进程优先级为audio 

   

   ```java
   int priority = android.os.Process.getThreadPriority(android.os.Process.myTid());
   if(priority != android.os.Process.THREAD_PRIORITY_AUDIO) {
       android.os.Process.setThreadPriority(android.os.Process.THREAD_PRIORITY_AUDIO);
   }
   ```

   

Q1 声音是如何采集和输出的?

video体系

Q1 视频是如何采集和输出的?

#### 思网络

#### 思基础

##### 1. Java类初始化顺序

##### 2. hashmap实现原理