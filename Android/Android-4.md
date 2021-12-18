---
title: Android-4
categories:
- Android
---

基于《第一行代码——Android》@郭霖



# 第四章 UI开发

## 4.2 常用控件

### TextView

**android:id** 唯一标识符

**android:layout_width** 和 **android:layout_height** 宽度高度 

它有三种参数 **wrap_content** **match_parent** **固定值 dp** 顾名思义 不解释

**android:gravity** 文字的对齐方式

**android:textColor** 和 **android:textSize** 文字颜色和大小 大小单位是sp

**android:textAllCaps** 文字大小写



### Button

之前一直在用 不赘述

值得注意的是Button里的文字默认大写



### EditText

允许用户输入编辑内容 并让程序处理

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >
    
	<!--Something...-->
    
    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/editText"
        android:hint="Type sth. here"
        android:maxLines="2"/> 

</LinearLayout>
```



### ImageView

在 **res** 下新建 **drawable-xxhdpi** 文件夹 图片丢进去

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >

    <ImageView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/imageView"
        android:src="@drawable/img1"/>

</LinearLayout>
```



### ProgressBar

用于显示一个进度条 表示正在加载一些数据

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >

    <ProgressBar
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/progressBar"
        />

</LinearLayout>
```

那如何在加载完以后让进度条消失呢

**android:visibility** 属性 

visible 可见 占屏幕

invisible 不可见 占屏幕

gone 不可见 不占屏 

i.e. 点击后进度条跑路 

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        button.setOnClickListener {
            if (progressBar.visibility == View.VISIBLE) {
                progressBar.visibility = View.GONE
            } else {
                progressBar.visibility = View.VISIBLE
            }

        }
    }
}
```

i.e. 点击推进的水平进度条

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >

    <ProgressBar
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/progressBar"
        style="?android:attr/progressBarStyleHorizontal"
        android:max="100"
        />

</LinearLayout>
```



```kotlin
class MainActivity : AppCompatActivity() {
    private var pic = 1
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        button.setOnClickListener {
            progressBar.progress+=10
        }
    }
}
```



### AlertDialog

```kotlin
class MainActivity : AppCompatActivity() {
    private var pic = 1
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        button.setOnClickListener {
            AlertDialog.Builder(this).apply {
                setTitle("This is Dialog")
                setMessage("Something important")
                setCancelable(false)
                setPositiveButton("OK") {dialog, which ->} //lambda 表达式是函数最后一个参数可以放在外面
                setNegativeButton("Cancel") { dialog, which ->}
                show()
            }
        }
    }
}
```



## 4.3 三种基本布局

### LinearLayout

**android:orientation** 指定水平或者竖直排列

**android:layout_gravity** 指定控件在布局中的对齐方式 与 **android:gravity** 区分开

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="top"
        android:id="@+id/button1"
        android:text="Button 1" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_vertical"
        android:id="@+id/button2"
        android:text="Button 2" />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom"
        android:id="@+id/button3"
        android:text="Button 3" />

</LinearLayout>
```



**android:layout_weight** 给各个部件分配权重

以下是一种适配各种屏幕的两控件横向布局

因为一个控件 **设置了android:layout_weight** 一个依然是 **wrap_content**

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >
    <EditText
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:hint="Type something"
        android:id="@+id/input_message"
        />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Send"
        android:id="@+id/send"
        />

</LinearLayout>
```



### RelativeLayout

所有的标签 都可以说 顾名思义

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/button1"
        android:layout_alignParentLeft="true"
        android:layout_alignParentTop="true"
        android:text="Button 1"
        />

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/button2"
        android:layout_below="@id/button1"
        android:layout_toRightOf="@id/button1"
        android:text="Button 2"
        />
</RelativeLayout>
```



