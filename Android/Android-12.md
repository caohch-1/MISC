---
title: Android-12
categories:
- Android


---

基于《第一行代码——Android》@郭霖



# 第十二章：最佳的UI体验，Material Design

## 12.1 简介

一言蔽之，好康！！！~~颜狗狂喜~~



## 12.2 Toolbar

首先把**src/main/res/values/styles.xml**中的

**style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar"**修改为 

**style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar"**

现在就把 **ActionBar**隐藏起来了，接下来使用 **ToolBar**替换 **ActionBar**



然后修改 **activity_main.xml**

**android:theme**指定Toolbar主题

**app:popupTheme**指定Toolbar菜单项主题

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="@color/colorPrimary"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
        app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />

</FrameLayout>
```



最后修改 **MainActivity**

```kotlin
pclass MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
		//add one line here
        setSupportActionBar(toolbar)
    }
}
```





接着学习Toolbar比较常用的功能

比如修改标题栏上的显示文字内容

注意到我们在 **src/main/AndroidManifest.xml**中的 **<activity>**中添加了**label**来达到目的

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.materialtest">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity"
            android:label="Fruits">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```



现在Toolbar还是很单调，我们添加一些action按钮

创建 **src/main/res/menu/toolbar.xml**

**<item>**标签定义action按钮

app:showAsAction 指定按钮的显示位置，主要有以下几个值可选

- always: 永远显示，除非屏幕空间不够
- ifRoom: 屏幕空间足够就显示在Toolbar中，不够的话就在菜单中
- never: 永远显示在菜单中

注意，Toolbar中只显示图标，菜单中只显示文字

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/backup"
        android:icon="@drawable/ic_backup"
        android:title="Backup"
        app:showAsAction="always" />
    <item
        android:id="@+id/delete"
        android:icon="@drawable/ic_delete"
        android:title="Delete"
        app:showAsAction="ifRoom" />
    <item
        android:id="@+id/settings"
        android:icon="@drawable/ic_settings"
        android:title="Settings"
        app:showAsAction="never" />
</menu>
```



然后修改下 **MainActivity.kt**，这里很常规，没啥好说的

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        setSupportActionBar(toolbar)
    }

    override fun onCreateOptionsMenu(menu: Menu?): Boolean {
        menuInflater.inflate(R.menu.toolbar, menu)
        return true
    }

    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        when (item.itemId) {
            R.id.backup -> Toast.makeText(this, "You clicked Backup", Toast.LENGTH_SHORT).show()
            R.id.delete -> Toast.makeText(this, "You clicked delete", Toast.LENGTH_SHORT).show()
            R.id.settings -> Toast.makeText(this, "You clicked Settings", Toast.LENGTH_SHORT).show()
        }
        return true
    }
}
```



## 12.3 滑动菜单

### DrawerLayout

**DrawerLayout**是一个布局，在布局中允许放入两个直接子控件：

1. 主屏幕中显示的内容
2. 滑动菜单中显示的内容



上手试试

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawerLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.appcompat.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="@color/colorPrimary"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />

    </FrameLayout>

    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:background="#FFF"
        android:text="This is menu"
        android:textSize="30sp" />

</androidx.drawerlayout.widget.DrawerLayout>
```



万一用户不知道有侧边栏可以滑动呢？

可以添加一个导航按键

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        setSupportActionBar(toolbar)
        supportActionBar?.let {
            it.setDisplayHomeAsUpEnabled(true) //让导航栏显示出来
            it.setHomeAsUpIndicator(R.drawable.ic_menu) //设置导航按钮图案
        }
    }

    override fun onCreateOptionsMenu(menu: Menu?): Boolean {
        menuInflater.inflate(R.menu.toolbar, menu)
        return true
    }

    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        when (item.itemId) {
            android.R.id.home -> drawerLayout.openDrawer(GravityCompat.START)  //显示滑动菜单，参数属性保持一致
            R.id.backup -> Toast.makeText(this, "You clicked Backup", Toast.LENGTH_SHORT).show()
            R.id.delete -> Toast.makeText(this, "You clicked delete", Toast.LENGTH_SHORT).show()
            R.id.settings -> Toast.makeText(this, "You clicked Settings", Toast.LENGTH_SHORT).show()
        }
        return true
    }
}
```



