---
title: Android基础进阶之EffectiveJava翻译系列(第九章-并发)
date: 2020-02-06 08:35:05
tags: 
categories: Android日记





---



	线程允许多个任务同时执行.并发编程比单线程难,因为很多事情一起处理容易出错,也很难减少错误,但是你不能避免并发.这章帮助你编写简洁的,正确的,良好阅读性的并发编程

##### Item66 同步共享的可变数据

​	 synchronized 关键字可以保证一次只有一个线程访问代码块,许多开发者认为同步就是一种互斥,防止对象在另一个线程修改时处于不一致的状态.在这种观点中,对象处于一种正确的状态,因为访问它的方法锁住了.这些方法确保对象的状态由一种状态安全的转移到另一种状态.

​	这种观点只正确了一半,不同步的话,一个线程的改变对其它线程是不可见的.通过相同的锁,同步不仅阻止线程在不一致状态下观察对象，而且确保每个进入同步方法或块的线程都能看到所有一致性的效果.

​	考虑一下从一个线程停止另一个线程,Java lib提供了Thread.stop方法,但是这个方法被遗弃了,因为它是不安全的---将导致数据损坏.一种建议方法是获取到第一个线程的boolean变量,一些人可能会这么写:

```Java
//bad
public class StopThread {
private static boolean stopRequested;
public static void main(String[] args)
	throws InterruptedException {
		Thread backgroundThread = new Thread(new Runnable() {
			public void run() {
			int i = 0;
			while (!stopRequested)
				i++;
			}
		});
		backgroundThread.start();
		TimeUnit.SECONDS.sleep(1);
		stopRequested = true;
		}
}
```



​	你可能期望这个程序运行大约一秒，然后主线程设置stopRequested为true，从而导致后台线程的循环终止 .然而在我的机器上，程序永远不会停：子线程永远在循环！

​	问题在于，在没有同步的情况下，无法保证后台线程何时会看到主进程所做的修改. 在没有同步的情况下，虚拟机转换成以下代码：

```java
while (!done)
	i++;
//转换
if (!done)
	while (true)
		i++;
```

修复方式如下:

```Java
//good
public class StopThread {
	private static boolean stopRequested;
	private static synchronized void requestStop() {
		stopRequested = true;
	}
	private static synchronized boolean stopRequested() {
		return stopRequested;
	}
public static void main(String[] args)
	throws InterruptedException {
		Thread backgroundThread = new Thread(new Runnable() {
			public void run() {
				int i = 0;
				while (!stopRequested())
					i++;
			}
		});
		backgroundThread.start();
		TimeUnit.SECONDS.sleep(1);
		requestStop();
	}
} 
```

​	**注意:读和写都是同步的,光对写方法同步,同步会失效**

还可以使用volatile关键字修复为:

```Java
public class StopThread {
	private static volatile boolean stopRequested;
	public static void main(String[] args)
		throws InterruptedException {
			Thread backgroundThread = new Thread(new Runnable() {
				public void run() {
					int i = 0;
					while (!stopRequested)
						i++;
				}
			});
			backgroundThread.start();
			TimeUnit.SECONDS.sleep(1);
			stopRequested = true;
	}
}
```

但使用volatile关键字要小心,考虑如下代码

```Java
//bad
private static volatile int nextSerialNumber = 0;
public static int generateSerialNumber() {
	return nextSerialNumber++;
} 
```

乍看之下没有什么问题,但是"++"操作不是原子性的,包含了两个操作,一个是读旧值,另一个是在旧值的基础上加一,在赋值.修复方式为加上synchronized关键字:

```Java
//good
private static volatile int nextSerialNumber = 0;
public static synchronized int generateSerialNumber() {
	return nextSerialNumber++;
} 
```

​	避免此类问题最好的方式是不要共享可变数据.要么共享不可变的数据,要么就不共享.换句话说,**在一个线程中定义可变数据.**如果采用此策略，则必须将其文档化，以便程序维护此原则

