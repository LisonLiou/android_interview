# Android基础

## 1. 什么是Activity

​		Activity是安卓四大组件之一，一般的一个用户交互界面对应一个Activiy。activity是Context的子类，同时实现了window.callback和keyevent.callback，可以处理用户与窗体交互的事件。常用的有FragmentActivity、ListActivity、PreferenceActivity、TabActivity等。

## 2.Activity的生命周期

​		Activity从创建到销毁有多种状态，从一个状态切换到另一个状态时会激发相应的回调方法，这些回调方法包括：onCreate onStart onResume onPause onStop onDestroy 这些方法都是两两对应的，onCreate创建对应onDestroy销毁；onStart可见于onStop不可见；onResume可编辑（即获得焦点）与onPause（可见但未获取焦点，例如被对话框遮挡）；如果界面有共同特点或者功能的时候，一般会自定义一个BaseActivity，例如统一处理进度对话框的显示与销毁。

## 3. 如何保存Activity的状态或者说Activity重启如何保存数据

​		Activity的状态通常情况下系统会自动保存，只有当我们需要保存额外的数据时才需要用到这样的功能。一般来说，onPause()和onStop()方法被调用后activity实例仍然存在于内存中，activity的所有信息和状态数据不会消失，当activity重新回到前台之后，所有的改变都会得到保留。

​		但是当系统内存不足时，onPause()和onStop()方法被调用后activity可能会被系统摧毁，此时内存中就不会存有该activity的实例对象了。如果之后这个activity重新回到前台，之前所作的改变都会消失。为了避免此种情况的发生，我们可以重写onSavedInstanceState()方法。onSavedInstanceState()方法接收一个Bundle类型的参数，开发者可以将状态数据存储到这个Bundle对象中，这样即使activity被系统销毁，当用户重新启动这个activity而调用它的onCreate()方法时，上述的Bundle对象会作为实参传递给onCreate()方法，开发者可以从Bundle对象中取出保存的数据，然后利用这些数据将activity恢复到被销毁之前的状态。

​		需要注意的是onSavedInstanceState()方法并不是一定会被系统调用，因为有些场景是不需要保存状态数据的。比如用户按下BACK键退出activity时，用户显然是要关闭这个activity，此时是没有必要保存数据以供下次恢复的，也就是onSavedInstanceState()方法不会被调用。如果系统决定调用onSavedInstanceState()方法，那调用将发生在onPause()或onStop()方法之前。


```java
private String TAG = MainActivity.class.getSimpleName();

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    if (savedInstanceState != null) {
        Log.e(TAG, savedInstanceState.getString("test"));
    }
}

@Override
protected void onSaveInstanceState(@NonNull @org.jetbrains.annotations.NotNull Bundle outState) {
    super.onSaveInstanceState(outState);
    outState.putString("test", "letme");
    Log.e(TAG, "onSaveInstanceState");
}
```

## 4.  两个Activity之间跳转时必然会执行的是哪几个方法？

​		一般情况下比如有两个activity，分别为A和B。当在A中启动B时，A会调用onPause()方法，然后B调用onCreate()、onStart()、onResume()方法。

​		这个时候B覆盖了A，A会调用onStop()方法，如果B窗体是透明的，或者是对话框样式，则不会调用A的onStop()方法。

## 5. 横竖屏切换时Activity的生命周期

​		此时的生命周期跟Manifest.xml中的配置有关系。

		1. 当不设置activity的android:configChanges时，屏幕切换时会重新调用各个生命周期，默认首先销毁当前activity，然后重新加载。
		2. 当设置了activity的android:configChanges="orientation|keyboardHidden|screenSize"时，屏幕切换不会重新调用各个生命周期，只会执行onConfigurationChanged方法。

## 6. 如何退出Activity，如何安全退出已调用多个Activity的Application？

		1. 通常用户退出activity只需按返回键，代码退出activity直接调用finish()方法即可。
		2. 记录打开的Activity：也可维护一个Activity栈，在需要退出时，依次弹出每个activity并执行finish()方法。
		3. 发送特定广播：在需要结束应用时，发送一个特定的广播，每个Activity受到广播后，关闭即可。
		4. 递归退出：再打开新的activity时使用startActivityForResult，然后自己设定标记，在onActivityResult中处理，递归关闭。
		5. 通过Intent的Flag来实现：intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP)激活一个新的activity。此时如果该任务栈中已经有该activity，那么系统会把这个activity上面所有的activity干掉。相当于给activity设置启动模式为SingleTop。

