---
title: Android-10
categories:
- Android
---

基于《第一行代码——Android》@郭霖





# 第十章：后台劳动者，探究Service

## 10.1 Service是什么

即使程序切换到后台或者用户打开了另外一个应用程序，Service仍然能保持正常运行



## 10.2 Android多线程编程

### 基本用法

定义一个线程: 

```kotlin
class MyThread: Thread() {
    override fun run () {
        //Something
    }
}
```



启动线程

```kotlin
MyThread.start()
```



使用继承的方式耦合性有点高~~啥意思~~，我们一般使用实现 **Runnable**接口的方式来定义一个线程

```kotlin
class MyThread : Runnable {
    override fun run() {
        //Something
    }
}
```



相应的启动方式也要改变

```kotlin
val myThread = MyThread()
Thread(myThread).start()
```



当然，如果不想专门定义一个类去实现 **Runnable**接口，也可以写成如下形式

```kotlin
Thread {
    //Something
}.start()
```



而Kotlin提供了一种更简单的方式

```kotlin
thread {
    //Something
}
```

这里的thread是一个内置的顶层函数



### 在子线程中更新UI

Android的UI线程不安全，也就是说，如果要更新应用程序里的UI元素，必须在主线程中进行，否则会出现异常



i.e.改变TextView中的text

```kotlin
class MainActivity : AppCompatActivity() {
    val updateText = 1 //用于表示更新TextView这个动作

    val handler = object : Handler() {
        override fun handleMessage(msg: Message) {
            //可以在这里进行UI操作 在主线程中
            when (msg.what) {
                updateText -> textView.text = "Nice to meet you"
            }
        }

    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        changeTextBtn.setOnClickListener {
            thread {
                val msg = Message()
                msg.what = updateText
                handler.sendMessage(msg) //将Message对象发送出去
            }
        }
    }
}
```



### 解析异步消息处理机制

Android中的异步消息处理主要由四个部分组成

#### Message

用于在线程之间传递消息，除了之前用过 **what**字段，还可以使用 **arg1, arg2**字段携带一些整型数据，使用 **obj**字段携带一个Object对象



#### Handler

主要用处理、发送消息

发送一般用到 **sendMessage, post**等方法

最终会传递到 **Handler**的 **handleMessage**方法中



#### MessageQueue

主要用于存放所有通过 **Handler**发送的消息，这部分消息会一直在队列中等待被处理

每个线程只有一个该对象



#### Looper

调用该对象的 **loop**方法后，就会进入无限循环中，每当发现 **MessageQueue**中存在一条消息，就会将他取出，并且传递到 **Handler**的 **handleMessage**方法中

每个线程只有一个该对象



梳理一下整个流程，看下图