> 总之,当多个线程共享数据时，读取或写入数据的每个线程都必须执行同步.没有同步,无法保证一个线程的修改对另一个线程可见.这将会导致程序安全问题,而且很难调试.如果你只需要内部间的线程通信而不考虑互斥, volatile 关键字可以替代synchronized,但是volatile很难被正确使用

---

##### Item 67: 避免过度同步

Item66警示了不使用同步的危险性,Item 67讨论完全相反的一面.在某种情况下,过度使用同步会导致性能下降,死锁或者不可预期的行为

为了避免重复和安全故障，千万不要在同步方法或块内控制客户端.换句话说,不要再同步方法中调用需要重写的方法或者从客户端提供的对象方法(这种方法被称为"外星人").因为同步块不知道这个方法是干什么的也不能控制这个客户端,调用它将会导致异常或者数据损坏

考虑如下的"外星人"代码

```Java
//bad Broken - invokes alien method from synchronized block!
public class ObservableSet<E> extends ForwardingSet<E> {
  public ObservableSet(Set<E> set) { super(set); }
  private final List<SetObserver<E>> observers = 
                    new ArrayList<SetObserver<E>>();
  public void addObserver(SetObserver<E> observer) {
  	  synchronized(observers) {
		  observers.add(observer);
	  }
  }
  public boolean removeObserver(SetObserver<E> observer) {
	  synchronized(observers) {
		  return observers.remove(observer);
	  }
  }
  private void notifyElementAdded(E element) {
	  synchronized(observers) {
		for (SetObserver<E> observer : observers)
			observer.added(this, element);
	  }
  }
@Override public boolean add(E element) {
	boolean added = super.add(element);
	if (added)
		notifyElementAdded(element);
	return added;
}
@Override public boolean addAll(Collection<? extends E> c) {
	boolean result = false;
	for (E element : c)
		result |= add(element); 
	return result;
	}
}
```

observers通过addObserver/removeObserver 订阅/取消订阅,但是SetObserver<E> observer被传递进来了:

```Java
public interface SetObserver<E> {
	// Invoked when an element is added to the observable set
	void added(ObservableSet<E> set, E element);
}
```



在测试运行时,上述代码似乎运行良好,如打印0~99:

```Java
public static void main(String[] args) {
	ObservableSet<Integer> set =
		new ObservableSet<Integer>(new HashSet<Integer>());
	set.addObserver(new SetObserver<Integer>() {
		public void added(ObservableSet<Integer> s, Integer e) {
		System.out.println(e)
	}});
	for (int i = 0; i < 100; i++)
		set.add(i);
}
```

现在我们来做一点改变:

```Java
set.addObserver(new SetObserver<Integer>() {
	public void added(ObservableSet<Integer> s, Integer e) {
		System.out.println(e);
		if (e == 23) s.removeObserver(this);
	}
});
```

我们会期望打印出0-23,实际上会发生什么呢,打印出0~23后接着会报ConcurrentModificationException.因为我们正在移除集合中的元素,此时notifyElementAdded正在遍历集合

解决方法是:

```Java
//good 将"外星人"代码移除同步块
private void notifyElementAdded(E element) {
	List<SetObserver<E>> snapshot = null;
	synchronized(observers) {
		snapshot = new ArrayList<SetObserver<E>>(observers);
	}
	for (SetObserver<E> observer : snapshot)
		observer.added(this, element);
}
```

事实上,有一种更好的方式移除同步代码中的外星人代码,在JDK1.5之后,Java提供了一系列并发集合,如 CopyOnWriteArrayList刚好可以解决上面的问题,很适合于观察者模式:

```Java
//good Thread-safe observable set with CopyOnWriteArrayList
private final List<SetObserver<E>> observers = 
	new CopyOnWriteArrayList<SetObserver<E>>();
public void addObserver(SetObserver<E> observer) {
	observers.add(observer);
}
public boolean removeObserver(SetObserver<E> observer) {
	return observers.remove(observer);
}
private void notifyElementAdded(E element) {
	for (SetObserver<E> observer : observers)
		observer.added(this, element);
}
```

