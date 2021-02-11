---
title: Android基础进阶之EffectiveJava翻译系列(第二章-通用方法)
date: 2021-02-06 08:35:05
tags:
categories: Android日记


---



#### 2.所有对象都有的方法(equals,hashCode...)
##### Item8 遵循equals的通用规则

+ 反射性:if x != null ,x.equals(x) must return true; 
+ 对称性:if x != null && y != null ,x.equals(y) is true; y.equals(x) must return true
+ 传递性:if x != null && y != null && z != null ,x.equals(y) is true; y.equals(z) is true ; x.equals(z) must return true
+ 一致性:不管调用多少次equals方法,返回的值一致
+ if x != null,x.equals(null) must return false
>除非有数学上的倾向要违背这些,否则必要遵循
当父类和子类都有自己的equals方法时,某些场景会违背这些原则,非要使用equals的话,建议将子类替换成组合的方式
当自己重写equals方法问自己三个问题:它是对称性的吗?是可传递的吗?是一致的吗?
#####Item9 重写equals时始终重写hashCode
重写equals而没有重写hashCode会导致用到hash值的类出现bug,如HashMap,HashSet..
下面是使用Object在JavaSE6中的规则:
+ 在一次程序操作中的同一个对象,任何时候hashCode保持一致
+ 如果两个对象根据equals(Object)方法相等，那么对这两个对象中的每个对象调用hashCode方法都必须产生相同的整数结果
+ 如果两个对象根据equals(Object)方法不相等，那么对这两个对象中的每个对象没必要产生相同的整数结果,但是有利于提高HashTable的性能
>生成hashCode时,属性中可计算的则直接参与计算,不可计算的字段调用它的hashCode()参与计算



#####  Item10 始终重写toString方法

>不重写的话会得到Object@163b91一串无意义的打印
最好是打印出自己需要的关键字段信息



##### Item11 明智的重写clone方法

>使用的较少,理解不深,后续补充



##### Item12 实现 Comparable接口
对于需要排序和比较的对象,实现此接口定义自己的排序规则

---

[上一章:创建和销毁对象](https://www.jianshu.com/p/96ba1f9da2b6) 
[下一章:类和接口](https://www.jianshu.com/p/d8e8a6916325) 