![RelativeLayout](https://s1.ax1x.com/2020/07/17/U6rT1K.png)

### FrameLayout

又叫作帧布局 

全窝在左上角 应用场景很少

不多作解释



## 4.4自定义控件

![常用控件和布局继承结构](https://s1.ax1x.com/2020/07/17/U67iuj.png)

### 引入布局

新建 **src/main/res/layout/title.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="@drawable/title_bg">

    <Button
        android:id="@+id/titleBack"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:layout_margin="5dp"
        android:background="@drawable/back_bg"
        android:text="Back"
        android:textColor="#fff" />

    <TextView
        android:id="@+id/titleText"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:gravity="center"
        android:layout_weight="1"
        android:text="Title text"
        android:textColor="#fff"
        android:textSize="24sp" />

    <Button
        android:id="@+id/titleEdit"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:layout_margin="5dp"
        android:background="@drawable/edit_bg"
        android:text="Edit"
        android:textColor="#fff" />

</LinearLayout>

```

在 **activity_main.xml** 中 **include**

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <include layout="@layout/title" />

</LinearLayout>
```

别忘了 **MainActivity.kt** 的 **onCreate** 里去掉原有的标题栏

```kotlin
supportActionBar?.hide()
```

~~有内味儿了~~

![title_layout](https://s1.ax1x.com/2020/07/17/U6Oec8.png)



### 创建自定义控件

新建 **src/main/java/com/example/uiwidgettest/TitleLayout.kt**

**LayoutInflater.from** 方法可以构建出一个 **LayoutInflater** 对象

```kotlin
class TitleLayout(context: Context, attrs: AttributeSet) : LinearLayout (context,attrs) {
    init {
        LayoutInflater.from(context).inflate(R.layout.title, this) //前一个参数是布局文件id，后一个是父布局
    }
}
```

在 **activity_main.xml** 中添加

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.example.uiwidgettest.TitleLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        />

</LinearLayout>
```



为标题栏中的按钮注册事件 修改 **src/main/java/com/example/uiwidgettest/TitleLayout.kt**

```kotlin
class TitleLayout(context: Context, attrs: AttributeSet) : LinearLayout (context,attrs) {
    init {
        LayoutInflater.from(context).inflate(R.layout.title, this)
        //changs
        titleBack.setOnClickListener {
            val activity = context as Activity //as 是强制类型转换
            activity.finish()
        }
        titleEdit.setOnClickListener {
            Toast.makeText(context,"You clicked Edit button", Toast.LENGTH_SHORT).show()
        }
    }
}
```



## 4.5最难用也是最常用的ListView

### 先来简单用法

首先常规操作 **src/main/res/layout/activity_main.xml** 中添加

接着借助 **适配器** 把数据传给 **ListView**

```kotlin
class MainActivity : AppCompatActivity() {
    private val data = listOf("apple","banana","orange","watermelon","pear","grape","pineapple","strawberry","cherry",
    "mango","apple","banana","orange","watermelon","pear","grape","pineapple","strawberry","cherry", "mango")

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        //add two lines
        val adapter = ArrayAdapter<String>(this,android.R.layout.simple_list_item_1,data) //data 是String的数组
        listView.adapter = adapter
    }
}
```



### 定制ListView界面

Step1: **src/main/java/com/example/listviewtest/Fruit.kt**

作为ListView适配器的适配类型

```kotlin
class Fruit (val name:String, val imageId: Int)
```



Step2: **src/main/res/layout/fruit_item.xml**

为ListView的子项指定一个自定义布局

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="60dp">

    <ImageView
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:id="@+id/fruitImage"
        android:layout_gravity="center_vertical"
        android:layout_marginLeft="10dp"
        />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/fruitName"
        android:layout_gravity="center_vertical"
        android:layout_marginLeft="10dp"
        />

</LinearLayout>
```



Step3: **src/main/java/com/example/listviewtest/FruitAdapter.kt**

**getView()** 在子项被滚动到屏幕内的时候会被调用

```kotlin
class FruitAdapter(activity: Activity, val resourceId: Int, data: List<Fruit>) : ArrayAdapter<Fruit> (activity,resourceId,data) {
    override fun getView(position: Int, convertView: View?, parent: ViewGroup): View {
        val view = LayoutInflater.from(context).inflate(resourceId,parent,false)
        val fruitImage:ImageButton = view.findViewById(R.id.fruitImage)
        val fruitName: TextView = view.findViewById(R.id.fruitName)
        val fruit = getItem(position)
        if (fruit != null) {
            fruitImage.setImageResource(fruit.imageId)
            fruitName.text = fruit.name
        }
        return view
    }
}
```



Step4: **src/main/java/com/example/listviewtest/MainActivity.kt**

```kotlin
class MainActivity : AppCompatActivity() {

    private val fruitList = ArrayList<Fruit>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        initFruits()
        val adapter = FruitAdapter(this,R.layout.fruit_item,fruitList)
        listView.adapter = adapter
    }

    private fun initFruits() {
        repeat(2) {
            fruitList.add(Fruit("Apple",R.drawable.img1))
            fruitList.add(Fruit("Banana",R.drawable.img2))
            fruitList.add(Fruit("Orange",R.drawable.img3))
            fruitList.add(Fruit("Watermelon",R.drawable.img4))
            fruitList.add(Fruit("Pear",R.drawable.img5))
            fruitList.add(Fruit("Grape",R.drawable.img6))
            fruitList.add(Fruit("Pineapple",R.drawable.img7))
            fruitList.add(Fruit("Strawberry",R.drawable.img8))
            fruitList.add(Fruit("Cherry",R.drawable.img9))
            fruitList.add(Fruit("Mango",R.drawable.img10))
        }
    }
}
```

### 提升运行效率

修改 **src/main/java/com/example/listviewtest/FruitAdapter.kt**

```kotlin
class FruitAdapter(activity: Activity, val resourceId: Int, data: List<Fruit>) : ArrayAdapter<Fruit> (activity,resourceId,data) {
    override fun getView(position: Int, convertView: View?, parent: ViewGroup): View {
        //利用convertView参数将之前加载好的布局进行缓存
        val view:View
        if (convertView == null) {
            view = LayoutInflater.from(context).inflate(resourceId,parent,false)
        } else {
            view = convertView
        }


        val fruitImage:ImageView = view.findViewById(R.id.fruitImage)
        val fruitName: TextView = view.findViewById(R.id.fruitName)
        val fruit = getItem(position)
        if (fruit != null) {
            fruitImage.setImageResource(fruit.imageId)
            fruitName.text = fruit.name
        }
        return view
    }
}
```



进一步优化

使得所有控件实例缓存在ViewHolder里 不用每次findViewById 了

```kotlin
class FruitAdapter(activity: Activity, val resourceId: Int, data: List<Fruit>) : ArrayAdapter<Fruit> (activity,resourceId,data) {
    inner class ViewHolder(val fruitImage:ImageView, val fruitName:TextView)

    override fun getView(position: Int, convertView: View?, parent: ViewGroup): View {
        val view:View
        val viewHolder: ViewHolder
        if (convertView == null) {
            view = LayoutInflater.from(context).inflate(resourceId,parent,false)
            val fruitImage:ImageView = view.findViewById(R.id.fruitImage)
            val fruitName: TextView = view.findViewById(R.id.fruitName)
            viewHolder=ViewHolder(fruitImage,fruitName)
            view.tag = viewHolder
        } else {
            view = convertView
            viewHolder=view.tag as ViewHolder
        }



        val fruit = getItem(position)
        if (fruit != null) {
            viewHolder.fruitImage.setImageResource(fruit.imageId)
            viewHolder.fruitName.text = fruit.name
        }
        return view
    }
}
```



### 点击事件

注册子项点击事件 **position** 告知点击了哪个子项

```kotlin
class MainActivity : AppCompatActivity() {
    private val fruitList = ArrayList<Fruit>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        initFruits()
        val adapter = FruitAdapter(this, R.layout.fruit_item, fruitList)
        listView.adapter = adapter
        //changes
        listView.setOnItemClickListener { _, _, position, _ ->
            val fruit = fruitList[position]
            Toast.makeText(this, fruit.name, Toast.LENGTH_SHORT).show()
        }
    }

    private fun initFruits() {
        repeat(2) {
            fruitList.add(Fruit("Apple", R.drawable.img1))
            fruitList.add(Fruit("Banana", R.drawable.img2))
            fruitList.add(Fruit("Orange", R.drawable.img3))
            fruitList.add(Fruit("Watermelon", R.drawable.img4))
            fruitList.add(Fruit("Pear", R.drawable.img5))
            fruitList.add(Fruit("Grape", R.drawable.img6))
            fruitList.add(Fruit("Pineapple", R.drawable.img7))
            fruitList.add(Fruit("Strawberry", R.drawable.img8))
            fruitList.add(Fruit("Cherry", R.drawable.img9))
            fruitList.add(Fruit("Mango", R.drawable.img10))
        }
    }
}
```



## 4.6RecyclerView 更强大的滚动控件

### 基本用法

#### 准备: 

在 **app/build.gradle** 中引入 RecyclerView 库



#### Step1:

老规矩 **src/main/res/layout/activity_main.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_height="match_parent"
    android:layout_width="match_parent">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```



#### Step2:

创建 **src/main/java/com/example/listviewtest/FruitAdapter.kt**

```kotlin
class FruitAdapter(val fruitList: List<Fruit>) :
    RecyclerView.Adapter<FruitAdapter.ViewHolder>() {
    inner class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val fruitImage: ImageView = view.findViewById(R.id.fruitImage)
        val fruitName: TextView = view.findViewById(R.id.fruitName)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.fruit_item, parent, false)
        return ViewHolder(view)
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val fruit = fruitList[position]
        holder.fruitImage.setImageResource(fruit.imageId)
        holder.fruitName.text = fruit.name
    }

    override fun getItemCount() = fruitList.size
}
```



这是RecyclerView适配器的标准写法

首先定义了一个内部类 继承自 **RecyclerView.ViewHolder** 

由于FruitAdapter继承自 **RecyclerView.Adapter** ，之后三个函数是必须重写的

第一个创建ViewHolder实例

第二个对子项数据赋值 在子项滚动如屏幕时执行

第三个告诉 RecyclerView 一共有多少子项



#### Step3：

准备好了适配器 那就开始使用他

修改 **src/main/java/com/example/listviewtest/MainActivity.kt**

```kotlin
package com.example.listviewtest

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.widget.ArrayAdapter
import android.widget.Toast
import androidx.recyclerview.widget.LinearLayoutManager
import kotlinx.android.synthetic.main.activity_main.*