第一部分讨论了正确性,现在我们看看性能.在多核时代,过度同步的开销并不在CPU获取锁身上,真正的开销在于平行执行的机会和多个核心CPU保持一致的内存记忆模型.过度同步另一个隐藏的开销是减少了虚拟机优化代码块的执行

并发使用的前提是保证可变对象线程安全,我们仅仅需要在需要同步的地方同步,而不是将整个对象同步.因为这个原因,在1.5的版本,StringBuffer被StringBuild替代了,不要将整个对象同步,而仅仅告知它是非线程安全对象,然后客户端调用的时候在做适当处理

> 总之,为了避免死锁和数据损坏,不要写"外星人"代码,尽量减少同步块中的工作量.在多核时代,更重要的是不使用同步.当你需要设计一个可变类的时候,好好想想在该同步的地方同步,不要过度

---

##### Item 68:使用Java或者Android平台提供好的线程工具类

不要直接使用thread,因为它控制不住

在JDK1.5中,java.util.concurrent提供了基于接口的任务类框架Executor,创建了一种更好的工作流方式:

```Java
ExecutorService executor = Executors.newSingleThreadExecutor();
executor.execute(runnable);//执行
executor.shutdown();//优雅的通知停止
```



你可以用ExecutorService做很多事情,比如你可以等待特定任务完成,可以等待任务停止(用awaitTermination方法)等等

如果你想多个线程做任务队列里的工作,可以通过线程池创建不同的ExecutorService.  java.util.concurrent.Executors提供了很多你需要的静态工厂方法,你也可以直接使用 ThreadPoolExecutor.

对特定的任务选择合适的线程池.如果你想写一个小的轻量级的加载服务,用Executors.newCachedThreadPool是一个很好的选择.但是这个线程池不适合做很重的操作,Executors.newFixedThreadPool是一个很好的选择,内部维护了一个固定数量的线程.

完整的Executor介绍超出了本书的内容,有兴趣可以参考<Java Concurrency in Practice>

---
##### Item 69: 使用并发工具类代替wait和notify

随着1.5版本的诞生,wait和notify使用越来越少了,因为Java平台提供了高效的并发工具类

在java.util.concurrent提供了三种工具类

1.前面介绍的Executor框架

2.并发集合

3.同步器



并发集合提供了基于标准集合接口(如List,Queue,Map)的高性能并发,为了实现高并发,这些接口管理内部的同步.因此在不需要并发的情况下不要使用这些集合,因为访问锁的原因减慢了程序运行

这意味着客户端不能原子性的使用并发集合组合方法,因此有一些接口集成了基于状态修改的操作来提供原子性.如,ConcurrentMap 集成 Map并添加了自己的方法如putIfAbsent(key,value)

```Java
//未优化
private static final ConcurrentMap<String, String> map = 
	new ConcurrentHashMap<String, String>();
public static String intern(String s) {
	String previousValue = map.putIfAbsent(s, s);
	return previousValue == null ? s : previousValue;
}
```



事实上,有一种更好的方式,ConcurrentHashMap优化过检索操作,如get.因此:

```Java
//优化后,运行更快
public static String intern(String s) {
    long startNanos = System.nanoTime();
	String result = map.get(s);
	if (result == null) {
		result = map.putIfAbsent(s, s);
		if (result == null)
			result = s;
	}
    Log.d("time","spend "+System.nanoTime() - startNanos);
	return result;
}
```

对于间隔计时来说,建议使用System.nanoTime(),它比 System.currentTimeMillis更精确,而且不受系统时间调整的影响

虽然你总是应该使用并发工具,但是你可能有时会用到wait和notify.标准的调用wait方法模板如下:

```Java
// The standard idiom for using the wait method
synchronized (obj) {
	while (<condition does not hold>)
		obj.wait(); // (Releases lock, and reacquires on wakeup)
	... // Perform action appropriate to condition
}
```

**永远在循环内部调用wait方法**,循环会在wait之前或之后检查条件是否成立

