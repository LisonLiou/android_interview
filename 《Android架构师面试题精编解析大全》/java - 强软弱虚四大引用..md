# 强软弱虚四大引用 - JAVA



## 1. 强引用（GC: 不回收，死都不回收）

​	当内存不足时，JVM开始垃圾回收，对于强引用的对象，就算是出现了OOM也不会对该对象进行回收，死也不收。

​	强引用是我们最常见的普遍对象引用，只要还有强引用指向一个对象，就能表明对象还活着，垃圾收集器不会碰这种对象。JAVA中最常见的就是强引用，把一个对象付给另一个引用变量就是一个强引用。当一个对象被强引用变量引用时，它处于科大状态，它是不可能被垃圾回收机制回收的，即使该对象以后永远都不会被用到JVM也不会回收，因此强引用是造成JAVA内存泄漏的主要原因之一。

​	对于一个普通对象，如果没有其他的引用关系，只要超过了引用的作用域或者显示地将相应的（强）引用赋值为null，一般认为就是可以被垃圾收集了。

```java
public class StrongR {

	public static void main(String[] args) {
		Object obj1 = new Object();
		Object obj2 = obj1;
		
		obj1=null;
		System.gc();
		
		System.out.println(obj1);
		System.out.println(obj2);
	}
}

// 輸出
null
java.lang.Object@15db9742

```

## 2. 軟引用（GC：内存不夠就回收）

​	软引用是一种相对强引用弱化了一些的引用，需要用java.lang.ref.SoftReference类来实现，可以让对象豁免一些垃圾收集，对于只有软引用的对象来说：当JVM内存充足时它不会被回收，当JVM内存不足时它会被回收。

​	软引用通常用在对内存敏感的程序中，比如高速缓存就有用到软引用，内存够用时就保留，不够用就回收。下面的代码展示了系统内存足够跟不足时软引用对象的GC情况。

```java
public class SoftR {
	public static void main(String[] args) {
		Object o1 = new Object();
		SoftReference<Object> softReference = new SoftReference<Object>(o1);
		System.out.println(o1);
		System.out.println(softReference.get());
		
		o1 = null;
		System.gc();
		
		try {
			byte[] bytes = new byte[30*1024*1024];
		}
		finally {
			System.out.println(o1);
			System.out.println(softReference.get());
		}
	}
}

// 默认输出
java.lang.Object@15db9742
java.lang.Object@15db9742
null
java.lang.Object@15db9742
    
// 使用-Xms5m -Xmx5m  -XX:+PrintGCDetails 启动参数后输出
java.lang.Object@15db9742
java.lang.Object@15db9742
[GC (System.gc()) [PSYoungGen: 764K->496K(1536K)] 764K->680K(5632K), 0.0027296 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System.gc()) [PSYoungGen: 496K->0K(1536K)] [ParOldGen: 184K->537K(4096K)] 680K->537K(5632K), [Metaspace: 2656K->2656K(1056768K)], 0.0046556 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 0K->0K(1536K)] 537K->537K(5632K), 0.0005369 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 0K->0K(1536K)] 537K->537K(5632K), 0.0003787 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 0K->0K(1536K)] [ParOldGen: 537K->537K(4096K)] 537K->537K(5632K), [Metaspace: 2656K->2656K(1056768K)], 0.0017620 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 0K->0K(1536K)] 537K->537K(5632K), 0.0004040 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 0K->0K(1536K)] [ParOldGen: 537K->525K(4096K)] 537K->525K(5632K), [Metaspace: 2656K->2656K(1056768K)], 0.0047294 secs] [Times: user=0.11 sys=0.00, real=0.00 secs] 
null
null
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at reference.SoftR.main(SoftR.java:16)
Heap
 PSYoungGen      total 1536K, used 30K [0x00000000ffe00000, 0x0000000100000000, 0x0000000100000000)
  eden space 1024K, 2% used [0x00000000ffe00000,0x00000000ffe07ad0,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
  to   space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
 ParOldGen       total 4096K, used 525K [0x00000000ffa00000, 0x00000000ffe00000, 0x00000000ffe00000)
  object space 4096K, 12% used [0x00000000ffa00000,0x00000000ffa834d0,0x00000000ffe00000)
 Metaspace       used 2687K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 291K, capacity 386K, committed 512K, reserved 1048576K
```



## 3.弱引用（GC: 只要我被调用，我就回收，不用JVM内存够还是不够）

​	弱引用需要用java.lang.ref.WeakReference类来实现，它比弱引用的生存区更短。对于只有弱引用的对象来说，只要垃圾回收机制一运行，不管JVM的内存空间是否足够，都会回收该对象占用的内存。

```java
public class WeakR {
	public static void main(String[] args) {
		Object o1 = new Object();
		WeakReference<Object> weakReference = new WeakReference<Object>(o1);
		
		System.out.println(o1);
		System.out.println(weakReference.get());
		
		o1 = null;
		System.gc();
		
		System.out.println(o1);
		System.out.println(weakReference.get());
	}
}

//输出
java.lang.Object@15db9742
java.lang.Object@15db9742
null
null
    
-------------------------------------------------------------------------------
private static void myHashmap() {
		HashMap<Integer,String> map =new HashMap<>();
		Integer key= new Integer(1);
		String value="hashmap";
		map.put(key, value);
		System.out.println(map);
		
		key = null;
		System.out.println(map);
		
		System.gc();
		System.out.println(map);
	}

//输出    
{1=hashmap}
{1=hashmap}
{1=hashmap}


private static void myWeakHashmap() {
		WeakHashMap<Integer,String> map = new WeakHashMap<>();
		Integer key = new Integer(2);
		String value = "weakHashmap";
		map.put(key, value);		
		System.out.println(map);
		
		key = null;
		System.out.println(map);
		
		System.gc();
		System.out.println(map);
	}
//输出
{2=weakHashmap}
{2=weakHashmap}
{}

```

## 4. 虚引用

​		虚引用需要java.lang.ref.PhantomReference类来实现。顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象持有虚引用，那么它就和没有任何引用一样，在任何时间都可能被垃圾回收器回收，它不能单点使用也不能通过它访问对象，虚引用必须和引用队列（ReferenceQueue）联合使用。
​		虚引用的主要作用是跟踪对象被垃圾回收的状态，仅仅提供了一种确保对象被finalize以后，做某些事情的机制。PhantomReference的get()方法总是返回null，因此无法访问对应的引用对象，其意义在于说明一个对象已经进入了finalization阶段，可以被GC回收，用来实现比finalization机制更灵活的回收操作。
​		换句话说，设置虚引用关联的唯一目的，就是在这个对象被收集器回收的时候收到一个系统通知或者后续添加进一步的处理。Java技术允许使用finalize()方法在垃圾收集器将对象从内存中清除出去之前做必要的清理工作。

```java
public class PhantomR {

	public static void main(String[] args) throws Exception{
		Object o1 = new Object();
		ReferenceQueue<Override> referenceQueue=new ReferenceQueue<>();
		PhantomReference<Object> phantomReference = new PhantomReference(o1, referenceQueue);
		System.out.println(o1);
		System.out.println(phantomReference.get());
		System.out.println(referenceQueue.poll());
		
		System.out.println("------------------------------");
		o1 = null;
		System.gc();
		Thread.sleep(500);
		
		System.out.println(o1);
		System.out.println(phantomReference.get());
		System.out.println(referenceQueue.poll());
	}
}
//输出
java.lang.Object@15db9742
null
null
------------------------------
null
null
java.lang.ref.PhantomReference@6d06d69c

```