class MainActivity : AppCompatActivity() {
    private val fruitList = ArrayList<Fruit>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        initFruits()
        //changes
        val layoutManger = LinearLayoutManager(this)
        recyclerView.layoutManager = layoutManger
        val adapter = FruitAdapter(fruitList)
        recyclerView.adapter = adapter
    }

    private fun initFruits() {
        repeat(2) {
            fruitList.add(Fruit("Apple", R.drawable.img1))
            fruitList.add(Fruit("Banana", R.drawable.img2))
            fruitList.add(Fruit("Orange", R.drawable.img3))
            fruitList.add(Fruit("Watermelon", R.drawable.img4))
            fruitList.add(Fruit("Pear", R.drawable.img5))
            fruitList.add(Fruit("Grape", R.drawable.img6))
            fruitList.add(Fruit("Pineapple", R.drawable.img7))
            fruitList.add(Fruit("Strawberry", R.drawable.img8))
            fruitList.add(Fruit("Cherry", R.drawable.img9))
            fruitList.add(Fruit("Mango", R.drawable.img10))
        }
    }
}
```



### 实现横向滚动和瀑布流布局

现在来整点ListView整不了的好活

#### 横向滚动

首先对 **src/main/res/layout/fruit_item.xml** 修改 因为现在还是水平排列的

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="80dp"
    android:layout_height="wrap_content"
    android:orientation="vertical">

    <ImageView
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:id="@+id/fruitImage"
        android:layout_gravity="center_horizontal"
        android:layout_marginTop="10dp"
        />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/fruitName"
        android:layout_gravity="center_horizontal"
        android:layout_marginTop="10dp"
        />

</LinearLayout>

```



