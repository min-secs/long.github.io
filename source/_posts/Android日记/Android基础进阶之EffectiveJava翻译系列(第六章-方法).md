---
title: Android基础进阶之EffectiveJava翻译系列(第六章-方法)
date: 2020-02-06 08:35:05
tags: 
categories: Android日记






---



这一章介绍方法设计的几个方面:如何对待参数和返回值,如何设计方法签名,如何注释方法

##### Item38: 检查参数的合法性
大部分使用的方法参数都有一定的限制,如不为null,size>0等
通用的原则就是预防大于整改,提前发现错误可以更快的规避问题,而不是在程序运行中发生

对于公共方法，使用Javadoc@块标记,来记录在违反参数值限制时抛出的异常(Item62)。
```java
/**
* Returns a BigInteger whose value is (this mod m). This method
* differs from the remainder method in that it always returns a
* non-negative BigInteger.
*
* @param m the modulus, which must be positive
* @return this mod m
* @throws ArithmeticException if m is less than or equal to 0
*/
public BigInteger mod(BigInteger m) {
	if (m.signum() <= 0)
		throw new ArithmeticException("Modulus <= 0: " + m);
	... // Do the computation
}
```
对于私有的方法则使用断言
```java
// Private helper function for a recursive sort
private static void sort(long a[], int offset, int length) {
	assert a != null;
	assert offset >= 0 && offset <= a.length;
	assert length >= 0 && length <= a.length - offset;
	... // Do the computation
}
```
assert在实际项目中使用的很少,更多的还是使用的if判断
>每次写一个方法或者构造函数的时候,在方法的开头考虑参数的合法性是必不可少的
#####Item39:  必要的时候使用拷贝对象
 Java是一门安全的语言,即使在Java语言中,也要假设用户正想方设法的破坏你的程序.除了少部分人想破坏系统的安全性,大部分问题都是编程人员可以控制的
虽然没有对象的帮助,另一个类不太可能改变对象的内部状态,但偶尔也有疏忽的地方
```java
//一个不可变的周期类
// Broken "immutable" time period class
public final class Period {
	private final Date start;
	private final Date end;
	/**
	* @param start the beginning of the period
	* @param end the end of the period; must not precede start
	* @throws IllegalArgumentException if start is after end
	* @throws NullPointerException if start or end is null
	*/
	public Period(Date start, Date end) {
		if (start.compareTo(end) > 0)
			throw new IllegalArgumentException(
				start + " after " + end);
		this.start = start;
		this.end = end;
	}
	public Date start() {
		return start;
	}

	public Date end() {
		return end;
	}
	... // Remainder omitted
}
```
其中Date对象是可变的
```java
// bad
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // 修改了p的内部状态
```
为了保护对象的状态,我们需要在构造函数中对可变参数执行拷贝防御
```java
// Repaired constructor - makes defensive copies of parameters
public Period(Date start, Date end) {
	this.start = new Date(start.getTime());
	this.end = new Date(end.getTime());
	if (this.start.compareTo(this.end) > 0)
		throw new IllegalArgumentException(start +" after "+ end);
}

// Repaired accessors - make defensive copies of internal fields
public Date start() {
	return new Date(start.getTime());
}
public Date end() {
	return new Date(end.getTime());
}
```
>这里没有使用clone方法,因为Date有其它不可信任的子类

经验上讲,在内部尽量不要使用可变的类,如Date,可用long型替代Date.getTime()
>总结:如果一个类调用了或返回可变的对象,则需要用拷贝对象防御,如果很信任调用者不会修改类的内部状态,则需要有一份警告文档提示调用者不能修改类的状态
#####Item 40: 如何设计方法
- 选择一个合适的方法名称
你的主要目标是设计一个利于理解的方法名,次要目标是方法名称之间保持协调性,如向数据库中插入一条数据,有的使用addXX,有的使用setXX,有的使用insertXX,尽量保持统一
- 不要过分的使用方法
太多的方法容易使一个类难于维护和测试,只要当它需要经常调用的时候才考虑提出一个方法,否则就不管它
- 避免参数长的方法
尽量保持在四个参数或以下
有三种方式避免长参数
1.提出更多的方法
2.使用辅助类保存这些参数
3.使用建造者模式(Builder)
- 对于传入的参数,有接口可以传就使用接口
如需要传入HashMap 则在方法中将参数类型改为Map 避免使用者只能使用HashMap,也可以传入其它Map接口的子类型
- 对于布尔型参数,使用枚举更合适
例如，您可能有一个带有静态工厂的温度计类型( Thermometer 类)，其值为枚举：
```java
public enum TemperatureScale { FAHRENHEIT, CELSIUS }
```
Thermometer.newInstance(TemperatureScale.CELSIUS)不仅比Thermometer.newInstance(true)更有意义，而且可以在将来的发行版中将Kelvin添加到TemperatureScale，而不需要 在Thermometer 类中增加一个新的静态工厂方法
##### Item41: 谨慎地使用重载
看如下的例子,我们想区分放进去的是List或者set或者不知道什么类型的集合,想一下它会如何打印

```java
public class CollectionClassifier {
public static String classify(Set<?> s) {
	return "Set";
}
public static String classify(List<?> lst) {
	return "List";
}
public static String classify(Collection<?> c) {
	return "Unknown Collection";
}
public static void main(String[] args) {
	Collection<?>[] collections = {
		new HashSet<String>(),
		new ArrayList<BigInteger>(),
		new HashMap<String, String>().values()
	};
	for (Collection<?> c : collections)
	System.out.println(classify(c));
 }
}
```
它会顺序打印"Set","List","Unknown Collection"吗?不,它会打印三次"Unknown Collection" 原因在于重载方法是在编译时执行,所以会以Collection<?>为准
>我们应该避免使用相同参数数量的重载方法,使机器不懂,自己更易混淆
#####Item 42: 谨慎的使用可变参数
举一个例子
```java
static int sum(int... args) {
	int sum = 0;
	for (int arg : args)
		sum += arg;
	return sum;
}
```
sum(1,2,3) --> 6
sum() --> 0
>暂略
#####Item 43: Return empty arrays or collections, not nulls
像如下的代码很常见
```java
//bad
private final List<Cheese> cheesesInStock = ...;
/**
 * @return an array containing all of the cheeses in the shop,
 * or null if no cheeses are available for purchase.
 */
public Cheese[] getCheeses() {
	if (cheesesInStock.size() == 0)
		return null;
	...
}	
```
调用者很可能粗心大意忘记判null导致异常,也许很多年之后才会发现
有人说返回null避免了内存开销,首先你要证明是这段代码导致的性能问题,其次我们可以使用不可变的静态常量声明一个空集合
```java
// The right way to return an array from a collection
private final List<Cheese> cheesesInStock = ...;
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];
/**
* @return an array containing all of the cheeses in the shop.
*/
public Cheese[] getCheeses() {
	if(cheesesInStock.size() <= 0)
		return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}

//or

public List<Cheese> getCheeseList() {
 if (cheesesInStock.isEmpty())
	return Collections.emptyList(); // Always returns same list
 else
	return new ArrayList<Cheese>(cheesesInStock);
}
```
总之,使用集合或数组的任何情况下都不能返回null
##### Item 44: Write doc comments for all exposed API elements
为所有暴露出去的API写文档注释

---

[上一章:枚举和注解](https://www.jianshu.com/p/9cd61b89c79e)   
[下一章:通用原则](https://www.jianshu.com/p/b188135338f4)          