### NavigationView

首先，添加依赖

第二个库是用来实现图片圆形化

```
    implementation 'com.google.android.material:material:1.0.0'
    implementation 'de.hdodenhof:circleimageview:3.0.1'
```

在开始使用 **NavigationView**前还需要准备两个东西：menu和headerLayout

前者用来显示具体的菜单项，后者用来显示头部布局

先来准备menu

创建 **src/main/res/menu/nav_menu.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <group android:checkableBehavior="single">
        <item
            android:id="@+id/navCall"
            android:icon="@drawable/nav_call"
            android:title="Call" />
        <item
            android:id="@+id/navFriends"
            android:icon="@drawable/nav_friends"
            android:title="Friends" />
        <item
            android:id="@+id/navMail"
            android:icon="@drawable/nav_mail"
            android:title="Mail" />
        <item
            android:id="@+id/navTask"
            android:icon="@drawable/nav_task"
            android:title="Tasks" />
    </group>

</menu>
```



接着准备headerLayout

这是个可以随意定制的布局，这里简单起见，只放头像、用户名和邮箱地址

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="180dp"
    android:background="@color/colorPrimary"
    android:padding="10dp">

    <de.hdodenhof.circleimageview.CircleImageView
        android:id="@+id/iconImage"
        android:layout_width="70dp"
        android:layout_height="70dp"
        android:layout_centerInParent="true"
        android:src="@drawable/nav_icon" />

    <TextView
        android:id="@+id/mailText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:text="tonygreendev@gmail.com"
        android:textColor="#FFF"
        android:textSize="14sp" />

    <TextView
        android:id="@+id/userText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_above="@id/mailText"
        android:text="Tony Green"
        android:textColor="#FFF"
        android:textSize="14sp" />

</RelativeLayout>
```



终于能修改 **activity_main.kt**

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawerLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.appcompat.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="@color/colorPrimary"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />

    </FrameLayout>

    <com.google.android.material.navigation.NavigationView
        android:id="@+id/navView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        app:headerLayout="@layout/nav_header"
        app:menu="@menu/nav_menu" />

</androidx.drawerlayout.widget.DrawerLayout>
```



最后修改 **MainActivity.kt**

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        setSupportActionBar(toolbar)
        supportActionBar?.let {
            it.setDisplayHomeAsUpEnabled(true) //让导航栏显示出来
            it.setHomeAsUpIndicator(R.drawable.ic_menu) //设置导航按钮图案
        }
		//add four lines here
        navView.setCheckedItem(R.id.navCall) //将Call菜单项设置为默认选中
        navView.setNavigationItemSelectedListener {
            drawerLayout.closeDrawers()
            true
        }
    }
	//Something
}
```



## 12.4 悬浮按钮和可交互提示

###  FloatingActionButton

它的按钮点击事件和普通的没区别

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawerLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    	...Something

        <com.google.android.material.floatingactionbutton.FloatingActionButton
            android:id="@+id/fab"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="bottom|end"
            android:layout_margin="16dp"
            android:elevation="8dp"
            android:src="@drawable/ic_done" />

    </FrameLayout>

    ...Something

</androidx.drawerlayout.widget.DrawerLayout>
```



### Snackbar

更加先进的提示工具

Snackbar并不是Toast的替代品，Toast告诉用户发生了啥，但是用户只能被动接受，而Snackbar则在这方面进行了拓展，允许加入可交互按钮

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        //Something
		//change here
        fab.setOnClickListener { view ->
            Snackbar.make(view, "Data deleted", Snackbar.LENGTH_SHORT)
                .setAction("Undo") {
                    Toast.makeText(this, "Data restored", Toast.LENGTH_SHORT).show()
                }
                .show()
        }
    }
	//Something
}
```