修改 **src/main/java/com/example/listviewtest/MainActivity.kt**

```kotlin
package com.example.listviewtest

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.widget.ArrayAdapter
import android.widget.Toast
import androidx.recyclerview.widget.LinearLayoutManager
import kotlinx.android.synthetic.main.activity_main.*

class MainActivity : AppCompatActivity() {
    private val fruitList = ArrayList<Fruit>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        initFruits()
        val layoutManger = LinearLayoutManager(this)
        //just add one line
        layoutManger.orientation = LinearLayoutManager.HORIZONTAL
        recyclerView.layoutManager = layoutManger
        val adapter = FruitAdapter(fruitList)
        recyclerView.adapter = adapter
    }

    private fun initFruits() {
        repeat(2) {
            fruitList.add(Fruit("Apple", R.drawable.img1))
            fruitList.add(Fruit("Banana", R.drawable.img2))
            fruitList.add(Fruit("Orange", R.drawable.img3))
            fruitList.add(Fruit("Watermelon", R.drawable.img4))
            fruitList.add(Fruit("Pear", R.drawable.img5))
            fruitList.add(Fruit("Grape", R.drawable.img6))
            fruitList.add(Fruit("Pineapple", R.drawable.img7))
            fruitList.add(Fruit("Strawberry", R.drawable.img8))
            fruitList.add(Fruit("Cherry", R.drawable.img9))
            fruitList.add(Fruit("Mango", R.drawable.img10))
        }
    }
}
```





