---
title: Markdown
date: 2020-02-06 08:35:05
tags: 
categories: 随笔





---



>前言:之前一直用富文本的编辑模式,但是对代码的支持不友好,因此练习了一下markdown,记录下来,对自己有个提醒,也希望能帮助到大家[更多技巧](https://segmentfault.com/markdown)
***
#1.标题

#标题   ---->  #标题  
##标题   ---->  ##标题  
###标题   ---->  ###标题  
####标题   ---->  ####标题  
#####标题   ---->  #####标题  
######标题   ---->  ######标题  
目前支持六种标题大小
#2.引用
输入大于号后跟上文本,就像上面前言中显示的样式.eg: >你好
>你好
- # 程序员版本
##3.1插入一行代码
首先我们另起一行,然后在行首连续输入两个"tab"键,跟上代码即可(`一定要另起一行`) tabtab db.addColumn(x.class,"test");//新增的字段

    db.addColumn(x.class,"test");//新增的字段
##3.2插入代码块
连续插入代码,我们需要连续输入三个符号 ```,后面可以跟上语言种类如java,然后代码块输入完毕,换行,在输入三个符号表示代码块结束 `键盘esc下面,波浪号的那个键`
```java
//```java(某种编程语言)
//代码段
//```
for (Element e : c) {
    doSomething(e);
}

//before JDK1.5
for (Iterator i = c.iterator(); i.hasNext(); ) {
    doSomething((Element) i.next());
}
```

>上面基本上可以满足程序员的基本要求了
##4.插入链接
在简书中的用法是![链接语法.png](https://upload-images.jianshu.io/upload_images/2226681-dca0142cb829876a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
[DDMS性能调优](https://www.jianshu.com/p/829ed0e6010e)
