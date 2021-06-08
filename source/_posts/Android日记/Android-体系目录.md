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



##### 2.binder

###### Q1 Service完整传输数据流程

 >= 21的版本, 不准隐式启动服务
 >


###### Q2 如何从java中定义的native方法, 找到实现类

A2 根据命名规则, 一般为(native)__javaName_(native)

###### Q3 服务何时注册到native层ServiceManager中的



##### 3. Activity启动流程

##### 4.PMS

##### 5.WMS

##### 6.签名

##### 7.四大组件



#### 思架构

view体系

input体系

audio体系

video体系

#### 思网络