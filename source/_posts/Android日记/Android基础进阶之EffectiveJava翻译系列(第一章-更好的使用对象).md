---
title: Android基础进阶之EffectiveJava翻译系列(第一章-更好的使用对象)
date: 2020-02-06 08:35:05
tags: 
categories: Android日记






---



#### 1.创建和销毁对象
这个章节包含创建和销毁对象,什么时候和怎样创建,什么时候避免创建,如何确保对象在准确的时机销毁,如何管理与清理销毁的对象
##### Item1 考虑用静态工厂方法替代构造方法
通常来说获取一个类的实例是通过它的构造方法,但这有一种更科学的方式:

```java
public static Boolean valueOf(boolean b){
  return b ? Boolean.TRUE : Boolean.FALSE;
}
```
>注意,静态工厂方法并不等价工厂设计模式

当然使用这种方式有利有弊
**利:
1.静态工厂方法有名字,可以一目了然 eg:BigPerson.name**
**2.在被调用的时候只创建一个对象**
**3.封装了实现细节,可返回任意需要的对象**
**4.减少对象的创建参数
eg:```java
Map<String,List<String>> m = new HashMap<String,List<String>>();
Map<String,List<String>> m = MyMap.newInstance();```**

弊:
1.无法从它派生出子类
继承的特性使用不上
2.无法和其它静态方法明显做出区分
推荐常用的几个
```
newInstance()
newType()
getInstance()
valueOf()
```
---
##### Item2 考虑用建造者模式创建多个参数的对象

```java
public class NutritionFacts {
  private final int servingSize;
  private final int servings;
  private final int calories;
  private final int fat;
  private final int sodium;
  private final int carbohydrate;

  public static class Builder {
    private final int servingSize;
    private final int servings;
    private final int calories = 0;
    private final int fat= 0;
    private final int sodium= 0;
    private final int carbohydrate= 0;

    public Builder(int ser,int servings){
      this.servingSize = ser;
      this.servings = servings;
    }
    public Builder calories(int val){calories = val;return this;}
    public Builder fat(int val){fat= val;return this;}
    public Builder sodium(int val){sodium= val;return this;}
    public Builder carbohydrate(int val){carbohydrate= val;return this;}
  }
    public NutritionFacts build(){return new NutritionFacts(this);}

    private NutritionFacts (Builder builder){
	    servingSize  = builder.servingSize ;
        servings = builder.servings ;
        calories = builder.calories ;
        fat= builder.fat;
        sodium= builder.sodium;
        carbohydrate= builder.carbohydrate;
}
}
```
用法
```java
NutritionFacts obj = new NutritionFacts.Builder(240,8).calories(100)
  .sodium(33).carbohydrate(22).build();
```
>比如典型的Dialog就是用的建造者模式
---
##### Item3 (强制)单例属性使用私有的构造方法或者使用枚举

```java
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();
  private Elvis () {...}
  public static Elvis getInstance() {return INSTANCE ;}

//枚举
private Object readRssolve() {
  return INSTANCE ;
}
public enum Elvis {
  INSTANCE ;
}
}
```
##### Item4 (强制)对于只有静态方法的工具类不允许使用私有构造方法

```java
public calss UtilityClass{
  private UtilityClass(){
    //error
  }
  public static void doSomething(){
    //...
  }
}
```
##### Item5 避免创建不必要的对象
重复利用一个对象好过重新创建一个新对象
重复利用访问更快而且更时尚,理论上说一个对象不可变可一直重复利用

```java
//bad DON'T DO THIS!
public class Person{
  private final Date birthDate;
  public boolean isBabyBoomer(){
    //分配了不必要的强度对象
    Calendar gmtCal = 
        Calendar.getInstance(TimeZone.getTimeZone("GMT"));
    gmtCal .set(1946,Calendar.JANUARY,1,0,0,0);
    Date boomStart = gmtCal .getTime();
    gmtCal .set(1965,Calendar.JANUARY,1,0,0,0);
    Date boomEnd = gmtCal .getTime();
    return birthDate.compareTo(boomStart) >= 0 
      && birthDate.compareTo(boomEnd) < 0;
  }

}
```
isBabyBoomer每次调用都会创建Calendar,TimeZone,和两个Date实例,优化后如下
```java
//good
class Person{
	private final Date birthDate;
	private static final Date BOOM_START;
	private static final Date BOOM_END:
	
	static {
		Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
		gmtCal.set(1946,Calendar.JANUARY,1,0,0,0);
		BOOM_START = gmtCal.getTime();
		gmtCal.set(1946,Calendar.JANUARY,1,0,0,0);
		BOOM_END = gmtCal.getTime();
	}
	
	public boolean isBabyBoomer(){
		return birthDate.compareTo(BOOM_START) >= 0 
          && birthDate.compareTo(BOOM_END) < 0;
	}
}
```

重复调用1000万次isBabyBoomer,使用bad-耗时32,000ms 使用good-耗时130ms,Wow~

---
##### Item6 清除过时的对象引用
Java有GC机制,并不意味着Java程序员不需要考虑内存了
你能找到如下代码内存泄漏之处吗?

```java
public class Stack {
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_SIZE = 16;
	
	public Stack(){
		elements = new Object[DEFAULT_SIZE];
	}
	
	public void push(Object e){
		ensureCapacity();
		elements[size++] = e;
	}
	
	public Object pop(){
		if(size == 0)
			throw new EmptyStackException();
		return elements[--size];
	}
	
	private void ensureCapacity(){
		if(elements.length == size)
			elements = Arrays.copyOf(elements,2 * size + 1);
	}
}
```
乍一看没什么问题吧?你也可以用你想到方式测试它,它都能运行良好 但这有一个内存泄漏的问题哦
嘟嘟嘟~
so,问题出在哪里呢?
嘟嘟嘟~





...


...

...


...


...


...


...


...
问题在于当我们pop了一些对象,GC并不知道这些对象已经过时,它只知道elements[]还处于活跃状态,所以一直不会清理那部分过时的对象
优化如下
```java
public Object pop(){
		if(size == 0)
			throw new EmptyStackException();
		Object result = elements[--size];
		elements[size] = null;//清理过时的对象
		return result;
	}
```
>类似的情况还有在集合中保存了对象,常常会忘记这些对象的存在

>对于监听的listener和callback也要及时注册和反注册,不然也会导致内存异常
##### Item7 避免使用finalizers
略
---

[下一章:Object的方法](https://www.jianshu.com/p/e9933b7e9009) 
