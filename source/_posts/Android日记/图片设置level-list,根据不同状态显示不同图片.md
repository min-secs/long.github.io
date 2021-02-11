---
title: 图片分层根据状态显示
date: 2021-02-06 08:35:05
tags:
categories: Android日记


---



>前言:工作需求,wifi图标根据不同的强度,显示不同的状态.于是想到用图层的方法
####bg.xml
```java
<?xml version="1.0" encoding="utf-8"?>

<level-list xmlns:android="http://schemas.android.com/apk/res/android" >
    <item android:maxLevel="0" android:drawable="@drawable/icon_stop_n"></item>
    <item android:maxLevel="1" android:drawable="@drawable/icon_pause"></item>

</level-list>
```
`note:android:maxLevel 必须从0递增,顺序错误后只会显示第一张图片`
####布局文件
```
 <Button
                android:id="@+id/bt_pause_bt"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginLeft="115px"
                android:background="@drawable/bg"
                android:tag="play" />
```
####代码中使用
```java
LevelListDrawable pauseDrawable = (LevelListDrawable) yourwiget
				.getBackground();
pauseDrawable.setLevel(1);//根据业务需要,对应图片等级
```
