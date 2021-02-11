---
title: Android基础进阶之EffectiveJava翻译系列(第四章-泛型)
date: 2020-02-06 08:35:05
tags: 
categories: Android日记






---



#### 4. 泛型 
在JDK1.5中加入了泛型,类型不正确将在编译期间知道,而不是在运行时导致异常错误
##### Item23 不要使用原始类型
例如当使用到集合的时候

```
//bad
private final Collection stamps = ... ;

//good
private final Collection<String> stamps = ... ;
```
泛型保证数据的安全性
##### Item24 清除未经检查的异常
在Eclipse或者Android studio中,敲写的代码或多或少有一些红色或黄色的警告,尽可能的消除这些警告

无法避免的话而且确保代码无误的话,慎重使用 @SuppressWarnings("unchecked")注解
##### Item25 集合和数组倾向集合
数组存储类型较灵活,将会导致不可预期的运行时异常
尽量让错误在编译时暴露而不是在不可控的运行时
##### Item26 泛型类
略
##### Item27 泛型方法
略
##### Item28 使用有界通配符增加API的灵活性
List<String>不是List<Object>的子类型,这个是有意义的
你给List<Object>中放入任何对象,但是List<String>中只能存放String    
但有时我们不想存放固定的类型,因此需要一些灵活性

```java
public class Stack<E> {
	public Stack();
	public void push(E e);
	public E pop();
	public boolean isEmpty();
}
//push所有元素
public void pushAll(Iterable<E> src) {
	for (E e : src)
	push(e);
}

//使用
Stack<Number> numberStack = new Stack<Number>();
Iterable<Integer> integers = ... ;
numberStack.pushAll(integers);
```
上述代码编译正常,看着也没什么问题 然而当我们添加的src不是我们期望的类型呢 如Stack<Number>,我们添加push(int) int将被自动装箱成Integer Integer也是Number的子类型
然而
```
//error
pushAll(Iterable<Number>) in Stack<Number>
cannot be applied to (Iterable<Integer>)
numberStack.pushAll(integers);
```
幸运的是Java提供了一种机制来解决子类的问题
```java
// Wildcard type for parameter that serves as an E producer
public void pushAll(Iterable<? extends E> src) {
  for (E e : src)
  push(e);
}
```
>经典原则:PECS stands for producer-extends, consumer-super.
##### Item29 多参数的泛型安全
略

---

[上一章:类和接口](https://www.jianshu.com/p/d8e8a6916325) 
[下一章:枚举和注解](https://www.jianshu.com/p/9cd61b89c79e) 
