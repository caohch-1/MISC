---
title: Android-3
categories:
- Android
---

基于《第一行代码——Android》@郭霖

# 第三章：探究Activity

## 3.1 介绍

Activity是包含用户界面的组建 主要用于与用户进行交互

## 3.2 基本使用

任何Activity都应该重写 onCreate() 方法

**app/src/main/java/com.example.activitytest** 下创建**Activity**

### 创建并加载一个布局

创建一个button

**app/src/main/res** 下创建**layout**目录 再在**layout**下创建**Lay resource file**

```xml
<!--@+id/button1 该语法定义一个id 用于后续引用、操作-->
<!--wrap_content 表示刚好包下内容即可-->    
<Button
        android:id="@+id/button1"  
        android:layout_width="match_parent" 
        android:layout_height="wrap_content" 
        android:text="Button 1"
        />
```



回到**app/src/main/java/com.example.activitytest/FirstActivity** 在**onCreate()**方法中加载这个Button布局 

项目中添加的任何资源都会在**R**文件中生成一个相应的资源id

```kotlin
class FirstActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.first_layout) //加入这一行 一般传入一个布局文件的id到该函数
    }
}
```

### 在AndroidManifest.xml中注册

所有**Activity**需要在**AndroidManifest.xml**中注册才能生效 AS会自动完成这一步

但是至此我们还有配置**主Activity** 所以程序无法运行

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.activitytest">

    <application
		...
        <activity android:name=".FirstActivity"
            android:label="This is FirstActivity">   <!--指定标题栏和启动器中应用程序内容及名称-->
            <intent-filter>          <!--配置主Activity-->
                <action android:name="android.intent.action.MAIN" />        <!--配置主Activity-->
                <category android:name="android.intent.category.LAUNCHER" />    <!--配置主Activity-->
            </intent-filter>        <!--配置主Activity-->
        </activity>
    </application>

</manifest>
```



### Toast

将短小的信息通知给用户

定义一个弹出触发点 不如就用刚刚的Button

```kotlin
class FirstActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.first_layout)
        //following are new lines
        val button1: Button = findViewById(R.id.button1) //该方法获取布局文件中定义的元素
        button1.setOnClickListener { //调用该方法为button1注册一个监听器
            Toast.makeText(this, "You clicked Button 1", Toast.LENGTH_SHORT).show() 
            //makeText()创建一个Toast对象 第一个参数为一个Context对象 Activity本身就是 第二个参数不解释 第三个为显示时长
        }
    }
}

//简化写法 因为一个插件会根据布局文件中定义的控件id自动生成一个具有相同名称的变量 供我们直接调用 （该插件已经被自动引入项目）
class FirstActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.first_layout)
        //following are new lines
        button1.setOnClickListener {
            Toast.makeText(this, "You clicked buuton1", Toast.LENGTH_SHORT).show()
        }
    }
}
```



### Menu

**app/src/main/res** 下创建**menu**目录 再在**menu**下创建**Menu resource file**

下列代码用<item>标签创建了两个菜单项 

其中的 id title不解释

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/add_item"
        android:title="Add" />
    <item
        android:id="@+id/remove_item"
        android:title="Remove"
        />
</menu>
```



回到**app/src/main/java/com.example.activitytest/FirstActivity** 重写**onCreateOptionsMenu** ( )

Ctrl + O快捷键  注意千万别选错了 别问我怎么知道的 ~~时间就是这么消逝的~~

```kotlin
    override fun onCreateOptionsMenu(menu: Menu?): Boolean {
        menuInflater.inflate(R.menu.main,menu)  //第一个参数指定通过哪个资源文件夹来创建  第二个参数指定添加到哪一个Menu对象
        return true
    }
```

现在菜单仅仅能够显示 还需要定义响应事件 重写**onOptionsItemSelected ( )**

内容不做解释

```kotlin
    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        when (item.itemId) {
            R.id.add_item -> Toast.makeText(this,"You clicked Add",Toast.LENGTH_SHORT).show()
            R.id.remove_item -> Toast.makeText(this,"You clicked Remove", Toast.LENGTH_SHORT).show()
        }
        return true
    }
```

### 销毁

不解释

```kotlin
        button1.setOnClickListener {
            finish()
        }
```



## 3.3 Intent

由主Activity跳转到其他Activity

### 显式Intent

Intent有多个构造函数重载 其中一个是

```kotlin
Intent(Context packageContext, Class<?> cls) 
```

第一个参数：启动Activity的上下文

第二个参数：指定想要启动的目标Activity

Activity类中提供了一个**startActivity( )**方法 专门用于启动**Activity** 接受一个**Intent**参数

