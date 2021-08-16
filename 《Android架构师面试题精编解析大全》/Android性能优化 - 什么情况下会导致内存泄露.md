# 什么情况下会导致内存泄漏



Android 的虚拟机是基于寄存器的Dalvik，它的最大堆大小一般是16M，有的机器为24M。因此我们所能利用的内存空间是有限的。如果我们的内存占用超过了一定的水平就会出现OutOfMemory的错误。内存溢出的原因一般如下：

1. 资源释放问题

   程序代码中长期保持某些资源的引用，如Context、Cursor、IO流的引用，当引用资源得不到释放时造成内存泄漏，即引用对象与被引用对象的生命周期尽量保持一致。

2. 对象内存过大问题

   程序中保存了多个好用内存过大的对象（如Bitmap、XML文件等），造成内存超出限制。

3. static关键字的使用问题

   static是java中的一个关键字，当用它来修饰成员变量时，那么该变量就属于该类，而不是该类的实例。所以用static修饰的变量，它的生命周期是很长的，如果用它来引用一些资源耗费过多的实例（Context的情况最多），就需要谨慎对待了。

   ```java
   public class ClassName{
   	private static Context mContext;
   }
   ```

   ​	以上代码中，如果将Activity实例赋值给mContext的话，那么即使该Activity已经onDestroy，但是由于仍有对象保存它的引用，因此该Activity依然不会被释放。举个例子：

   ```java
   private static Drawable sBackground;
   @Override
   protected void onCreate(Bundle bunele){
   	TextView lable = new TextView(this);	//getApplicationContext
   	label.setText("Leaks are bad");
   	
   	if(sBackground == null){
   		sBackground = getDrawable(R.drawable.large_bitmap);
   	}
   	
   	label.setBackgroundDrawable(sBackground);
   	setContentView(label);
   }
   ```

   ​		如上，sBackground是一个静态变量，代码中我们并没有显示的保存Context的引用，但是当Drawable与View连接之后，Drawable就将View设置为一个回调，由于View中是包含Context引用的，所以实际上我们依然保存了Context的引用，引用链如下：

   ```java
   Drawble -> TextView -> Context
   ```

   所以最终该Context也没有得到释放，从而导致内存泄漏。

> 针对Static的解决方案

```java
1. 应当尽量避免static成员变量引用资源耗费过多的实例，比如Context。
2. Context尽量使用ApplicationContext，因为Application的Context生命周期比较长，引用它不会出现内存泄漏问题。
3. 使用WeakReference代替强引用，比如可以使用WeakReference<Context> mContextRef；
```

4. 线程导致的内存溢出

   线程产生的内存泄漏主要原因在于线程生命周期的不可控。

   ```java
   public class MyActivity extends Activity{
   	@Override
   	public void onCreate(Bundle savedInstanceState){
   		super.onCreate(savedInstanceState);
   		setContentView(R.layout.main);
   		
   		new MyThread().start();
   	}
   	
   	private class MyThread extends Thread{
   		@Override
   		public void run(){
   			super.run();
   			//do something while(true)
   		}
   	}
   }
   ```

   ​		这段代码很简单，是我们经常使用的形式。假设一个场景：MyThread的run函数是一个很费时的操作，当我们开启该线程后，将设备的横屏变成了竖屏，一般情况下屏幕宣传会重建Activity，按照我们的想法，老的Activity应该被销毁才对，然而事实上并非如此。

   ​		由于我们的线程是Activity内部类，所以MyThread中保存了Activity的一个引用，当MyThread的run函数没有结束时，MyThread是不会销毁的，所以它所引用的老的Activity也不会被销毁，因此就出现了内存泄漏的问题。

   ​		有些人喜欢用Android提供的AsyncTask，但事实上AsyncTask的问题更加严重，Thread只有在run函数不结束时才会出现这种内存泄漏问题，然而AsyncTask内部的实现机制是运用了ThreadPoolExcutor，该类产生的Thread对象的生命周期是不确定的，是应用程序无法控制的，因此如果AsyncTask作为Activity的内部类，就更容易出现内存泄漏的问题。

   > 针对这种线程导致的内存泄漏问题的解决方案：

   1. 将线程的内部类，改为静态内部类（因为非静态内部类拥有外部类对象的强引用，而静态类则不拥有）。
   2. 在线程内部采用若引用保存Context引用：WeakReference<Context> mContext;
