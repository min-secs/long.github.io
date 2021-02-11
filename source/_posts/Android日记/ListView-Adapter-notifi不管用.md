---
title: ListView-Adapter刷新无作用
date: 2020-02-06 08:35:05
tags: 
categories: Android日记






---



>前言,在使用到adapter的时候，一般要用List来装数据实体，这里两种不同的写法容易遇到不同的问题。

* 第一种写法

```java
class TestAdapter{
    private List<Node> list;
    ***

    public TestAdapter(List list){
        this.list = list;
        ***
    }
}
```
这样的话,我们在list更新的时候直接调用adapter的notifydatasetchanged就可以了.
* 第二种写法
```java
class TestAdapter{
    private List<Node> mList = new ArrayList<Node>();
    ***

    public TestAdapter(List list){
        mList.addAll(list);
        ***
    }

    public void addAll(List list){
        mList.clear();
        mList.addAll(list);
        notifydatasetchanged();
    }

 public void addOne(Node node){
        
        mList.add(node);
        notifydatasetchanged();
    }
}
```
用这种写法在数据变化的时候,需要调用adapter.add*()的对应方法
* 总结
adapter更新是看对象的地址有没有变化,调用notifydatasetchanged()才会管用.
* 问题
notifydatasetchanged不管作用
1.一般情况下,遇到notifydatasetchanged不管作用是指向的对象已经不是初始化adapter时的那个对象了.比如使用了上面第二种写法,却调用的第一种的方式.
2.list的size==0;
3.***