为什么ListView做不到而RecyclerView可以呢 ~~这个ListView他就是逊啦~~

RecyclerView的LayoutManager制定一套可扩展的布局排列接口 ~~我LayoutManager超勇的~~

#### 瀑布流布局

现在就用这个超勇的~~阿伟~~LayoutManager整点更酷的好活

首先对 **src/main/res/layout/fruit_item.xml** 修改

修改了LinearLayout的宽度 因为瀑布流根据布局的列数自动适配

修改了LinearLayout的margin 让子项之间有点距离

修改了TextView的gravity 因为等下会把文字长度变长 居中就会怪怪的

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:layout_margin="5dp">

    <ImageView
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:id="@+id/fruitImage"
        android:layout_gravity="center_horizontal"
        android:layout_marginTop="10dp"
        />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/fruitName"
        android:layout_gravity="left"
        android:layout_marginTop="10dp"
        />

</LinearLayout>
```



接着修改 **src/main/java/com/example/listviewtest/MainActivity.kt**

为了展现出效果 创建了**getRandomLengthString**函数实现字体不同长度

```kotlin
package com.example.listviewtest

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.widget.ArrayAdapter
import android.widget.Toast
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.StaggeredGridLayoutManager
import kotlinx.android.synthetic.main.activity_main.*
import java.lang.StringBuilder

