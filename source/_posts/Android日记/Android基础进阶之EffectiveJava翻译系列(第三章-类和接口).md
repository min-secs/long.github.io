---
title: Android基础进阶之EffectiveJava翻译系列(第三章-类和接口)
date: 2020-02-06 08:35:05
tags: 
categories: Android日记






---



#### 3. 类和接口

>类和接口是Java编程的核心
#####Item13 最小化类和成员变量的访问权限
信息隐藏与封装是程序设计的基本原则
通用经验是能让类或者变量不可访问就让它不可访问
四种访问权限:
- private
只在它声明的地方可用
- package-private
同一个包内可用
- protected
同一个包或子类可用
- public
任何地方可用
>公共类不应包含公共字段。确保static final引用的对象是不可变的。
绝不可以用private static final来声明一个数组对象

##### Item14 用公共方法提供公共字段

```java
//bad
public class Point {
  public int x;
  public int y;
}

//good
class Point {
	private double x;
	private double y;
	public Point(double x, double y) {
		this.x = x;
		this.y = y;
	}
	public double getX() { return x; }
	public double getY() { return y; }
	public void setX(double x) { this.x = x; }
	public void setY(double y) { this.y = y; }
}
```
##### Item15 最小化可变性
对象始终不变的类比可变的更安全和好用,如String
我们设计不可变对象时应遵循如下准则:

- 不要提供任何方法来改变对象的状态
- 确保类不可继承
- 所有的字段使用private final修饰
- 确保不要让类中的可变对象访问,使用深拷贝替代
如果一个类确实可变,也要保证其它地方尽可能不变,TimerTask是很好的例子
#####Item16 组合和继承,优先使用组合
>继承虽然有很多好处,但是违背了封装性,暴露了父类的实现细节
#####Item17 专用于继承的设计，否则禁止继承
>未能理解,后续补充  
#####Item18 接口和抽象类之间倾向于接口
Java提供了两种机制来提供多个类型定义的实现:接口和抽象类

最明显的区别在于抽象类允许某些方法的实现,更重要的区别在于一个类使用了抽象类的方式来定义,则这个类必须是抽象类的子类.因为Java只允许单一继承，因此对抽象类的这种限制严重影响了它们作为类型定义的使用 

- 已经存在的类可以很容易的实现一个新的接口
比如实现Comparable接口
- 接口是定义混合器的理想选择
比如为一个主要类型的类添加比较方法,可通过实现Comparable接口实现实例之间的排序

有一个特例是抽象类比接口更加易用,如果注重程序的易用性而不是灵活性则可以考虑抽象类
#####Item19 仅使用接口定义类型
当一个类实现了一个接口,这个接口是为类的实例服务的而不是为了其他目的

所以有一种常量接口尽量不要使用
```java
//bad
public interface PhysicalConstants {
	// Avogadro's number (1/mol)
	static final double AVOGADROS_NUMBER = 6.02214199e23;
	// Boltzmann constant (J/K)
	static final double BOLTZMANN_CONSTANT = 1.3806503e-23;
	// Mass of the electron (kg)
	static final double ELECTRON_MASS = 9.10938188e-31;
}
```
如果有这种常量需要使用,可以把它定义在与之关联的类里面
##### Item20 用继承来替代复合类(原单词: tagged classes)
偶尔会遇到一个类的实例包含了多种不同的类别,如下

```java
// bad Tagged class - vastly inferior to a class hierarchy!
class Figure {
	enum Shape { RECTANGLE, CIRCLE };
	// Tag field - the shape of this figure
	final Shape shape;
	// These fields are used only if shape is RECTANGLE
	double length;
	double width;
	// This field is used only if shape is CIRCLE
	double radius;
	// Constructor for circle
	Figure(double radius) {
		shape = Shape.CIRCLE;
		this.radius = radius;
	}
	// Constructor for rectangle
	Figure(double length, double width) {
		shape = Shape.RECTANGLE;
		this.length = length;
		this.width = width;
	}
	double area() {
		switch(shape) {
		case RECTANGLE:
			return length * width;
		case CIRCLE:
			return Math.PI * (radius * radius);
		default:
			throw new AssertionError();
		}
	}
}  
```
优化如下
```java
//Class hierarchy replacement for a tagged class
abstract class Figure {
	abstract double area();
}
class Circle extends Figure {
	final double radius;
	Circle(double radius) { this.radius = radius; }
	double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
	final double length;
	final double width;
	Rectangle(double length, double width) {
		this.length = length;
		this.width = width;
	}
	double area() { return length * width; }
} 
```
##### Item21  使用函数对象表示策略
直译过来有点不好理解,大意是对于多个类都要使用的策略(或用方法表示),采用接口来实现

如一个类需要比较方法,另一个类也需要,则抽象出一个Comparable接口来给各个类通用
##### Item22 嵌套类的使用
有四种嵌套类,分别是静态成员类、非静态成员类、匿名内部类和本地类
除了第一个都是内部类

- 静态成员类
可以访问外部类的私有属性
- 非静态成员类
同静态成员类,区别在于没有static修饰
```java
public class MySet<E> extends AbstractSet<E> {
	public Iterator<E> iterator() {
		return new MyIterator();
	}
    //典型用法
	private class MyIterator implements Iterator<E> {
	...
	}
}
```
- 匿名内部类
不用声明,常用于监听器,随用随销
- 本地类
略 未能理解,后续补充,欢迎留言增益知识
>如果成员类需要外部类的引用,则使用非静态成员类,否则使用静态成员类

---

[上一章:Object的方法](https://www.jianshu.com/p/e9933b7e9009) 
[下一章:泛型](https://www.jianshu.com/p/dd9c5d7cdaa5) 
