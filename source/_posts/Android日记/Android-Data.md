---
title: Android-Data
date: 2021-02-06 08:35:05
tags:
categories: Android日记


---



### Android文件系统入门知识

> 内外存储的由来:因为历史原因,老版本的Android手机可用容量小, 加载外部存储设备以便加大容量, 如sdcard. 后来随着技术的不断提高, 手机本身的容量逐渐加大, 但是这个分法一直保留了下来.

**二者区别**

*   内部存储

    不需要权限

    其它APP不可访问, 用户也不能操作

    卸载APP时会移除内部存储数据

*   外部存储

    首先要确认可访问性, 因为外部存储可卸载

    其它APP可以访问

    需要权限

 ```java
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
 ```

**内部存储简单介绍**

所在路径: Android 6.0在/data/user/0/package_name/

其它版本也对应在/data/下的某个地方, 以包名作为区分, 未root的文件管理中查看不了

**使用:**

在继承自ContextWrapper中, 如Activity, Application

```java
//得到文件对象
File file = getFilesDir();

String inPath = file.getAbsolutePath();

Log.d("test-file","getFileDir:"+ inPath);

//在inPath中写入hello
FileOutputStream fileOutputStream = openFileOutput("hello.txt", MODE_PRIVATE);
fileOutputStream.write("hello".getBytes());

fileOutputStream.close();
```

**外部存储简单介绍**

通过android.os包中的Environment类

```java
//判断文件可用性
/* 可写 */
public boolean isExternalStorageWritable() {
    String state = Environment.getExternalStorageState();
    if (Environment.MEDIA_MOUNTED.equals(state)) {
        return true;
    }
    return false;
}

/* 可读 */
public boolean isExternalStorageReadable() {
    String state = Environment.getExternalStorageState();
    if (Environment.MEDIA_MOUNTED.equals(state) ||
        Environment.MEDIA_MOUNTED_READ_ONLY.equals(state)) {
        return true;
    }
    return false;
}

//得到外部存储私有对象(File),会随着APP的卸载而卸载
Environment.getExternalStorageDirectory();

//得到外部存储公共对象, 文件不会随着APP的卸载而卸载
//传入的参数代表文件类型 如Environment.DIRECTORY_MUSIC 代表音乐
//DIRECTORY_RINGTONES 代表铃音
Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_MUSIC));
```

得到File对象后, 你就可以为所欲为

[File类](https://developer.android.google.cn/reference/java/io/File.html#public-methods)

---
###共享数据
>记得第一份工作面试的时候被问了一个问题:如何自己实现类似友盟分享功能?
当时回答的很含糊, 现在再来看, 就是通过Intent来定义分享action, 遍历所有的APP, 显示符合这个action的应用, 按规则启动隐式Intent
- 简单数据
通过Intent
- 文件
通过V4包中的[FileProvider](https://developer.android.google.cn/reference/androidx/core/content/FileProvider.html)

