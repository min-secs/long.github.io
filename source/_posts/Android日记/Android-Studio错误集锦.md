---
title: AS错误集锦
date: 2021-02-06 08:35:05
tags:
categories: Android日记



---



- Q1:Error: INSTALL_FAILED_INVALID_APK
关闭 File/Setting/Build/instant run 

![instant run .png](https://upload-images.jianshu.io/upload_images/2226681-b03a7dfcfd4de1eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- Q2:Installation failed with message Failed to commit install session 872031752 with command pm install-commit 872031752. Error: INSTALL_FAILED_INVALID_APK: /data/app/vmdl872031752.tmp/1_slice__ signatures are inconsistent
上面那个方式未解决的话, 重新clean一次项目, 在运行

![clean.png](https://upload-images.jianshu.io/upload_images/2226681-984a8197dba513d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