class MainActivity : AppCompatActivity() {
    private val fruitList = ArrayList<Fruit>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        initFruits()
        //第一个参数表示分成多少列 第二个是方向
        val layoutManger = StaggeredGridLayoutManager(3, StaggeredGridLayoutManager.VERTICAL)
        recyclerView.layoutManager = layoutManger
        val adapter = FruitAdapter(fruitList)
        recyclerView.adapter = adapter
    }

    private fun initFruits() {
        repeat(2) {
            fruitList.add(Fruit(getRandomLengthString("picture_one"),R.drawable.img1))
            fruitList.add(Fruit(getRandomLengthString("picture_two"), R.drawable.img2))
            fruitList.add(Fruit(getRandomLengthString("picture_three"), R.drawable.img3))
            fruitList.add(Fruit(getRandomLengthString("picture_four"), R.drawable.img4))
            fruitList.add(Fruit(getRandomLengthString("picture_five"), R.drawable.img5))
            fruitList.add(Fruit(getRandomLengthString("picture_six"), R.drawable.img6))
            fruitList.add(Fruit(getRandomLengthString("picture_seven"), R.drawable.img7))
            fruitList.add(Fruit(getRandomLengthString("picture_eight"), R.drawable.img8))
            fruitList.add(Fruit(getRandomLengthString("picture_nine"), R.drawable.img9))
            fruitList.add(Fruit(getRandomLengthString("picture_ten"), R.drawable.img10))
        }
    }

    private fun getRandomLengthString(str:String) : String {
        val n = (1..20).random()
        val builder = StringBuilder()
        repeat(n) {
            builder.append(str)
        }
        return builder.toString()
    }
}
```

效果图:

![瀑布流布局](https://s1.ax1x.com/2020/07/19/UW1VH0.png)



### 点击事件

RecyclerView 没有提供类似于 **setOnClickListener** 的注册监听器方法

需要我们自己给子项具体的View注册事件

修改 **src/main/java/com/example/listviewtest/FruitAdapter.kt**

```kotlin
class FruitAdapter(val fruitList: List<Fruit>) :
    RecyclerView.Adapter<FruitAdapter.ViewHolder>() {
    inner class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val fruitImage: ImageView = view.findViewById(R.id.fruitImage)
        val fruitName: TextView = view.findViewById(R.id.fruitName)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.fruit_item, parent, false)
        //changes
        val viewHolder = ViewHolder(view)
        viewHolder.itemView.setOnClickListener {
            val position = viewHolder.adapterPosition
            val fruit = fruitList[position]
            Toast.makeText(parent.context, "You clicked view ${fruit.name}",Toast.LENGTH_SHORT).show()
        }
        viewHolder.fruitImage.setOnClickListener {
            val position = viewHolder.adapterPosition
            val fruit = fruitList[position]
            Toast.makeText(parent.context, "You clicked image ${fruit.name}",Toast.LENGTH_SHORT).show()
        }
        return viewHolder
        //changes end
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val fruit = fruitList[position]
        holder.fruitImage.setImageResource(fruit.imageId)
        holder.fruitName.text = fruit.name
    }

    override fun getItemCount() = fruitList.size
}
```



## 4.7实践编写一个聊天界面

### 制作9-Patch图片

原图片右键Create 9-Patch file

编辑界面下在四个边界拉小黑条

最后把原图删除

### 编写界面

主界面

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#d8e0e8"
    android:orientation="vertical">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <EditText
            android:id="@+id/inputText"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:hint="Type sth here"
            android:maxLines="2" />

        <Button
            android:id="@+id/send"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Send" />

    </LinearLayout>


</LinearLayout>
```



Msg类

```kotlin
package com.example.uibestpratice

class Msg(val context: String, val type: Int) {
    companion object {
        const val TYPE_RECEIVED = 0
        const val TYPE_SENT = 1
    }

}
```



子项布局

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="10dp">

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="left"
        android:background="@drawable/message_left">

        <TextView
            android:id="@+id/leftMsg"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:layout_margin="10dp"
            android:textColor="#fff" />

    </LinearLayout>

</FrameLayout>

```



```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="10dp">

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:background="@drawable/message_right">

        <TextView
            android:id="@+id/rightMsg"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:layout_margin="10dp"
            android:textColor="#000" />

    </LinearLayout>

</FrameLayout>

```



RecyclerView适配器

```kotlin
package com.example.uibestpratice

import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView
import java.lang.IllegalArgumentException

class MsgAdapter(val msgList: List<Msg>) : RecyclerView.Adapter<RecyclerView.ViewHolder>() {

    inner class LeftViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val leftMsg: TextView = view.findViewById(R.id.leftMsg)
    }

    inner class RightViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val rightMsg: TextView = view.findViewById(R.id.rightMsg)
    }

    override fun getItemViewType(position: Int): Int {
        val msg = msgList[position]
        return msg.type
    }

    //奇怪的新写法增加了
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int) =
        if (viewType == Msg.TYPE_RECEIVED) {
            val view =
                LayoutInflater.from(parent.context).inflate(R.layout.msg_left_item, parent, false)
            LeftViewHolder(view)
        } else {
            val view =
                LayoutInflater.from(parent.context).inflate(R.layout.msg_right_item, parent, false)
            RightViewHolder(view)
        }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        val msg = msgList[position]
        when (holder) {
            is LeftViewHolder -> holder.leftMsg.text = msg.context
            is RightViewHolder -> holder.rightMsg.text = msg.context
            else -> throw  IllegalArgumentException()
        }
    }

    override fun getItemCount() = msgList.size
}
```



最后部署

```kotlin
package com.example.uibestpratice

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.view.View
import android.widget.LinearLayout
import androidx.recyclerview.widget.LinearLayoutManager
import kotlinx.android.synthetic.main.activity_main.*
import kotlin.math.sign

