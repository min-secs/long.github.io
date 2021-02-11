---
title: Android基础进阶之EffectiveJava翻译系列(第七章-通用原则)
date: 2020-02-06 08:35:05
tags: 
categories: Android日记






---



本章主要讨论语言的具体内容。它讨论了局部变量的处理、控制结构、库的使用、各种数据类型的使用，以及使用反射和本地方法。最后，讨论了优化和命名约定

##### Item 45:最小化局部变量作用域

>作用域:一个花括号{}包裹起来的区域

此条例同Item13相似:最小化类和成员变量的访问权限
Java允许你在任何地方声明变量,但是最重要的是*在首次使用的地方声明变量,并初始化*
循环提供了一种实现此种方式的机制,而且for循环比while循环好,如
```java
for (Element e : c) {
	doSomething(e);
}

//before JDK1.5
for (Iterator i = c.iterator(); i.hasNext(); ) {
	doSomething((Element) i.next());
}
```
为什么for比while好呢?
```java
//bad
Iterator<Element> i = c.iterator();
while (i.hasNext()) {
	doSomething(i.next());
}
...
Iterator<Element> i2 = c2.iterator();
while (i.hasNext()) { // BUG! 应该是i2
	doSomethingElse(i2.next());
}
```
当我们写一个差不多的代码,从一个地方copy过来的时候,很有可能忘记修改某个变量值(如i2),它不会在编译期报错,我们很可能长时间遗留这个bug
使用for循环可以避免这个bug
```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
	doSomething(i.next());
}
 ...
// Compile-time error - cannot find symbol i
for (Iterator<Element> i2 = c2.iterator(); i.hasNext(); ) {
	doSomething(i2.next());
}
```
这就是最小化作用域的好处
#####Item 46: Prefer for-each loops to traditional for loops
对于不需要下标来做特殊操作的遍历,推荐使用增强for循环
你能发现下面的bug吗?
```java
enum Suit { CLUB, DIAMOND, HEART, SPADE }
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
 NINE, TEN, JACK, QUEEN, KING }
...
Collection<Suit> suits = Arrays.asList(Suit.values());
Collection<Rank> ranks = Arrays.asList(Rank.values());
List<Card> deck = new ArrayList<Card>();
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
	for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
		deck.add(new Card(i.next(), j.next()));
```
...
...
...
...
...
...
发现不了也不要难过,很多有经验的程序员也会犯这个错误
原因在于i.next()会被重复调用,导致结果异常
可以修复如下
```java

for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ){
    Suit suit = i.next();
	for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
		deck.add(new Card(suit, j.next()));
}
```
虽然解决了问题,但是很丑,更简洁的写法如下
```java
for (Suit suit : suits)
	for (Rank rank : ranks)
		deck.add(new Card(suit, rank));
```
>尽可能的使用增强for循环
#####Item 47: Know and use the libraries
不要重复造轮子
如果需要一个常用的功能,去发现一个库并使用它,往往比自己写的要好
因为库会随着时间推移更新迭代越来越好,自己也节约了时间
这并不会评论一个人的能力
#####Item 48: Avoid float and double if exact answers are required
>当需要一个精确的答案时,避免使用float或double类型的变量

float或double是为了科学和工程计算而设计的,特别不适用于货币计算

推荐使用int或long代替
#####Item 49: Prefer primitive types to boxed primitives
>使用原始类型替代装箱类型

byte short char int long float double boolbean等是Java中的基本类型,也有对应的装箱类型,如 Integer, Double, and Boolean.应当谨慎对待这两者之间的差别

首先第一个差别,原始类型仅仅包含对应的值,装箱类型既包含对应的值也有对应的引用,第二个差别是装箱类型有可能为null,第三个差别是原始类型在时间和空间消耗中更高效
#####Item 50: Avoid strings where other types are more appropriate
>如果有更合适的类型,避免使用String

string被设计成描述文本类型的数据,而且干得很好,本章主要讨论将string用于其它情况的错误用法

**string不能替代值类型** 如果我们正在等待键盘的输入,或者从网络获取某个值,我们很方便的使用string作为接收类型,但是如果我们输入的是数字或者真假值的话,对应的int或boolean能更好的标识输入 虽然这条规则很明显,但是经常被违反

