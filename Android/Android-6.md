---
title: Android-6
categories:
- Android
---

基于《第一行代码——Android》@郭霖



# 第六章：广播机制

## 6.1 简介

Android中的每个app可以对自己感兴趣的广播进行注册， 这样该程序只会受到自己所关心的广播内容

发送广播使用 **Intent**

接受广播使用 **BroadcastReceiver**

广播分为两种：

- 标准广播： 所有**BroadcastReceiver**几乎会在同一时刻收到消息  效率高，但是无法截断
- 有序广播： 同一时刻只有一个**BroadcastReceiver**能够收到，人如其名可以规定先后接受顺序，也可以截断



## 6.2 接受系统广播

### 动态注册监听时间变化

在代码中注册称为动态注册

在**AndroidManifest.xml**中注册称为静态注册

```kotlin
class MainActivity : AppCompatActivity() {
    //Step1：创建Receiver
    lateinit var timeChangeReceiver: TimeChangeReceiver

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        //Step2:创建IntentFilter实例 通过addAction方法明确要接受什么广播
        val intentFilter = IntentFilter()
        intentFilter.addAction("android.intent.action.TIME_TICK")

        //Step3:注册
        timeChangeReceiver = TimeChangeReceiver()
        registerReceiver(timeChangeReceiver, intentFilter)
    }

    //Step4:动态注册的一定要取消注册
    override fun onDestroy() {
        super.onDestroy()
        unregisterReceiver(timeChangeReceiver)
    }


    //step1:定义一个继承自BoardcastReceiver的内部类 重写onReceive方法 这样当时间改变 该函数就会弹出文本消息
    inner class TimeChangeReceiver : BroadcastReceiver() {
        override fun onReceive(p0: Context?, p1: Intent?) {
            Toast.makeText(p0, "Time has changed", Toast.LENGTH_SHORT).show()
        }
    }
}
```



这仅仅是时间改变，完整的系统广播列表可以到如下路径查看

**<Android SDK>/platforms/<任意 android api 版本>/data/broadcast_actions.txt**



### 静态注册实现开机启动

右键**com.example.broadcasttest** -> **New** -> **Other** -> **Broadcast Receiver** 来创建

这种快捷创建的好处在于自动帮我们完成了注册

修改其中代码 也就添加一行

```kotlin
class BootCompleteReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        // This method is called when the BroadcastReceiver is receiving an Intent broadcast.
        Toast.makeText(context,"Boot Complete",Toast.LENGTH_LONG).show()
    }
}
```



接着修改**src/main/AndroidManifest.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.broadcasttest">
    
    <!--add line 权限获取-->
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <receiver
            android:name=".BootCompleteReceiver"
            android:enabled="true"
            android:exported="true">

            <!--add 3 lines-->
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
            </intent-filter>

        </receiver>

        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```



**注意** ：不要再**onReceive**方法中写过多逻辑或者耗时操作，因为BroadcastReceiver不允许开启线程，运行太久不结束程序就会错误



## 6.3 发送自定义广播

#### 发送标准广播

##### Step1：新建Receiver类

类似 不再解释

```kotlin
class MyBroadcastReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        // This method is called when the BroadcastReceiver is receiving an Intent broadcast.
        Toast.makeText(context,"received in MyBroadcastReceiver",Toast.LENGTH_SHORT).show()
    }
}
```



##### Step2：修改AndroidManifest

类似 不再解释

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.broadcasttest">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <receiver
            android:name=".MyBroadcastReceiver"
            android:enabled="true"
            android:exported="true">
            <!-- add 3 lines -->
            <intent-filter>
                <action android:name="com.example.broadcasttest.MY_BROADCAST" />
            </intent-filter>
            
        </receiver>
        ...Something
    </application>

</manifest>
```



##### Step3:定义按钮 发送广播

添加按钮布局

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    >

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/button"
        android:text="Send Broadcast"
        />