```kotlin
            val intent = Intent(this, SecondActivity::class.java)
            startActivity(intent)
```



### 隐式Intent

#### 基础

首先修改 **src/main/AndroidManifest.xml** 

```xml
        <activity android:name=".SecondActivity">
            <intent-filter>
                <!--表明当前Activity 可以响应ACTION_START这个action-->
                <action android:name="com.example.activitytest.ACTION_START" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </activity>
```

然后修改 **src/main/java/com/example/activitytest/FirstActivity.kt**

由于category设置为缺省值 不用额外匹配

```kotlin
            val intent = Intent("com.example.activitytest.ACTION_START")
            startActivity(intent)
```



#### 更多用法

##### 启动其他程序的Activity

```kotlin
            val intent =Intent(Intent.ACTION_VIEW) //Intent.ACTION_VIEW是Android的一个内置动作
            intent.data = Uri.parse("https://www.baidu.com") //把字符串解析成Uri对象
            startActivity(intent)
```

```kotlin
            val intent =Intent(Intent.ACTION_VIEW)
            intent.data = Uri.parse("tel:18121448120")
            startActivity(intent)
```



##### 新建一个Activity 能够响应打开网页的Intent

修改 **src/main/AndroidManifest.xml**

忽略 **tools:ignore="AppLinkUrlError"** 因为会涉及到后面的知识

```xml
        <activity android:name=".ThirdActivity">
            <intent-filter tools:ignore="AppLinkUrlError">
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <data android:scheme="https" />
            </intent-filter>
        </activity>
```

### 向下一个Activity传递数据

修改 **src/main/java/com/example/activitytest/FirstActivity.kt**

```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.first_layout)
        button1.setOnClickListener {
            val data ="Hello SecondActivity"
            val intent = Intent(this,SecondActivity::class.java)
            intent.putExtra("extra_data",data) //前键后值
            startActivity(intent)
        }
```

修改 **src/main/java/com/example/activitytest/SecondActivity.kt**

```kotlin
class SecondActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.second_layout)

        val extraData = intent.getStringExtra("extra_data")
        Log.d("SecondActivity", "extra data is $extraData")
    }
}
```

### 向上一个Activity返回数据

第一步 修改 **src/main/java/com/example/activitytest/FirstActivity.kt**

```kotlin
class FirstActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.first_layout)
        //change here
        button1.setOnClickListener {
            val intent = Intent(this, SecondActivity::class.java)
            startActivityForResult(intent,1) //第二个参数是请求码 用于在回调中判断数据来源
        }
    }
```



第二步 修改 **src/main/java/com/example/activitytest/SecondActivity.kt**

```kotlin
class SecondActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.second_layout)
		//change here
        button2.setOnClickListener {
            val intent=Intent()
            intent.putExtra("data_return","Hello FirstActivity")
            setResult(Activity.RESULT_OK,intent) //第一个参数用于向上一个Activity返回处理结果 第二个参数把才有数据的intent返回去
            finish()
        }
    }
}
```



第三步 修改 **src/main/java/com/example/activitytest/FirstActivity.kt**

```kotlin
    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        when (requestCode) {
            1 -> if (resultCode == Activity.RESULT_OK) {
                val returnedData = data?.getStringExtra("data_return")
                Log.d("FirstActivity","returned data is $returnedData")
            }
        }
    }
```

onActivityResult()的三个参数

第一个：请求码 区分来自哪个上一级Activity

第二个：返回数据时的处理结果

第三个：带着数据的intent



那万一用户按屏幕上的 **BACK** 按键返回而不是我们设置的button呢

修改 **src/main/java/com/example/activitytest/SecondActivity.kt**

```kotlin
    override fun onBackPressed() {
        val intent =Intent()
        intent.putExtra("data_return", "Hello FirstActivity")
        setResult(Activity.RESULT_OK,intent)
        finish()
    }
```



## 3.4 Activity的生命周期

Activity被存放在栈中 称为返回栈(back stack)

运行状态：位于栈顶 

暂停状态：不在栈顶但仍然可见 

停止状态：不在栈顶也不可见 

销毁状态：不在栈中

### 七大Activity回调方法

**onCreate ()** 在Activity第一次被创建时调用

**onStart ()** 由不可见变为可见时调用

**onResume ()** 位于栈顶 处于运行状态

**onPause ()** 准备启动或恢复另一个Activity时调用 该方法执行速度一定要

**onStop ()** Activity**完全**不可见时调用  

**onDestory ()** 被销毁前调用

**onRestart ()** 顾名思义 由停止变为运行状态时调用