![异步消息处理](https://s1.ax1x.com/2020/07/26/aCRoZT.png)



### 使用AsyncTask

方便我们在子线程中对UI操作的抽象类

在继承时我们可以为 AsyncTask类指定三个泛型参数

- Params 在执行 AsyncTask时需要传入的参数，可用于在后台任务中使用
- Progress 在后台执行任务时，如果需要在界面上显示当前的进度，则使用这里指定的泛型作为进度单位
- Result 当任务执行完毕后，如果需要对结果进行返回，则使用这里指定的泛型作为返回值类型

因此，一个最简单的自定义 **AsyncTsak**类可以写成如下形式

```kotlin
class DownloadTask: AsyncTask<Unit, Int, Boolean> () { //这里第一个泛型参数为Unit 表示不需要传入参数给后台任务 第二个时Int 表示使用整型来作为进度显示单位 第三个为Boolean 表示用布尔型反馈执行结果
    //Something
}
```

当然目前我们定义的这个还是个空任务，需要重写如下几个方法才能完成对任务的定制

#### onPreExcute ( )

在后台任务开始执行之前调用，用于进行一些界面上的初始化操作，比如显示一个进度条对话框

#### doInBackground (Params...)

该方法的所有代码都会在子线程中运行

在这里执行耗时任务

注意，该方法不可以进行UI操作

如果要更新UI元素，比如说反馈当前任务的执行进度，可以调用 **publishProgress (Progress...)**来实现

#### onProgressUpdate (Progress...)

该方法中携带的参数就是在后台任务中传递过来的

该方法中可以对UI进行操作

#### onPostExcute (Result)

当后台任务执行完毕并通过 return语句返回时，该方法很快被调用

返回的数据被作为参数传递到次方法中，可以利用返回数据进行一些UI操作

```kotlin
class DownloadTask : AsyncTask<Unit, Int, Boolean>() {
    override fun onPreExecute() {
        progressDialog.show()
    }

    override fun doInBackground(vararg params: Unit?) = try {
        while (true) {
            val downloadPercent = doDownload() //这是一个虚构的方法
            publishProgress(downloadPercent)
            if (downloadPercent == 100) {
                break
            }
        }
        true
    } catch (e: Exception) {
        false
    }

    override fun onProgressUpdate(vararg values: Int?) {
        progressDialog.setMessage("Download ${values[0]}%")
    }

    override fun onPostExecute(result: Boolean) {
        progressDialog.dismiss() //关闭进度对话框
        //在这里提示下载速度
        if (result) {
            Toast.makeText(context, "Download succeed", Toast.LENGTH_SHORT).show()
        } else {
            Toast.makeText(context, "Download failed", Toast.LENGTH_SHORT).show()
        }
    }

}
```





## 10.3 Service的基本用法

### 定义一个Service

New->Service->Service 没啥好说的

```kotlin
class MyService : Service() {

    override fun onBind(intent: Intent): IBinder { //Service中唯一的抽象方法 必须重写
        TODO("Return the communication channel to the service.")
    }
    
    //以下三个里来写我们的处理逻辑

    override fun onCreate() { //Service创建时调用
        super.onCreate()
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int { //每次启动时调用
        return super.onStartCommand(intent, flags, startId)
    }

    override fun onDestroy() { //销毁时调用
        super.onDestroy()
    }
}

```

AS会自动对组件注册，可以检查一下



### 启动和停止Service

借助 **Intent**完成

直接看代码

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        startServiceBtn.setOnClickListener {
            val intent = Intent(this, MyService::class.java)
            startService(intent) //启动
        }

        stopServiceBtn.setOnClickListener {
            val intent = Intent(this, MyService::class.java)
            stopService(intent) //停止
        }
    }
}
```



### Activity和Service进行通信

举个例子，我们希望在MyService里提供一个下载功能，然后再Activity中可以决定何时开始下载，以及随时查看下载进度

实现这个功能的思路是创建一个专门的Binder对象来对下载功能进行管理

修改两份代码

```kotlin
class MyService : Service() {
    private val mBinder = DownloadBinder()

    class DownloadBinder() : Binder() {
        fun startDownload() {
            Log.d("MyService", "startDownload executed")
        }

        fun getProgress(): Int {
            Log.d("MyService", "getProgress executed")
            return 0
        }
    }


    override fun onBind(intent: Intent): IBinder {
        return mBinder
    }

    override fun onCreate() {
        super.onCreate()
        Log.d("MyService", "onCreate executed")
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        Log.d("MyService", "onStartCommand executed")
        return super.onStartCommand(intent, flags, startId)
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d("MyService", "onDestroy executed")
    }
}
```



```kotlin
class MainActivity : AppCompatActivity() {

    lateinit var downloadBinder: MyService.DownloadBinder

    private val connection = object : ServiceConnection {
        override fun onServiceConnected(name: ComponentName, service: IBinder?) {
            downloadBinder = service as MyService.DownloadBinder
            downloadBinder.startDownload()
            downloadBinder.getProgress()
        }

        override fun onServiceDisconnected(p0: ComponentName?) {
        }
    }


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        startServiceBtn.setOnClickListener {
            val intent = Intent(this, MyService::class.java)
            startService(intent) //启动
        }

        stopServiceBtn.setOnClickListener {
            val intent = Intent(this, MyService::class.java)
            stopService(intent) //停止
        }

        bindServiceBtn.setOnClickListener {
            val intent = Intent(this,MyService::class.java)
            //第三个参数是一个标志位 此处传入的表示在绑定后自动创建Service 这会使得onCreate方法被调用 而onStartCommand不会被调用
            bindService(intent,connection,Context.BIND_AUTO_CREATE) //绑定Service
        }

