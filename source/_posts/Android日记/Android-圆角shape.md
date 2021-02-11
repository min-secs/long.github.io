---
title: 圆角shape
date: 2021-02-06 08:35:05
tags:
categories: Android日记


---



```java
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <!-- 填充的颜色 -->
    <solid android:color="@color/bottom_controller_color" />

    <!-- 边框的颜色和粗细 -->
    <stroke
        android:width="1dp"
        android:color="@color/bottom_controller_color"
        />

    <corners
        android:bottomLeftRadius="0dp"
        android:bottomRightRadius="0dp"
        android:radius="20dp"
        android:topLeftRadius="0dp"
        android:topRightRadius="18dp"
        />

</shape>
```
