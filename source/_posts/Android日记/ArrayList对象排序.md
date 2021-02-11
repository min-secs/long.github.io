---
title: List对象排序
date: 2020-02-06 08:35:05
tags: 
categories: Android日记






---



>前言:最近对收音机的开发,遇到一个需求,将收藏列表显示在前,电台列表显示在后,所以需要对列表进行对象排序,在此做一个总结.
#### 步骤1 创建比较器,指定排序规则
导入此包 java.util.Comparator
```java
comparator = new Comparator<RadioNode>() {
			public int compare(RadioNode s1, RadioNode s2) {
				if (s1.isFavor == s2.isFavor) {
					return s2.frequent - s1.frequent;
				}else{
					if(s1.isFavor)	return -1;
					if(s2.isFavor)	return 1;
				}
				return -1;
			}
		};
```
这里面有两个对象s1和s2,下面是制定的比较规则,如果isFavor相同,则比较frequent
返回1表示s1比s2大,则s1的位置不动,s2继续与后面的比较
返回0表示俩一样大,位置不变
返回-1表示s1与s2交换位置,s1继续按规则比较
#### 步骤2 将集合传入
导入此包java.util.Collections

    Collections.sort(favorList,comparator);
`end`
感谢android