</LinearLayout>
```

添加按钮点击事件

```kotlin
class MainActivity : AppCompatActivity() {
    //Something
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        button.setOnClickListener {
            //构建Intent对象 并把要发送的广播植入 并传入当前包名 最后发送
            val intent = Intent("com.example.broadcasttest.MY_BROADCAST")
            intent.setPackage(packageName) //一定要设定是发送给哪个程序的 从而让他变成显式广播 因为静态注册的Receiver不能接受隐式广播
            sendBroadcast(intent)
        }
        //Something
    }
	//Something
}
```





### 发送有序广播

##### Step1: 新建Receiver类

一样的 不说了

```kotlin
class AnotherBroadcastReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        // This method is called when the BroadcastReceiver is receiving an Intent broadcast.
        Toast.makeText(context,"received in AnotherBroadcastReceiver", Toast.LENGTH_SHORT).show()
    }
}

```



##### Step2: 修改AndroidManifest

注意**priority**属性 他是确定有序广播发送顺序的关键

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.broadcasttest">
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <receiver
            android:name=".AnotherBroadcastReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="com.example.broadcasttest.MY_BROADCAST" />
            </intent-filter>
        </receiver>
        <receiver
            android:name=".MyBroadcastReceiver"
            android:enabled="true"
            android:exported="true">

            <!-- add 3 lines priority属性实现有序广播先后-->
            <intent-filter android:priority="100">
                <action android:name="com.example.broadcasttest.MY_BROADCAST" />
            </intent-filter>
        </receiver>
		...Something
    </application>

</manifest>
```



##### Step3: 发送有序广播

就改一行

```kotlin
class MainActivity : AppCompatActivity() {
    //Something
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        button.setOnClickListener {
            val intent = Intent("com.example.broadcasttest.MY_BROADCAST")
            intent.setPackage(packageName)
            //change here
            sendOrderedBroadcast(intent,null)
        }
        //Something
    }
	//Something
}
```



##### Extra Step: 截断操作

就加一行

```kotlin
class MyBroadcastReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        // This method is called when the BroadcastReceiver is receiving an Intent broadcast.
        Toast.makeText(context,"received in MyBroadcastReceiver",Toast.LENGTH_SHORT).show()
        //add this line
        abortBroadcast()
    }
}

```



## 6.4 实践： 强制下线功能

看书看码吧



# Kotlin拓展：高阶函数详解答

## 定义高阶函数

Def：一个函数接收另一个函数作为参数，或者返回值的类型是另一个函数

Kotlin中函数是一个类型 就像 String、Int...

基本语法

```kotlin
(Type1, Type2) -> Type3
```

~~人类迷惑行为大赏++~~

看个例子吧还是

```kotlin
fun example(func: (String, Int) -> Unit) { //Unit类似于void
    //Something
}
```

更清晰的例子

```kotlin
fun num1AndNum2(num1: Int, num2: Int, operation: (Int, Int) -> Int): Int {
    return operation(num1, num2)
}

fun plus(num1: Int, num2: Int): Int {
    return num1 + num2
}

fun minus(num1: Int, num2: Int): Int {
    return num1 - num2
}

fun main() {
    val num1 = 10
    val num2 = 38
    val result1 = num1AndNum2(num1, num2, ::plus)
    val result2 = num1AndNum2(num1, num2, ::minus)
    println("First: $result1") //48
    println("Second: $result2") //-28
}
```



更高级的例子

```kotlin
package com.example.broadcastbestpractice

import java.lang.StringBuilder

//StringBulider. 是啥 定义高阶函数语法的完整形式中要在函数类型前面加上ClassName 表示函数定义在哪一个类里
fun StringBuilder.build(block: StringBuilder.() -> Unit): StringBuilder {
    block()
    return this
}

fun main() {
    val list = listOf("XCB", "GZA", "YCH", "CHC", "LAT", "MKW")
    val result = StringBuilder().build {
        append("WTF?")
        for (student in list) {
            append(student).append("\n")
        }
        append("WTTTTTFFFFFF?")
    }
    println(result.toString()) //自己想象
}
```



## 内联函数的作用

在定义高阶函数之前加上关键字**inline** 

作用是消除Lambda表达式带来的运行时候的开销

具体原理涉及Java 但是爷不懂Java ~~有机会一定学~~

~~为什么涉及Java 因为Kotlin最终还是编译成Java字节码的~~



## nonline 和 crossinline

一个高阶函数有两个函数参数 只想内联一个 那么给另一个前面加上 **noinline**

**注意**：内联函数的Lambda表达式允许使用return进行返回 而非内联只能局部返回

**crossinline**可以声明在高阶函数的函数参数前面 用于保证内联函数的Lambda表达式中不会使用return 除此以外保留所有内联函数特性



