class MainActivity : AppCompatActivity(), View.OnClickListener {

    private val msgList = ArrayList<Msg>()

    private var adapter: MsgAdapter? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        initMsg()
        val layoutManager = LinearLayoutManager(this)
        recyclerView.layoutManager = layoutManager
        adapter = MsgAdapter(msgList)
        recyclerView.adapter = adapter
        send.setOnClickListener(this)
    }

    override fun onClick(p0: View?) {
        when (p0) {
            send -> {
                val content = inputText.text.toString()
                if (content.isNotEmpty()) {
                    val msg = Msg(content, Msg.TYPE_SENT)
                    msgList.add(msg)
                    //这两个方法 不是很清楚
                    adapter?.notifyItemInserted(msgList.size - 1) //当有新消息 刷新显示
                    recyclerView.scrollToPosition(msgList.size - 1) //定位到最后一行
                    inputText.setText("")
                }
            }
        }
    }

    private fun initMsg() {
        val msg1 = Msg("他就是逊啦", Msg.TYPE_RECEIVED)
        msgList.add(msg1)
        val msg2 = Msg("这么说，你很勇哦", Msg.TYPE_SENT)
        msgList.add(msg2)
        val msg3 = Msg("我超勇的啦", Msg.TYPE_RECEIVED)
        msgList.add(msg3)
    }
}
```





# Kotlin拓展

## 对变量延迟初始化

看看上面聊天界面项目 MainActivity中的adapter变量

初始化为null 所以后面需要大量判空操作

可以先延迟初始化

```kotlin
lateinit var adapter: MsgAdapter
```

判断变量有没有初始化 ~~小编也觉得很鬼畜，但事实就是这样~~ 

```kotlin
::adapter.isInitialized
```



## 使用密封类优化代码

使用密封类 我们发现 when中不再需要写else

因为编译器会自动检测改密封类有哪些子类 并强制要求对每一个编写反应

```kotlin
sealed class Result
class Success(val msg: String) : Result()
class Failure(val error:Exception) : Result()

fun getResultMsg(result: Result) = when(result) {
    is Success -> result.msg
    is Failure -> result.error.message
}
```



接下来看看它如何优化MsgAdapter 中的 ViewHolder

创建 **src/main/java/com/example/uibestpratice/MsgViewHolder.kt**

```kotlin
sealed class MsgViewHolder(view: View) : RecyclerView.ViewHolder(view)

class LeftViewHolder(view: View) : MsgViewHolder(view) {
    val leftMsg: TextView = view.findViewById(R.id.leftMsg)
}

class RightViewHolder(view: View) : MsgViewHolder(view) {
    val rightMsg: TextView = view.findViewById(R.id.rightMsg)
}
```



修改 **src/main/java/com/example/uibestpratice/MsgAdapter.kt**

```kotlin
class MsgAdapter(val msgList: List<Msg>) : RecyclerView.Adapter<MsgViewHolder>() { //change1  MsgViewHolder

    override fun getItemViewType(position: Int): Int {
        val msg = msgList[position]
        return msg.type
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int) =
        if (viewType == Msg.TYPE_RECEIVED) {
            val view =
                LayoutInflater.from(parent.context).inflate(R.layout.msg_left_item, parent, false)
            LeftViewHolder(view)
        } else {
            val view =
                LayoutInflater.from(parent.context).inflate(R.layout.msg_right_item, parent, false)
            RightViewHolder(view)
        }

    override fun onBindViewHolder(holder: MsgViewHolder, position: Int) { //change2 holder: MsgViewHolder
        val msg = msgList[position]
        when (holder) {
            is LeftViewHolder -> holder.leftMsg.text = msg.context
            is RightViewHolder -> holder.rightMsg.text = msg.context
            //change 3 delete else condition
        }
    }

    override fun getItemCount() = msgList.size
}
```













 