然而我们发现弹出消息挡住了我们的悬浮按钮，咋办？

借助 **CoordinatorLayout**实现



### CoordinatorLayout

可以说是一个加强版的FrameLayout

把**activity_main.xml**中的**FrameLayout**换成**androidx.coordinatorlayout.widget.CoordinatorLayout**即可

会发现提示消息一出现，悬浮按钮也自动上移了



## 12.5 卡片式布局

### MaterialCardView

添加依赖，第一个不用说，第二个是一个超级强大的开源图片加载库

```
    implementation 'androidx.recyclerview:recyclerview:1.0.0'
    implementation 'com.github.bumptech.glide:glide:4.9.0'
```



首先在 **activity_main.xml**中添加一个 **recyclerView**控件，注意放在 **CoordinatorLayout**里

接着定义一个实体类 **Fruit**

```kotlin
class Fruit(val name: String, val imageId: Int)
```

理所当然接下来还要为**recyclerView**的子项指定自定义布局，创建 **src/main/res/layout/fruit_item.xml**

**ImageView**的**android:scaleType**指定缩放模式

```xml
<?xml version="1.0" encoding="utf-8"?>
<com.google.android.material.card.MaterialCardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="5dp"
    app:cardCornerRadius="4dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <ImageView
            android:id="@+id/fruitImage"
            android:layout_width="match_parent"
            android:layout_height="100dp"
            android:scaleType="centerCrop" />

        <TextView
            android:id="@+id/fruitName"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            android:layout_margin="5dp"
            android:textSize="16sp" />
    </LinearLayout>


</com.google.android.material.card.MaterialCardView>
```



接下来为 **RecycleView**准备一个适配器，新建 **FruitAdapter**类

```kotlin
class FruitAdapter(val context: Context, val fruitList: List<Fruit>) :
    RecyclerView.Adapter<FruitAdapter.ViewHolder>() {

    inner class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val fruitImage: ImageView = view.findViewById(R.id.fruitImage)
        val fruitName: TextView = view.findViewById(R.id.fruitName)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(context).inflate(R.layout.fruit_item, parent, false)
        return ViewHolder(view)
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val fruit = fruitList[position]
        holder.fruitName.text = fruit.name
        Glide.with(context).load(fruit.imageId)
            .into(holder.fruitImage) //load加载图片，into将图片设置到某ImageView中
    }

    override fun getItemCount() = fruitList.size
}
```



最后修改 **MainActivity.kt**

```kotlin
class MainActivity : AppCompatActivity() {

    val fruits = mutableListOf(
        Fruit("Apple", R.drawable.apple),
        Fruit("Banana", R.drawable.banana),
        Fruit("Orange", R.drawable.orange),
        Fruit("Watermelon", R.drawable.watermelon),
        Fruit("Pear", R.drawable.pear),
        Fruit("Grape", R.drawable.grape),
        Fruit("Pineapple", R.drawable.pineapple),
        Fruit("Strawberry", R.drawable.strawberry),
        Fruit("Cherry", R.drawable.cherry),
        Fruit("Mango", R.drawable.mango)
    )

    val fruitList = ArrayList<Fruit>()

    override fun onCreate(savedInstanceState: Bundle?) {
        
        //Something

        initFruits()
        val layoutManager = GridLayoutManager(this, 2)
        recyclerView.layoutManager = layoutManager
        val adapter = FruitAdapter(this, fruitList)
        recyclerView.adapter = adapter
    }

    private fun initFruits() {
        fruitList.clear()
        repeat(50) {
            val index = (0 until fruits.size).random()
            fruitList.add(fruits[index])
        }
    }

    //Something
    
}
```



问题来了，我们的Toolbar被挡住了

借助 **AppBarLayout**解决



### AppBarLayout

这个实际上是一个垂直方向的 **LinearLayout**但是内部做了很多的滚动事件的封装

