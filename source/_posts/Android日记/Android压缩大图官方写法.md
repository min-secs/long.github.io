---
title: Android压缩大图官方写法
date: 2020-02-06 08:35:05
tags: 
categories: Android日记






---



>前言:之前写多媒体的时候,加载图片使用的Picasso框架,但图片列表很多依然有oom的问题,原来的图片分辨率(5024*4280)太大了,遂要压缩图片

对于一张5024\*4280的图片(ARGB_8888 )来说,系统要分配多少内存呢?计算方法如下
5024\*4280*4byte 约等 82.026M,吓人不
对于android设备来说,丢失一点像素点肉眼基本看不出来(肉眼八倍镜除外),所以如下是官方提供的demo
```java
public static Bitmap decodeSampledBitmapFromResource(Resources res, int resId,
        int reqWidth, int reqHeight) {

    // First decode with inJustDecodeBounds=true to check dimensions
    final BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    BitmapFactory.decodeResource(res, resId, options);

    // Calculate inSampleSize
    options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);

    // Decode bitmap with inSampleSize set
    options.inJustDecodeBounds = false;
    return BitmapFactory.decodeResource(res, resId, options);
}

public static int calculateInSampleSize(
            BitmapFactory.Options options, int reqWidth, int reqHeight) {
    // Raw height and width of image
    final int height = options.outHeight;
    final int width = options.outWidth;
    int inSampleSize = 1;

    if (height > reqHeight || width > reqWidth) {

        final int halfHeight = height / 2;
        final int halfWidth = width / 2;

        // Calculate the largest inSampleSize value that is a power of 2 and keeps both
        // height and width larger than the requested height and width.
        while ((halfHeight / inSampleSize) >= reqHeight
                && (halfWidth / inSampleSize) >= reqWidth) {
            inSampleSize *= 2;
        }
    }

    return inSampleSize;
}
```
用法如下
```java
mImageView.setImageBitmap(
    decodeSampledBitmapFromResource(getResources(), R.id.myimage, 100, 100));
```
网上有人说decodeStream会比decodeResource更节省内存,官方并无相关说明,需要用哪种方法解析Bitmap就用哪个方法吧,主要还是在于压缩图片

压缩原理就我的理解是先不分配图片的内存,待针对不同的设备(中密度,高密度,超高密度等)计算好压缩比例后再分配内存

ps:Picasso加载图片有一个fit()方法会根据imageview的大小自动压缩图片,就不需要上面的步骤了