**string不能替代枚举** 如[Item30:枚举](https://www.jianshu.com/p/9cd61b89c79e)讨论的那样,对于静态常量使用枚举

**string不能替代聚合字符** 

```java
String compoundKey = className + "#" + i.next();//bad
```



我们经常写上述代码,这样的写法有很多缺点.如果我们想使用某一部分字段需要解析字符串,很耗时而且容易出错.String提供的equals,compareTo等方法也不能使用

比较好的方式是写一个静态内部类来表示聚合字符

```Java
static class CompoundKey{
    private String className;
    private String next;
    ...
}
```


#####Item 51: Beware the performance of string concatenation
> 考虑字符连接(+)的性能 

使用(+)能很方便的拼接若干个字符串,但是我们也要考虑到开销

如下两个代码都是拼接字符

```Java
//方式一
public String statement() {
	String result = "";
	for (int i = 0; i < numItems(); i++)
		result += lineForItem(i); // String concatenation
	return result;
}

//方式二
public String statement() {
	StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
	for (int i = 0; i < numItems(); i++)
		b.append(lineForItem(i));
	return b.toString();
}
```

当numItems()==100;lineForItem(i)返回80个长度的字符时,在作者的机器上方式二比方式一快85倍

如果我们需要拼接大量字符时,使用StringBuilder代替
#####Item 52: Refer to objects by their interfaces
> 使用接口代替对象引用

如果有一个合适的接口来描述当前类的时候使用这个接口来引用,如

```Java
// Good - uses interface as type
List<Subscriber> subscribers = new Vector<Subscriber>();

// Bad - uses class as type!
Vector<Subscriber> subscribers = new Vector<Subscriber>();
```

如果你养成了这个习惯,那么你的代码将会更灵活
有几种情况不能使用接口
1.没有合适的接口声明
2.接口中没有想要的某个方法
3.使用的类集成自framework,并且是一个抽象类
#####Item 53: Prefer interfaces to reflection
使用接口代替反射

反射核心类java.lang.reflect提供了对加载类信息的访问

例如，Method.Invoke允许在任何类的任何对象上调用任何方法(受通常的安全约束)。 即使编译后的类不存在，也允许一个类使用另一个类。然而，这种力量是有代价的。

- **你不能在编译时检查出异常** 如果你使用了对应的反射操作,在发生异常时,只有在运行时才能发现

- **阅读性极差**

- **性能有影响** 使用反射比普通调用要慢些,因为受很多因素的影响,慢多少很难说,在作者的机器上,速度差在两倍到五十倍不止

  

反射核心用在基于组件设计的应用,为了按需加载类,使用反射找到对应的类构造与否;普通应用尽量不要使用反射,找到代替的接口或者父类对象
#####Item 54: Use native methods judiciously 
明智地使用本地方法

JNI允许Java调用C或C++写的本地方法

从历史上看，本地方法有三种主要用途。

1.它们提供了对特定于平台的设施的访问，例如注册表和文件锁。

2.他们提供了对旧代码库(Java想使用历史上C或C++写的库)

的访问，可以反过来提供对旧数据的访问。

3.使用本地方法用本地语言编写应用程序关键部分，以提高性能。

Java平台在不断发展,访问特定平台设施,使用Java提供的工具类就可做到,而且也不建议使用本地方法来提升程序性能

使用本地方法有严重的缺点

1.本地语言是不安全的,会受机器内存错误的影响

2.依赖于平台,不便于移植

3.本地代码很难调式

4.访问本地代码开销很大

5.本地代码不宜阅读

> 总之,要再三思考是否使用本地代码,如果需要使用以前的代码库,请尽可能减少本地代码片段并加强测试,很小很小的本地代码错误将破坏你的整个程序
#####Item 55: Optimize judiciously
明智的优化

有三个人人都应该知道的优化格言

>More computing sins are committed in the name of efficiency (without necessarily achieving it) than for any other single reason—including blind stupidity.
—William A. Wulf [Wulf72]
更多的计算罪恶是以效率的名义犯下的(不一定要达到这一目的)，而不是因为任何其他单一的原因-包括盲目的愚蠢

​							
>We should forget about small efficiencies, say about 97% of the time: premature optimization is the root of all evil.
—Donald E. Knuth [Knuth74]
我们应该忽略小效率,大约百分之97的情况下,邪恶之源就是过早优化

​							
>We follow two rules in the matter of optimization:
Rule 1. Don’t do it.
Rule 2 (for experts only). Don’t do it yet—that is, not until you have a
perfectly clear and unoptimized solution.
—M. A. Jackson [Jackson75]
关于优化遵循如下两个原则:
原则1.别这样做
原则2.(仅对专家而言).还不要做,直到你找到一个清晰的解决方案或者一个未优化的解决办法

不要为了性能而损坏架构,**致力于写出好的程序而不是快的程序**,如果一个好的程序还不够快,它的架构会允许优化.

这并不意味着当你的程序完后不需要优化,你应该在设计阶段就考虑到性能

**尽量避免影响性能的设计**,已经实现好的组件很难在改变,尤其是API,数据结构,多方约定好的协议

**考虑好API使用效果**,设计一个可变的类可能会导致后续使用中出现过多的深拷贝,造成对象分配的额外开销

API的设计对性能有很真实的影响,如java.awt.Component中的getSize()方法,每调用一次就会返回一个新的 Dimension 实例(JDK1.2版本已经修复),虽然分配一个实例的开销很小,但是成百上千次调用也会对程序有严重影响

幸运的是,好的API设计自然带来了好的性能

> 总之,致力于写出好的程序,快随之而来 当做出一部分改变后就要测量代码的性能,对于Android来说内存,卡顿,anr等方面

参考[DDMS性能调优](https://www.jianshu.com/p/829ed0e6010e)



---

​							
#####Item 56: Adhere to generally accepted naming conventions 
遵循公共的命名规范,参考阿里巴巴发布的Android手册

---

[上一章:方法](https://www.jianshu.com/p/6bca1fc1945a)  
[下一章:异常](https://www.jianshu.com/p/009327a97ae9)