两步就能解决问题

首先把 **Toolbar**嵌套到 **AppBarLayout**中

然后给 **RecyclerView**指定一个布局行为 **app:layout_behavior**

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawerLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.coordinatorlayout.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <com.google.android.material.appbar.AppBarLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

            <androidx.appcompat.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:background="@color/colorPrimary"
                android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />

        </com.google.android.material.appbar.AppBarLayout>

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/recyclerView"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_behavior="@string/appbar_scrolling_view_behavior" />

        ...

    </androidx.coordinatorlayout.widget.CoordinatorLayout>


    ...

</androidx.drawerlayout.widget.DrawerLayout>
```



进一步优化

对**Toolbar**指定 **app:layout_scrollFlags="scroll|enterAlways|snap"**

实现了下滑时隐藏Toolbar，上滚时出现



## 12.6 下拉刷新

**SwipeRefreshLayout**就是用于实现下拉刷新的核心类

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawerLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.coordinatorlayout.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <com.google.android.material.appbar.AppBarLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

           ...

        <androidx.swiperefreshlayout.widget.SwipeRefreshLayout
            android:id="@+id/swipeRefresh"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_behavior="@string/appbar_scrolling_view_behavior">

            <androidx.recyclerview.widget.RecyclerView
                android:id="@+id/recyclerView"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                app:layout_behavior="@string/appbar_scrolling_view_behavior" />

        </androidx.swiperefreshlayout.widget.SwipeRefreshLayout>


        ...

    </androidx.coordinatorlayout.widget.CoordinatorLayout>


    ...

</androidx.drawerlayout.widget.DrawerLayout>
```

```kotlin
class MainActivity : AppCompatActivity() {

    //...

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
		
        //...

        swipeRefresh.setColorSchemeResources(R.color.colorPrimary)
        swipeRefresh.setOnRefreshListener {
            refreshFruits(adapter)
        }
    }

    private fun refreshFruits(adapter: FruitAdapter) {
        thread {
            Thread.sleep(2000)
            runOnUiThread {
                initFruits()
                adapter.notifyDataSetChanged()
                swipeRefresh.isRefreshing = false
            }
        }
    }
	
    //...

}
```



## 12.7 可折叠式标题栏

### CollapsingToolbarLayout

它不能独立存在，被限定只能作为 **AppBarLayout**的直接子布局

新建一个**Activity**命名为 **FruitActivity**

具体内容太多了，直接看书吧

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.google.android.material.appbar.AppBarLayout
        android:id="@+id/appBar"
        android:layout_width="match_parent"
        android:layout_height="250dp">

        <com.google.android.material.appbar.CollapsingToolbarLayout
            android:id="@+id/collapsingToolbar"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:contentScrim="@color/colorPrimary"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

            <ImageView
                android:id="@+id/fruitImageView"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scaleType="centerCrop"
                app:layout_collapseMode="parallax" />

            <androidx.appcompat.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin" />

        </com.google.android.material.appbar.CollapsingToolbarLayout>
    </com.google.android.material.appbar.AppBarLayout>

    <androidx.core.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">

            <com.google.android.material.card.MaterialCardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginLeft="15dp"
                android:layout_marginTop="35dp"
                android:layout_marginRight="15dp"
                android:layout_marginBottom="15dp"
                app:cardCornerRadius="4dp">

                <TextView
                    android:id="@+id/fruitContentText"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_margin="10dp" />
            </com.google.android.material.card.MaterialCardView>
        </LinearLayout>
    </androidx.core.widget.NestedScrollView>

    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:src="@drawable/ic_comment"
        app:layout_anchor="@id/appBar"
        app:layout_anchorGravity="bottom|end" />

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```



```kotlin
class FruitActivity : AppCompatActivity() {

    companion object {
        const val FRUIT_NAME = "fruit_name"
        const val FRUIT_IMAGE_ID = "fruit_image_id"
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_fruit)