### 生命周期基础

**onCreate**和**onDestory** 之间是完整生存期

**onStart**和**onStop** 之间是可见生存期

**onResume**和**onPause** 之间是前台生存期

![生命周期示意图](https://developer.android.com/guide/components/images/activity_lifecycle.png)





### Activity被回收了怎么办

回收前可以保留临时数据

**onSaveInstanceState ( )** 携带一个Bundle类型参数 用于保存数据



## 3.5 Activity的启动模式

在 **src/main/AndroidManifest.xml** 中通过给 **<activity>** 标签指定 **android:launchMode** 来选择启动模式

### standard

缺省值

该模式下系统不在乎Activity是否已经在BackStack中 每次启动都会创建一个新的

![standard](https://s1.ax1x.com/2020/07/16/UrA8fJ.png)

### singleTop

该模式下系统判断Activity是否已经在BackStack的**栈顶** 在则不创建新的 不在则创建新的

![singleTop](https://s1.ax1x.com/2020/07/16/UrEDbV.png)

### singleTask

该模式下系统判断Activity是否已经在BackStack中 

在则使用它 并把它之上的Activity统统出栈

不在就创建新的

![singleTask](https://s1.ax1x.com/2020/07/16/UrE65F.png)

### singleInstance

启用新的返回栈来管理这个Activity

![singleInstance](https://s1.ax1x.com/2020/07/16/UrE28J.png)

## 3.6几种实践技巧

### 知晓当前在哪一个Activity

简单来说就是

创建一个继承自 **AppCompatActivity** 的 **BaseActivity**

让他在被创建时 Log.d 出当前Activity的名字

让所有别的Activity 成为他的子类

### 随时随地退出程序

~~Home键+上滑不香么~~

简单来说就是

创建一个单例类Collector 所有Activity在创建时被add进该类的list里 (我们有BaseActivity 所以这是容易实现的)

从而实现统一化管理

```kotlin
object ActivityCollector {
    private val activities = ArrayList<Activity>()
    fun addActivity(activity: Activity) {
        activities.add(activity)
    }

    fun removeActivity(activity: Activity) {
        activities.remove(activity)
    }

    fun finishAll() {
        for (activity in activities) {
            if (!activity.isFinishing) {
                activity.finish()
            }
        }
        activities.clear()
        android.os.Process.killProcess(android.os.Process.myPid())
    }
}
```

### 启动Activity的最佳写法

这样的好处在于方便别人启动时明白需要什么数据

在合作工程中建议这么干

```kotlin
class SecondActivity : BaseActivity() {
    //Something

    companion object {
        fun actionStart(context: Context, data1: String, data2: String) {
            val intent = Intent(context, SecondActivity::class.java)
            intent.putExtra("data1", data1)
            intent.putExtra("data2", data2)
            context.startActivity(intent)
        }
    }
}
```



# Kotlin拓展

## 标准函数 with, run和apply





### with

```kotlin
val result = with(obj) {
    // obj的上下文
    "value" //with的返回值
}
```

i.e.

```kotlin
val list = listOf(1,2,3,4)
val builder = StringBuilder()
builder.append("Start!\n")
for (num in list) {
    builder.append(num).append("\n")
}
builder.append("Over!\n")
val res=builder.toString()
println(res)
```

可以简化为

```kotlin
val list = listOf(1,2,3,4)
val res = with(StringBuilder()) {
    append("Start!\n")
    for (num in list) {
        append(num).append("\n")
    }
    append("Over!\n")
    toString()
}
println(res)
```

### run

与with基本类似

i.e

```kotlin
val list = listOf(1,2,3,4)
val res = StringBuilder().run {
    append("Start!\n")
    for (num in list) {
        append(num).append("\n")
    }
    append("Over!\n")
    toString()
}
println(res)
```

### apply

也很类似 但是只能返回对象本身 不能指定返回值

```kotlin
val list = listOf(1,2,3,4)
val res = StringBuilder().apply {
    append("Start!\n")
    for (num in list) {
        append(num).append("\n")
    }
    append("Over!\n")
}
println(res.toString())
```



## 定义静态方法

静态方法指不需要创建实例就能调用的方法

但是Kotlin弱化了这一概念 因为更加推荐单例类

**companion object { //Something }**  支持了类似静态方法调用的写法

如果一定要真正的静态方法 Two ways

### 注解

在**companion object** 的一个方法前面加上 **@JvmStatic**

注意 这一注解只能加在单例类 和 **companion object** 里

### 顶层方法

简单来说 就是新建一个kt文件 里面写的就是顶层方法了

在Java中调用该方法时需要 **file_name.function_name**
