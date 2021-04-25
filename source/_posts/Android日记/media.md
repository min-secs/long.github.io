---
title: Android-Media
date: 2021-04-25 08:35:05
categories: Android日记

---



# 图像

1. 图片大小计算
  
* 分辨率x图片位深(kb)
  
  ```
  360px*360px*4 bytes-per-pixel * 50 images = 506 kiB * 50 images = 24.7 MiB
  ```
2. 内存优化 

   首次获取图片的宽高, 计算取样比例

   然后加载真实图片

```kotlin
val point = Point()
windowManager.defaultDisplay.getSize(point)
val options = BitmapFactory.Options()
options.inJustDecodeBounds = true
BitmapFactory.decodeFile(columnName, options)
val heightRadio = options.outHeight / point.y
val widthRadio = options.outWidth / point.x
options.inSampleSize = Math.max(heightRadio, widthRadio)
options.inJustDecodeBounds = false
val decodeFile = BitmapFactory.decodeFile(columnName, options)
```



<!-- more -->

## 库

* Glide
  1. activity销毁加载怎么停止的
  2. 一张图片如何加载
  3. 缓存机制
* Picasso
* universalImageLoader



















# 音频

dumpsys media_session //查看当前session状态

dumpsys audio //查看当前audio状态

## 1.**基础概念**

> 模拟信号 -> 采样 -> 量化 -> 编码 -> 数字信号  

​		人耳听到的最大频率为20kHz, 根据奈奎斯特采样定理：为了不失真地恢复模拟信号，采样频率应该不小于模拟信号频谱中最高频率的2倍。

所以常用的采样率就是44.1KHz  --> 保存频率

位数 --> 保存振幅(取相似的整数值)

##### 声道数

声道数，是指支持能**不同发声**（注意是不同声音）的音响的个数。

单声道：1个声道
 双声道：2个声道
 立体声道：默认为2个声道
 立体声道（4声道）：4个声道

##### 码率

码率，是指一个数据流中每秒钟能通过的信息量，单位bps（bit per second）

码率 = 采样率 * 采样位数 * 声道数



# 视频

## 1.基础概念

### 帧: 一副画面

### 帧率: 单位fps, (frames per second) 一秒内帧数

​	24/25 fps --> 电影

​	30/60 fps --> 游戏

	>85 fps 肉眼无法感知

### 色彩空间

* RGB

* YUV

  Y: 亮度, 含有较多的绿色通道

  U:蓝色通道与亮度差值

  V:红色通道与亮度差值

##### RGB和YUV的换算

```
Y = 0.299R ＋ 0.587G ＋ 0.114B 
U = －0.147R － 0.289G ＋ 0.436B
V = 0.615R － 0.515G － 0.100B
——————————————————
R = Y ＋ 1.14V
G = Y － 0.39U － 0.58V
B = Y ＋ 2.03U
```





# 调优

## 1. ANR

* 抓取data/anr下面的日志分析

```java
ANR in
```

```
group="main" sCount=1 
```

有可能是主线程繁忙,有可能是其他线程持有了主线程需要的锁



* 使用Cpu Profile分析主线程使用情况

* bugreport

  同步时间轴, 开始日志 ```dumpstate: begin```

  ```
  dumpsys activity processes//查看应用oom_adj
  ```

## 2. Crash

## 3. 续航

## 4.  减小体积

## 5. 启动优化

## 6. 内存

* dumpsys meminfo packagename

![image-20201118144402322](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201118144402322.png)

查看内存占用,和当前Activity数量

* 使用AS profiler

  MEMORY中GC之后,dump当前内存, Leaks之后,跟踪排查相关引用

* MAT

  导入hprof 
  
## 7. CPU
## other 黑屏如何分析?