        unbindServiceBtn.setOnClickListener {
            unbindService(connection) //解绑Service
        }
    }
}
```



## 10.4 生命周期

start后stop，这时onDestroy就会执行

类似bind后unbind，onDestroy就会执行

如果同时start和bind，那两个对应方法都要执行



## 10.5 更多技巧

### 前台Service

只有当应用保持在前台可见状态的情况下，Service才能保证稳定运行，一旦应用进入后台，Service随时可能被系统回收

如果希望Service一直运行，考虑使用前台Service

创建一个前台Service

```kotlin
class MyService : Service() {
    private val mBinder = DownloadBinder()

    class DownloadBinder() : Binder() {
        fun startDownload() {
            Log.d("MyService", "startDownload executed")
        }

        fun getProgress(): Int {
            Log.d("MyService", "getProgress executed")
            return 0
        }
    }


    override fun onBind(intent: Intent): IBinder {
        return mBinder
    }

    override fun onCreate() {
        super.onCreate()
        Log.d("MyService", "onCreate executed")
        val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                "my_service",
                "前台Service通知",
                NotificationManager.IMPORTANCE_DEFAULT
            )
            manager.createNotificationChannel(channel)
        }

        val intent = Intent(this, MainActivity::class.java)
        val pi = PendingIntent.getActivity(this, 0, intent, 0)
        val notification = NotificationCompat.Builder(this, "my_service")
            .setContentTitle("This is content title")
            .setContentText("This is content text")
            .setSmallIcon(R.drawable.small_icon)
            .setLargeIcon(BitmapFactory.decodeResource(resources, R.drawable.large_icon))
            .setContentIntent(pi)
            .build()
        startForeground(1, notification)

    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        Log.d("MyService", "onStartCommand executed")
        return super.onStartCommand(intent, flags, startId)
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d("MyService", "onDestroy executed")
    }
}

```

注意，Android9.0开始 前台Service必须在AndroidManifest.xml中进行权限声明



### 使用IntentService

Service默认运行在主线程，如果在其中进行耗时的操作，容易出现ANR(Application Not Responding)，所与需要用到多线程编程

为了简单地创建一个异步、会自动停止的Service，Android专门提供了IntentService类

i.e.

创建**src/main/java/com/example/servicetest/MyIntentService.kt**

```kotlin
class MyIntentService : IntentService("MyIntentService") {
    override fun onHandleIntent(p0: Intent?) {
        Log.d("MyIntentService", "Thread id is ${Thread.currentThread().name}")
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d("MyIntentService", "onDestroy executed")
    }
}
```

**MainActivity.kt**中添加

```kotlin
        startIntentServiceBtn.setOnClickListener {
            Log.d("MainActivity","Thread id is ${Thread.currentThread().name}")
            val intent = Intent(this,MyIntentService::class.java)
            startService(intent)
        }
```

别忘了注册

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.servicetest">

    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        
        ...Something

        <service
            android:name=".MyIntentService"
            android:enabled="true"
            android:exported="true" />
    </application>

</manifest>
```



# Kotlin拓展

## 对泛型进行实化

首先，函数必须是内联函数

其次，声明泛型的地方必须加上 **reified**关键字

```kotlin
inline fun <reified T> getGenericType () = T::class.java
```

上述函数直接返回了当前指定泛型的实际类型



## 泛型实化的应用

启动一个Activity：

首先新建一个 **reified.kt**

```kotlin
inline fun <reified T> startActivity(context: Context) {
    val intent = Intent(context, T::class.java)
    context.startActivity(intent)
}

inline fun <reified T> startActivity(context: Context, block: Intent.() -> Unit) {
    val intent = Intent(context,T::class.java)
    intent.block()
    context.startActivity(intent)
}
```

现在可以按照如下方法启动Activity

```kotlin
startActivity<TestActivity>(context)

startActivity<TestActivity>(context) {
    putExtra("param1", "data")
    putExtra("param2", 123)
}
```

## 泛型的协变和逆变

理论性强，理解有难度，看书吧

























