        val fruitName = intent.getStringExtra(FRUIT_NAME) ?: ""
        val fruitImageId = intent.getIntExtra(FRUIT_IMAGE_ID, 0)
        setSupportActionBar(toolbar)
        supportActionBar?.setDisplayHomeAsUpEnabled(true)
        collapsingToolbar.title = fruitName
        Glide.with(this).load(fruitImageId).into(fruitImageView)
        fruitContentText.text = generateFruitContent(fruitName)
    }

    override fun onOptionsItemSelected(item: MenuItem): Boolean {
        when (item.itemId) {
            android.R.id.home -> {
                finish()
                return true
            }
        }
        return super.onOptionsItemSelected(item)
    }

    private fun generateFruitContent(fruitNmae: String) = fruitNmae.repeat(500)
}
```

```kotlin
class FruitAdapter(val context: Context, val fruitList: List<Fruit>) :
    RecyclerView.Adapter<FruitAdapter.ViewHolder>() {
	
    //...

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(context).inflate(R.layout.fruit_item, parent, false)
        //change start
        val holder = ViewHolder(view)
        holder.itemView.setOnClickListener {
            val position = holder.adapterPosition
            val fruit = fruitList[position]
            val intent = Intent(context, FruitActivity::class.java).apply {
                putExtra(FruitActivity.FRUIT_NAME, fruit.name)
                putExtra(FruitActivity.FRUIT_IMAGE_ID, fruit.imageId)
            }
            context.startActivity(intent)
        }
        return holder
        //end
    }

    //...
}
```



### 充分利用系统状态栏空间

实现将背景图与状态栏合二为一

借助 **android:fitSystemWindows**属性

把CoordinatorLayout，AppBarLayout，CollapsingToolbarLayout，ImageView的这一属性全部设置为true

然后修改 **src/main/res/values/styles.xml**

```xml
<resources>
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>

    <style name="FruitActivityTheme" parent="AppTheme">
        <item name="android:statusBarColor">@android:color/transparent</item>
    </style>

</resources>
```

最后修改 **src/main/AndroidManifest.xml** 中 **.FruitActivity**的 **android:theme**属性

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.materialtest">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity
            android:name=".FruitActivity"
            android:theme="@style/FruitActivityTheme"></activity>
        <activity
            android:name=".MainActivity"
            android:label="Fruits">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```



# Kotlin拓展：编写好用的工具方法

## 求N个数的最大最小值

```kotlin
fun <T :Comparable<T>> max(vararg nums:T):T {
    if (nums.isEmpty()) throw RuntimeException("Params can not be empty")
    var maxNum = nums[0]
    for (num in nums) {
        if (num>maxNum) {
            maxNum = num
        }
    }
    return maxNum
```

## 简化Toast用法

```kotlin
fun String.showToast(context: Context, duration: Int = Toast.LENGTH_SHORT) {
    Toast.makeText(context, this, duration).show()
}

fun Int.showToast(context: Context, duration: Int = Toast.LENGTH_SHORT) {
    Toast.makeText(context, this, duration).show()
}
//用法："This is example".showToast(context, Toast.Length_LONG)
```

## 简化Snackbar用法

```kotlin
fun View.showSnackbar(
    text: String,
    actionText: String? = null,
    duration: Int = Snackbar.LENGTH_SHORT,
    block: (() -> Unit)? = null
) {
    val snackbar = Snackbar.make(this, text, duration)
    if (actionText != null && block != null) {
        snackbar.setAction(actionText) {
            block()
        }
    }
    snackbar.show()
}

fun View.showSnackbar(
    text: String,
    actionResId: Int? = null,
    duration: Int = Snackbar.LENGTH_SHORT,
    block: (() -> Unit)? = null
) {
    val snackbar = Snackbar.make(this, text, duration)
    if (actionResId != null && block != null) {
        snackbar.setAction(actionResId) {
            block()
        }
    }
    snackbar.show()
}
//用法：view.showSnackbar("This is example", "Action") {Do Something}
```









