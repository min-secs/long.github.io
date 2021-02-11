---
title: Android基础进阶之EffectiveJava翻译系列(第五章-枚举与注解)
date: 2020-02-06 08:35:05
tags: 
categories: Android日记






---



Java1.5中提供的两种新类型

##### Item30: 用枚举替代int型常量
枚举:一系列常量类型的集合
没有枚举前大量定义的常量如下

```java
	// The int enum pattern - severely deficient!
	public static final int APPLE_FUJI = 0;
	public static final int APPLE_PIPPIN = 1;
	public static final int APPLE_GRANNY_SMITH = 2;
	public static final int ORANGE_NAVEL = 0;
	public static final int ORANGE_TEMPLE = 1;
	public static final int ORANGE_BLOOD = 2;

```
首先调试不方便,我们只会打印出数字,然后回归代码
其次没有命名空间做区别,命名累赘
使用枚举后
```java
	public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
	public enum Orange { NAVEL, TEMPLE, BLOOD }

```
再来看一个星球的例子
```java
public enum Planet {
	MERCURY(3.302e+23, 2.439e6),
	VENUS (4.869e+24, 6.052e6),
	EARTH (5.975e+24, 6.378e6),
	MARS (6.419e+23, 3.393e6),
	JUPITER(1.899e+27, 7.149e7),
	SATURN (5.685e+26, 6.027e7),
	URANUS (8.683e+25, 2.556e7),
	NEPTUNE(1.024e+26, 2.477e7);
	private final double mass; // In kilograms
	private final double radius; // In meters
	private final double surfaceGravity; // In m / s^2
	// Universal gravitational constant in m^3 / kg s^2
	private static final double G = 6.67300E-11;
	// Constructor
	Planet(double mass, double radius) {
		this.mass = mass;
		this.radius = radius;
		surfaceGravity = G * mass / (radius * radius);
	}
	public double mass() { return mass; }
	public double radius() { return radius; }
	public double surfaceGravity() { return surfaceGravity; }
	public double surfaceWeight(double mass) {
		return mass * surfaceGravity; // F = ma
	}
}
```
再来看一个算数的例子
```java
public enum Operation {
	PLUS { double apply(double x, double y){return x + y;} },
	MINUS { double apply(double x, double y){return x - y;} },
	TIMES { double apply(double x, double y){return x * y;} },
	DIVIDE { double apply(double x, double y){return x / y;} };
	abstract double apply(double x, double y);
}
```
>总结:枚举比int型常量更安全和具有可读性



##### Item31: Use instance fields instead of ordinals(使用实例字段代替序号)

所有的枚举都有
ordinal 方法(从0开始计数),如果我们想用序号的int值(从1开始计数),不要直接修改ordinal 方法
```java
//bad
public enum Ensemble {
	SOLO, DUET, TRIO, QUARTET, QUINTET,
	SEXTET, SEPTET, OCTET, NONET, DECTET;
	public int numberOfMusicians() { return ordinal() + 1; }
}
```
虽然上述代码能正常使用,但是却会导致一场噩梦
如果我们改变了枚举中的常量顺序,之前的序号就会一团糟
优化代码如下
```java
public enum Ensemble {
	SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
	SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
	NONET(9), DECTET(10), TRIPLE_QUARTET(12);
	
	private final int numberOfMusicians;
	
	Ensemble(int size) { 
		this.numberOfMusicians = size; 
	}
	
	public int numberOfMusicians() { 
		return numberOfMusicians; 
	}
}
```
##### Item32: Use EnumSet instead of bit fields(用EnumSet替代位字段)
如我们有一段代码

```java
// Bit field enumeration constants - OBSOLETE!
public class Text {
  public static final int STYLE_BOLD = 1 << 0; // 1
  public static final int STYLE_ITALIC = 1 << 1; // 2
  public static final int STYLE_UNDERLINE = 1 << 2; // 4
  public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8
  // Parameter is bitwise OR of zero or more STYLE_ constants
  public void applyStyles(int styles) { ... }
}
  //使用
  text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```
用EnumSet优化如下:
```java
// EnumSet - a modern replacement for bit fields
public class Text {
	public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
	// Any Set could be passed in, but EnumSet is clearly best
	public void applyStyles(Set<Style> styles) { ... }
}

  //使用
  text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```
##### Item33: Use EnumMap instead of ordinal indexing(使用EnumMap代替顺序索引)
[EnumSet/EnumMap详解](https://developer.android.google.cn/reference/java/util/EnumSet.html#of(E,%20E))
##### Item34: 使用接口提升枚举的扩展性
code say everything
普通操作 加减乘除

```java
// Emulated extensible enum using an interface
public interface Operation {
	double apply(double x, double y);
}
public enum BasicOperation implements Operation {
	PLUS("+") {
		public double apply(double x, double y) { return x + y; }
	},
	MINUS("-") {
		public double apply(double x, double y) { return x - y; }
	},
	TIMES("*") {
		public double apply(double x, double y) { return x * y; }
	},
	DIVIDE("/") {
		public double apply(double x, double y) { return x / y; }
	};

	private final String symbol;
	BasicOperation(String symbol) {
		this.symbol = symbol;
	}
	@Override public String toString() {
		return symbol;
	}
}
```
现需要扩展取余,异或等方法,不要再BasicOperation 中新加枚举,而是实现公共接口Operation 
```java
// Emulated extension enum
public enum ExtendedOperation implements Operation {
	EXP("^") {
		public double apply(double x, double y) {
			return Math.pow(x, y);
		}
	},
	REMAINDER("%") {
		public double apply(double x, double y) {
			return x % y;
		}
	};
 
	private final String symbol;
	ExtendedOperation(String symbol) {
		this.symbol = symbol;
	}
	@Override public String toString() {
		return symbol;
	}
}
```
##### Item36: 始终使用override注解

```java
// 你能找到这个bug吗?
public class Bigram {
	private final char first;
	private final char second;
	public Bigram(char first, char second) {
		this.first = first;
		this.second = second;
	}
	public boolean equals(Bigram b) {
		return b.first == first && b.second == second;
	}
	public int hashCode() {
		return 31 * first + second;
	}
	
	public static void main(String[] args) {
		Set<Bigram> s = new HashSet<Bigram>();
		for (int i = 0; i < 10; i++)
			for (char ch = 'a'; ch <= 'z'; ch++)
			  s.add(new Bigram(ch, ch));
		System.out.println(s.size());
	}
}
```
理论上我们希望打印26,因为HashSet没有重复元素,但是得到的值是260
因为没有override equals&hashCode方法,所以默认是使用的"=="比较的地址,得到的都是不同的对象
修复如下
```java
    @Override public boolean equals(Bigram b) {
		return b.first == first && b.second == second;
	}
	@Override public int hashCode() {
		return 31 * first + second;
	}
```
##### Item37: 使用标记接口定义类型(待补充)
- 标记接口:只为了标记某一种类型
如 Serializable 接口,表明可将实例序列化
优点:
1.标记接口定义了一种类型
2.可用于任何扩展的子类

- 标记注解:
优点:
1.描述信息更丰富,可在使用后添加方法
2.位于FrameWork层,各个应用使用同一套规则
  
---

[上一章:泛型](https://www.jianshu.com/p/dd9c5d7cdaa5)        
[下一章:方法](https://www.jianshu.com/p/6bca1fc1945a)