相关问题是是否应使用“notify”或“notifyAll”来唤醒等待线程.（重复调用notify只会唤醒一个等待的线程,notifyAll会唤醒所有的等待线程.).有一个合理的，保守的建议说你应该始终使用NotifyAll,它总是会产生正确的结果，因为它保证唤醒线程,可能也会唤醒其他线程，但这不会影响程序的正确性.这些线程将检查它们等待的条件，发现不满足，将继续等待

作为一种优化来讲,你应该选择notify.如果所有的线程都在等待同一个条件,notify保证同一时间只有一个线程的条件为真

> 总之,直接使用wait和notify的情况已经很少了,如果需要用的话请参考上面的模板并确保线程间的一致性

---
##### Item 70: Document thread safety
文档注释清楚线程安全性

一个类的行为怎么样,有很重要一部分取决于客户端.如果你没有声明清楚一个类的方法,用这个类的程序员将会被迫做出一些假设.如果这些假设是错误的,结果不言自明.

**线程安全等级有很多种,一个类必须注释好它支持的线程安全等级**

以下列出了部分线程安全等级:

- immutable-不可变的,这个类的实例输出常量,没有必要在外部同步,如String,BigInteger

- unconditional thread-safe-无条件线程安全,这个类的实例是可变的,但是内部处理了用于并发的同步,所以也没有必要外部同步,如Random,ConcurrentHashMap

- conditionally thread-safe-同无条件线程安全一样,除非一些方法为了并发需要外部同步.如Collections.synchronized返回的集合使用iterators需要外部同步

- not thread-safe-线程不安全,这个类的实例是可变的.并发使用的情况下调用端需要对每个方法都同步.如ArrayList,HashMap

- thread-hostile-线程敌对,很少使用.即使使用了外部同步依然是线程不安全的.如System.runFinalizersOnExit(已经被遗弃)

  

文档注释条件线程安全类需要小心.你必须声明哪些调用序列需要外部同步,调用序列都需要哪些锁对象.很典型的情况锁对象是实例本身,但是也有例外.如果一个对象持有了另一个view的引用,调用端需要对返回的对象同步,防止对象对view做修改.例如:

```Java
Map<K, V> m = Collections.synchronizedMap(new HashMap<K, V>());
...
Set<K> s = m.keySet(); // 不需要放在同步块中
...
synchronized(m) { // 同步m 而不是s
	for (K key : s)
		key.f();
}
```

当一个类提交使用一个可公开访问的锁时，它使客户端能够执行一系列的方法修改，但是这种灵活性是要付出代价的.它不兼容高性能的并发集合排序控制,如ConcurrentHashMap或ConcurrentLinkedQueue.此外，客户端可以通过长期持有公开访问锁制造"服务拒绝"攻击. 这可以是偶然的,也可以是有意的.

为了防止"服务拒绝"攻击,你应该使用私有对象锁来代替同步块:

```Java
private final Object lock = new Object();//避免"服务拒绝"攻击
public void foo() {
	synchronized(lock) {
	...
	}
}
```

注意lock是用final修饰的,防止修改了锁的内容

值得重申的是,私有对象锁只能用于无条件线程安全类,因为条件线程安全类需要在调用方法的时候声明需要获取的锁对象.

> 总之,每一个类应该文档注释它的线程安全属性
---

##### Item 71: Use lazy initialization judiciously
明智地使用延迟初始化

延迟初始化是为了在需要用到值的地方初始化.如果这个值从不需要,这个实例也从不会初始化.延迟初始化是一种优化策略,也可以用来打破实例初始化的有害循环.

对大部分的优化来说,最好的建议是**不要优化,除非你不得不做**

,延迟初始化是一把双刃剑.它降低了初始化一个类的代价,却增加了访问这种字段的代价.如下情况会损害性能:需要延迟加载字段在类当中使用占比,用于初始化的代价有多大,访问有多频繁.

也就是说,延迟初始化有其用途.如果一个字段仅在类实例的一小部分上被访问,并且初始化该字段的代价很高,那么延迟初始化可能是值得的.唯一确定的方法是测量有和没有延迟初始化的类的性能.

