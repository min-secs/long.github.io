---
title: Android基础进阶之EffectiveJava翻译系列(第八章-异常)
date: 2021-02-06 08:35:05
tags:
categories: Android日记

---



高效使用异常指南
##### Item 57: Use exceptions only for exceptional conditions
只有在异常条件下才使用异常

考虑如下的代码

```Java
//bad !! don't do this
try {
	int i = 0;
	while(true)
		range[i++].climb();
} catch(ArrayIndexOutOfBoundsException e){}
```

使用异常条件来终止遍历操作是非常错误的做法

使用如下代码代替

```Java
for (Mountain m : range)
	m.climb();
```

>名副其实,异常只能用在异常条件下,而不能用于流程控制

---

##### Item 58: Use checked exceptions for recoverable conditions and runtime exceptions for programming errors
对可恢复条件使用检查异常，对程序错误使用运行时异常。

Java提供了三种异常:checked exceptions,runtime exceptions and errors.大多数程序员对何时使用何种异常有困惑,有如下几种原则作参考

决定使用检查异常或非检查异常的原则是:**调用方可以合理的修复异常**,现如今的IDE工具如eclipse或Android studio会自动提示此类异常,自动填充try catch

非检查异常有两种:runtime exceptions and errors

**用运行时异常(runtime exceptions)识程序错误**,绝大多数的运行时异常都表明违反了某种前提条件,如ArrayIndexOutOfBoundsException数组越界

有一个普遍的约定是error用于JVM,所以所有的非检查异常都应该继承自 RuntimeException 
>常见的非检查异常runtime exception
>- NullPointerException, 空指针异常 
>- ArithmeticException, 算术异常
>- ClassCastException, 类型强制转换异常
>- IllegalArgumentException, 传递非法参数异常

---

##### Item 59: Avoid unnecessary use of checked exceptions
避免过度使用检查异常
如果一个方法有多个检查异常,调用者会包裹多个catch来处理异常,这里没有一个绝对的准则,可以使用if语句把条件先过滤一遍而避免抛异常

---

##### Item 60: Favor the use of standard exceptions
使用常用异常
当需要抛出一个异常时,尽量使用Java平台提供好的异常,因为大家都知道什么意思

---

##### Item 61: Throw exceptions appropriate to the abstraction
抛出更合适的抽象异常

当抛出和任务不相关的异常时容易让人困惑,当抛出一个低级的异常时,这种情况经常发生,它不仅仅令人困惑,也污染了上层的调用

为了避免这个问题，上层应该捕获较低级别的异常，并在它们的位置上抛出可以用更高级别的抽象来解释的异常,如:

```Java
// Exception Translation
try {
	// Use lower-level abstraction to do our bidding
	...
} catch(LowerLevelException e) {
	throw new HigherLevelException(...);
}
```

更具体的例子在List<E>的方法中

```Java
/**
* Returns the element at the specified position in this list.
* @throws IndexOutOfBoundsException if the index is out of range
* ({@code index < 0 || index >= size()}).
*/
public E get(int index) {
	ListIterator<E> i = listIterator(index);
	try {
		return i.next();
	} catch(NoSuchElementException e) {
		throw new IndexOutOfBoundsException("Index: " + index);
	}
}
```

这种特殊的异常处理方式被称为"异常链",但是也不应该过度使用

我们也可以在在上层调用中抛出底层错误的原因

```Java
// Exception Chaining
try {
	... // Use lower-level abstraction to do our bidding
} catch (LowerLevelException cause) {
	throw new HigherLevelException(cause);
}
```

> 最好的方式还是尽量在底层避免异常,如果不能处理在考虑"异常链"的方式

---

##### Item 62: Document all exceptions thrown by each method
为每个方法抛出的异常添加文档注释

对于检查异常,要说明前置条件,并用@throws标记合适的异常,不要为了图简单直接 @throws Exception

如果一个异常被多个方法抛出,并且是相同的原因,则不要在方法上注释,而是要在类上添加异常注释

---

##### Item 63: Include failure-capture information in detail messages
打印出异常的详细信息以便于分析

---

##### Item 64: Strive for failure atomicity
保证错误的原子性(调用一千次,输出一样)

即使一个异常发生了,我们也希望对象能正常使用

有几种方式可以达到这一点,最简单的就是创建不可变的对象,如果一个对象是不可变的,原子性也随之而来

对于可变对象,常见的方式是检查参数的合法性(Item 38)

第三种方式是发生异常后,回滚异常状态为使用前的初始状态

最后一种方式使用拷贝来避免发生错误时改变原来对象的状态,如Collections.sort,排序方法会先转成array数组来排序,本来是为了提高性能,额外的如果发生了错误不会改变原集合的状态

虽然原子性能保证,但是实际使用中并不总是能达到满意的状态,如两个线程同时修改一个对象,没有同步的情况下,引发了currentModificationException.这时如果恢复对象的状态依然不能使程序正确运行

> 作为一种规则来讲,当发生异常时总能确保当前对象的状态,可惜目前很多API都没有遵守

---

##### Item 65: Don’t ignore exceptions
不要忽略异常

这条建议看似很明显,但是值得重复,当API的设计者声明了一种异常,他们是为了告诉你什么,**不要忽略它!**很容易包裹一个空的try catch语句就不管这个异常了.如:

```Java
try {
	...
} catch (SomeException e) {
}
```

**空处理并不是抛异常的初衷,目的是为了强制解决这个异常条件**.忽略异常就像忽略火警广播,有人关了广播导致其他人并不知道发生了火灾.如果确实需要一个空的处理,需要详细说明为何空处理是合适的

---
[上一章:通用原则](https://www.jianshu.com/p/b188135338f4)
