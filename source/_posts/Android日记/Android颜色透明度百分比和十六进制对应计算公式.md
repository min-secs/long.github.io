---
title: Android颜色透明度计算公式
date: 2020-02-06 08:35:05
tags: 
categories: Android日记






---



>有时候UI设计颜色多少透明度的时候,都去网上找对应关系图,其实有一个公式可以自己算出来

十六进制的FF-->十进制的255

- 90%透明度 
十进制255*0.9 = 229.5 ≈ 230(四舍五入) -->十六进制E6

其它透明度以此类推
![程序员计算器](https://upload-images.jianshu.io/upload_images/2226681-47375f771c7e80f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