在大多数的文章中,都**建议使用普通初始化**.下面是典型的普通初始化声明:

```Java
private final FieldType field = computeFieldValue();
```

如果你想使用延迟初始化来中断初始化循环,使用synchronized关键字:

```Java
// Lazy initialization of instance field - synchronized accessor
private FieldType field;
synchronized FieldType getField() {
	if (field == null)
		field = computeFieldValue();
	return field;
}
```

当你需要使用延迟初始化来提高性能,有如下三种模型

- 静态字段,使用延迟初始化holder模型

```Java
// Lazy initialization holder class idiom for static fields
private static class FieldHolder {
	static final FieldType field = computeFieldValue();
}
static FieldType getField() { return FieldHolder.field; }
```

当getField()方法第一次被调用,它会读FieldHolder.field

,导致FieldHolder类初始化.这种模式的美妙之处在于getField()不需要同步修饰而且只执行字段访问,所以延迟初始化不会增加访问的开销.

- 实例字段,使用双重检查模型

```Java
private volatile FieldType field;
FieldType getField() {
	FieldType result = field;
	if (result == null) { // First check (no locking)
		synchronized(this) {
			result = field;
			if (result == null) // Second check (with locking)
				field = result = computeFieldValue();
		}
	}
	return result;
}
```

这段代码可能看起来有点复杂.特别是对局部变量结果可能不明确.此变量所做的是确保字段在它已经初始化的情况下只读取一次.虽然这并不是绝对必要的,但它可能会提高性能,并且按照适用于低级别并发编程的标准来说更加优雅.在我的机器上,上面的方法比没有局部变量的版本明显快25%。

在JDK1.5之前,双重检查模型不能很好的工作,因为volatile关键字不够强壮来支撑它.之后的版本修复了这个问题.当然你也可以用这种方式来延迟初始化静态实例,但holder模型对于静态实例是最好的选择

- 实例字段,不介意重复初始化,使用单检查模型

```Java
// Single-check idiom - can cause repeated initialization!
private volatile FieldType field;
private FieldType getField() {
	FieldType result = field;
	if (result == null) 
		field = result = computeFieldValue();
	return result;
}
```

以上不仅仅可以用于实例,也可以用于Object

> 总之,你应该正常地初始化大多数字段,而不是延迟加载.如果必须延迟初始化字段以实现性能目标,或打破有害的初始化循环.有如上三种模型做参考.
---
##### Item 72: Don’t depend on the thread scheduler
不要依赖线程调度

当许多线程运行时,线程调度决定哪些线程要运行,以及运行多长时间.任何合理的操作系统都将试图公平地做出这种判断,但是策略可以不同.因此,编写好的程序不应该取决于策略细节.任何依赖线程调度器进行正确性或性能的程序很可能是不可移植的

写一个强健的,响应快的,可移植的程序最好的方式是确保可运行的线程数量不明显超过处理器的数量.这样程序的行为在不同的策略下也没有太大变化.注意,可运行线程的数量不等同于线程总数.处于wait状态下的线程是非运行线程.

减少可运行线程数量的主要技术是让每个线程做一些有用的工作,然后等待更多的线程.如果线程没有做有用的工作,它们就不应该运行.在Executor Framework(Item68)中,这意味着适当地调整线程池的大小,并保持任务相当小且相互独立.任务不应该太小否则调度开销会损害性能

> 总之,不要依赖线程调度来确保程序的正确性.也不要依赖Thread.yield或线程优先级.线程优先级可能被谨慎地用于提高已经工作的程序的服务质量,但是绝不能用来"修复"无法工作的程序.
---
##### Item 73: Avoid thread groups
避免使用线程组

线程组并没有提供很多有用的功能,而且它们提供的许多功能都存在缺陷.线程组最好被看作是一个失败的实验.你应该忽略它们的存在.如果你设计了一个处理逻辑线程组的类,那么你应该使用线程池来替代

---

>?为什么xml布局中,使用了layout_weight="1",会提示将宽/高设为0dp?

本章完
[第八章:异常](https://www.jianshu.com/p/009327a97ae9)   
