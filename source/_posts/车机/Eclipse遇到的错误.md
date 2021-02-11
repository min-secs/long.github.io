---
title: Eclipse遇到的错误
date: 2020-02-06 08:35:05
tags: 
categories: 车机




---



### Q1 Conversion to Dalvik format failed: Unable to execute dex: Multiple dex files define L

A1：添加的jar包重复了，可能名字不同，里面的内容相同。这点就做的不好了，提示不出来哪两个文件相同。

### Q2 Unknown exception in parseSdkContent. java.lang.StackOverflowError

A2:主工程A  Lib (B C) BC之间相互依赖