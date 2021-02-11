---
title: 数组转集合新颖写法
date: 2021-02-06 08:35:05
tags:
categories: Android日记

---



>title: my new post
>date: 2021-02-05 19:04:35
>categories: Android日记
>tags: Android
>
>前言:啥也不说了,show me the code!
```java
static List<Integer> intArrayAsList(final int[] a){
    if(a == null)
        throw new NullPointerException();

    retrun new AbstractList<Integer>() {
        public Interger get(int i) {
            retrun a[i];
        }
        @Override public Integer set(int i, Integer val){
            int oldVal = a[i];
            a[i] = val;
            retrun oldVal;
        }

        public int size(){ retrun a.length;}
    };
}
```