## 7. Activity四种启动模式

| *启动模式*     | *简介*                                                       |
| -------------- | ------------------------------------------------------------ |
| standard       | 默认模式，**Act**ivity会被实例化多次                         |
| singleTop      | 如果栈顶存在Activity实例则通过onNewIntent重用，否则重新创建Activity实例。只要不在栈顶则重新创建，任务栈中可能存在多个实例。 |
| singleTask     | 只要在任务栈中存在Activity实例就重用，否则重新创建Activity实例。TaskId不变。 |
| singleInstance | 在新的任务栈中开启，并且全局只有一个Activity实例。TaskId会变化。 |
|                |                                                              |

## 8. Android中的Context，Activity，Service，Application有什么区别？

Context具体作用包括：

- 启动新的Activity
- 启动和停止Service
- 发送广播消息（Intent）
- 注册广播消息（Intent）接收者
- 访问APK中各种资源，例如Resources和AssetManager
- 创建View
- 访问Package的相关信息
- APK的各种权限管理

相同：Activity和Service都继承自ContextWrapper，而ContextWrapper又是Context的子类，那么Activity、Service最终也都继承自Context；而Applilcation直接继承自Context。Context字面意思是上是上下文的意思，在实际应用中它也确实起到了管理上下文环境中各个参数和变量的作用，方便我们可以简单的访问到各种资源。

不同：维护的生命周期不同。Context维护的是当前Activity的生命周期，Application维护的是整个项目的生命周期。

使用Context时需要注意内存泄漏问题，需注意一下几个方面：

1. 不要让生命周期长的对象引用activity context，即保证引用activity的对象的生命周期要与activity本身是一样的。
2. 对于生命周期长的对象，可以使用application context。
3. 避免非静态的内部类，尽量使用静态类，避免生命周期问题，注意内部类对外部对象的引用导致的生命周期变化。

## 9. Activity和Service通信的两种方式

	1. Binder对象具体实现两个组件之间的交互

```java
public class MyService extends Service {
 
    public MyService() {
    }
 
    private DownloadBinder mBinder = new DownloadBinder();
 
    class DownloadBinder extends Binder {
 
        public void startDownload() {
            Log.d("MyService", "startDownload executed");
        }//在服务中自定义startDownload()方法，待会活动中调用此方法
 
        public int getProgress() {
            Log.d("MyService", "getProgress executed");
            return 0;
        }//在服务中自定义getProgress()方法，待会活动中调用此方法
 
    }
 
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }//普通服务的不同之处，onBind()方法不在打酱油，而是会返回一个实例
```

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener{
 
    private MyService.DownloadBinder downloadBinder;
 
    private ServiceConnection connection = new ServiceConnection() {
		//可交互的后台服务与普通服务的不同之处，就在于这个connection建立起了两者的联系
        @Override
        public void onServiceDisconnected(ComponentName name) {
        }
 
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            downloadBinder = (MyService.DownloadBinder) service;
            downloadBinder.startDownload();
            downloadBinder.getProgress();
        }//onServiceConnected()方法关键，在这里实现对服务的方法的调用
    };
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button bindService = (Button) findViewById(R.id.bind_service);
        Button unbindService = (Button) findViewById(R.id.unbind_service);
        bindService.setOnClickListener(this);
        unbindService.setOnClickListener(this);
 
    }
 
    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.bind_service:
                Intent bindIntent = new Intent(this, MyService.class);
                bindService(bindIntent, connection, BIND_AUTO_CREATE); // 绑定服务和活动，之后活动就可以去调服务的方法了
                break;
            case R.id.unbind_service:
                unbindService(connection); // 解绑服务，服务要记得解绑，不要造成内存泄漏
                break;
            default:
                break;
        }
    }
}
```

2. 使用广播（略）
