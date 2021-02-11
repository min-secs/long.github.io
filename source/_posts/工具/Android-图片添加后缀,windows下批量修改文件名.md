---
title: windows批量修改文件名
date: 2021-02-06 08:35:05
tags: 
categories: 工具



---



>Android开发遇到了换肤的要求,使用了鸿洋大神的ChangeSkin库,需要大量的给图片加后缀


# [方法摘自:真紫艳星光](https://www.cnblogs.com/xingfan/)

#### 第一步 在文件夹(如F:/pic)放图片处shift加右键,在此处打开命令窗口
输入
dir *.* /b>rename.xls
会把当前文件夹下的文件名保存在表格中
![表格_rename](https://upload-images.jianshu.io/upload_images/2226681-0602f8df7ac9a5b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 第二步 复制一份到另一个表格
![新建表格](https://upload-images.jianshu.io/upload_images/2226681-e28df338454471ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Ctrl+F替换为对应的后缀名称
![替换](https://upload-images.jianshu.io/upload_images/2226681-09c12f57d7cb3638.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

复制到第一个表中
![复制](https://upload-images.jianshu.io/upload_images/2226681-3e57b9f236bfd870.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 第三步 输入公式
在C1单元格
="ren "&A1&" "&B1
![公式效果](https://upload-images.jianshu.io/upload_images/2226681-de23c5e6f436b5a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
鼠标放到这个点,变成黑色十字架时往下拖
![image.png](https://upload-images.jianshu.io/upload_images/2226681-c3894f8fe8d970d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/2226681-c9615d350dfcd067.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 第四步 复制C列的内容 在F:/pic新建文本文档拷贝
![image.png](https://upload-images.jianshu.io/upload_images/2226681-a6266be202e843a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 第五步 修改文本文档后缀为.bat,双击执行
原来的文件就添加上后缀啦!
![修改后的文件](https://upload-images.jianshu.io/upload_images/2226681-edc6a3730a836caa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
