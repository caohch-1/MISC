---
title: Android-7
categories:
- Android

---

基于《第一行代码——Android》@郭霖

# 第七章: 持久化技术

## 7.1 简介

Android系统主要提供了三种实现数据持久化的功能： **文件储存** **SharedPreferences储存** **数据库储存**

下面我们一个个来看



## 7.2 文件储存

### 将数据储存到文件中

**Context** 类中提供了一个 **openFileOutput** 方法，可以用于将数据储存到指定文件中

它有两个参数 第一个是 **文件名** 第二个是 **操作模式**

操作模式主要有 MODE_PRIVATE 和 MODE_APPEND 前者覆盖 后者追加



```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }

    override fun onDestroy() {
        super.onDestroy()
        //editText是一个EditText的id 我们已经在xml布局文件中创建了 这里不再演示
        val inputText = editText.text.toString()
        save(inputText)
    }

    private fun save(inputText: String) {
        try {
            // openFileOutput返回一个FileOutputStream对象
            val output = openFileOutput("data", Context.MODE_PRIVATE)
            // 利用上面的对象构建一个BufferedWriter
            val writer = BufferedWriter(OutputStreamWriter(output))
            //use是一个内置扩展函数 保证lambda表达式中的代码执行完后自动将外层的流关闭
            writer.use {
                it.write(inputText)
            }
        } catch (e: IOException) {
            e.printStackTrace()
        }
    }
}
```



介绍一个从未用过的工具 AS右下角的 **Device File Explorer**

可以在当中的**data/data/com.example.xxxxx/files/**中找到我们的data文件



### 从文件中读取数据

有Output就有Input 现在来看看**openFileInput**

它只有一个参数 就是要读取的文件名

直接看代码吧 没什么好说的

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
		//add lines
        val inputText: String = load()
        if (inputText.isNotEmpty()) {
            editText.setText(inputText)
            editText.setSelection(inputText.length)
            Toast.makeText(this, "Restoring succeed", Toast.LENGTH_SHORT).show()
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        val inputText = editText.text.toString()
        save(inputText)
    }
	//add function 和save非常类似
    private fun load(): String {
        val content = StringBuilder()
        try {
            val input = openFileInput("data")
            val reader = BufferedReader(InputStreamReader(input))
            reader.use {
                reader.forEachLine {
                    content.append(it)
                }
            }
        } catch (e: IOException) {
            e.printStackTrace()
        }
        return content.toString()
    }
	
    private fun save(inputText: String) {
        try {
            val output = openFileOutput("data", Context.MODE_PRIVATE)
            val writer = BufferedWriter(OutputStreamWriter(output))
            writer.use {
                it.write(inputText)
            }
        } catch (e: IOException) {
            e.printStackTrace()
        }
    }
}
```



## 7.3 SharedPreferences 储存

~~众所周知~~文件储存不适合保存较为复杂的结构型数据

现在看一种更简单易用的方式 他使用**键值对**方式储存



### 储存

要想用他储存数据，首先要获取SharedPreferences对象 两种方法

- Context类中的 getSharedPreferences( ) 使用方式和第一个介绍的函数类似
- Activity类中的 getPreferences( ) 他只接受一个参数是操作模式 他会把当前Activity的类名作为文件名

得到对象之后，开始储存数据，分为三步

1. 调用小宝贝的**edit**方法获取一个**SharedPreferences.Editor**对象
2. 添加数据 具体通过各种对应方法 如**putBoolean**, **putString**...
3. 调用**apply**提交，完成

i.e.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
		//saveButton已经在xml布局里写好
        saveButton.setOnClickListener {
            //是不是完美符合上述步骤 美汁er汁er
            val editor = getSharedPreferences("data", Context.MODE_PRIVATE).edit()
            editor.putString("name", "CHC")
            editor.putInt("age", 20)
            editor.putBoolean("married", false)
            editor.apply()
        }
    }
}
```



### 读取数据

SharedPreferences对象中提供了一系列get方法用于读取数据

两个参数 第一个是key 第二个是找不到时候的缺省值



i.e.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        saveButton.setOnClickListener {
            val editor = getSharedPreferences("data", Context.MODE_PRIVATE).edit()
            editor.putString("name", "CHC")
            editor.putInt("age", 20)
            editor.putBoolean("married", false)
            editor.apply()
        }
        
		//added function
        restoreButton.setOnClickListener {
            val prefs = getSharedPreferences("data", Context.MODE_PRIVATE)
            
            val name = prefs.getString("name", "Ghost")
            val age = prefs.getInt("age", -1)
            val married = prefs.getBoolean("married", false)
            
            Log.d("MainActivity", "name is $name")
            Log.d("MainActivity", "age is $age")
            Log.d("MainActivity", "married is $married")
        }
    }
}
```



### 实现记住密码功能

我们之前写过一个登陆界面 在上一章的最佳实践中 在他的基础上修改

改改界面布局先

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"

    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:orientation="horizontal">

        <TextView
            android:layout_width="90dp"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical"
            android:text="Account:"
            android:textSize="18sp" />

        <EditText
            android:id="@+id/accountEdit"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical"
            android:layout_weight="1" />
    </LinearLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="60sp"
        android:orientation="horizontal">

        <TextView
            android:layout_width="90dp"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical"
            android:text="Password: "
            android:textSize="18sp" />

        <EditText
            android:id="@+id/passwordEdit"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical"
            android:layout_weight="1"
            android:inputType="textPassword" />

    </LinearLayout>

    <!--new codes-->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <!--这是一个复选框 第一次见吧-->
        <CheckBox
            android:id="@+id/rememberPass"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Remember password"
            android:textSize="18sp" />

    </LinearLayout>
    <!--end-->
    
    <Button
        android:id="@+id/login"
        android:layout_width="200dp"
        android:layout_height="60dp"
        android:layout_gravity="center_horizontal"
        android:text="Login" />

</LinearLayout>
```



然后修改**src/main/java/com/example/broadcastbestpractice/ui/login/LoginActivity.kt**

```kotlin
class LoginActivity : BaseActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_login)
        
        //new lines
        val prefs = getPreferences(Context.MODE_PRIVATE) //获取对象
        val isRemember = prefs.getBoolean("remember_password", false) //查看是否有记录过密码
        if (isRemember) {
            val account = prefs.getString("account", "") //获取储存的账号
            val password = prefs.getString("password", "") //获取储存的密码
            accountEdit.setText(account) //把账号放到文本框中
            passwordEdit.setText(password) //把密码放到文本框中
            rememberPass.isChecked = true //设置为已经记录密码
        }

        login.setOnClickListener {
            val account = accountEdit.text.toString() //获取文本框中的账号
            val password = passwordEdit.text.toString() //获取文本框中的密码

            if (account == "admin" && password == "123456") {
                val editor = prefs.edit()  //获取一个SharedPreferences.Editor对象
                if (rememberPass.isChecked) { //检查复选框有没有选中 也就是说看看用户要不要记住
                    editor.putBoolean("remember_password", true) //储存是否已经记住账号密码
                    editor.putString("account", account) //储存账号
                    editor.putString("password", password) //储存密码
                } else {
                    editor.clear() //清除掉记录
                }
                editor.apply() //提交
                //end

                val intent = Intent(this, MainActivity::class.java)
                startActivity(intent)
                finish()
            } else {
                Toast.makeText(this, "account or password is wrong", Toast.LENGTH_SHORT).show()
            }
        }
    }
}
```



## 7.4 SQLite数据库储存

### 创建数据库

Android提供了一个**SQLiteOpenHelper**类 是抽象类哦

有两个抽象方法**onCreate**和**onUpgrade** 必须要重写哦

还有两个非常重要的实例方法 **getReadableDatabase**和**getWritableDatabase**

两个构造函数 一般用参数比较少的那个 依然有四个参数

1. Context 没啥好说的
2. 数据库名
3. 自定义的Cursor 一般传入null
4. 当前数据库版本号 用于对数据库进行升级操作

i.e.

新建**src/main/java/com/example/databasetest/MyDatabaseHelper.kt**

```kotlin
class MyDatabaseHelper(val context: Context, name: String, version: Int) :
    SQLiteOpenHelper(context, name, null, version) {
    //text 表示文本类型 real 表示浮点数类型
    //以下把建表语句定义成字符串
    private val createBook = "create table Book (" +
            " id integer primary key autoincrement," +
            "author text," +
            "price real," +
            "pages integer," +
            "name text)"

    override fun onCreate(p0: SQLiteDatabase) {
        //在此调用下述方法建库建表
        p0.execSQL(createBook)
        Toast.makeText(context, "Create succeed", Toast.LENGTH_SHORT).show()
    }

    override fun onUpgrade(p0: SQLiteDatabase?, p1: Int, p2: Int) {
    }

}
```



修改**src/main/java/com/example/databasetest/MainActivity.kt**

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val dbHelper = MyDatabaseHelper(this, "BookStore.db", 1)
        //createDatabase是已经在xml布局里写好了的
        createDatabase.setOnClickListener {
            dbHelper.writableDatabase
        }
    }
}
```



### 升级数据库

直接看代码

修改**src/main/java/com/example/databasetest/MyDatabaseHelper.kt**

```kotlin
class MyDatabaseHelper(val context: Context, name: String, version: Int) :
    SQLiteOpenHelper(context, name, null, version) {
    //text 表示文本类型 real 表示浮点数类型
    //以下把建表语句定义成字符串
    private val createBook = "create table Book (" +
            " id integer primary key autoincrement," +
            "author text," +
            "price real," +
            "pages integer," +
            "name text)"

    private val createCategory = "create table Category (" +
            "id integer primary key autoincrement," +
            "category_name text," +
            "category_code integer)"

    override fun onCreate(p0: SQLiteDatabase) {
        //在此调用下述方法建库建表
        p0.execSQL(createBook)
        p0.execSQL(createCategory)
        Toast.makeText(context, "Create succeed", Toast.LENGTH_SHORT).show()
    }

    override fun onUpgrade(p0: SQLiteDatabase, p1: Int, p2: Int) {
        //在升级时 先把表都删了 再重新调用onCreate
        p0.execSQL("drop table if exists Book")
        p0.execSQL("drop table if exists Category")
        onCreate(p0)
    }

}
```



再修改**src/main/java/com/example/databasetest/MainActivity.kt**

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        //最后一个参数 版本号一旦比之前大 就会调用onUpgrade
        val dbHelper = MyDatabaseHelper(this, "BookStore.db", 2)
        createDatabase.setOnClickListener {
            dbHelper.writableDatabase
        }
    }
}
```



### 添加数据

对表中数据的操作无非四种 **CRUD**

create retrieve update delete 增删改查

**getReadableDatabase**和**getWritableDatabase**用于创建数据库 同时也会返回一个**SQLiteDatabase**小宝贝 借助她就可以操作啦

先看看添加 使用**insert**方法

三个参数

1. 表名
2. 在未指定添加数据情况下为某些列自动赋值NULL 一般不用 直接传入null
3. 一个**ContentValues**对象 它提供一系列**put**方法重载 用于向**ContentValues**添加数据 只需要将表中的每个列名以及相应的待添加数据传入即可

i.e.

```kotlin
package com.example.databasetest

import android.content.ContentValues
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import kotlinx.android.synthetic.main.activity_main.*

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        //最后一个参数 版本号一旦比之前大 就会调用onUpgrade
        val dbHelper = MyDatabaseHelper(this, "BookStore.db", 2)
        createDatabase.setOnClickListener {
            dbHelper.writableDatabase
        }
        
		//add lines
        //显然 addData已经在xml布局里写好了
        addData.setOnClickListener {
            val db = dbHelper.writableDatabase //借助这个小宝贝对数据库操作

            val values1 = ContentValues().apply {
                put("name", "YCH NB")
                put("author", "CHC")
                put("pages", 114514)
                put("price", 20.20)
            }
            db.insert("Book", null, values1)

            val values2 = ContentValues().apply {
                put("name", "LAT NB")
                put("author", "GZA")
                put("pages", 114)
                put("price", 20.19)
            }
            db.insert("Book", null, values2)
        }
    }
}
```



### 更新数据

直接看代码 new lines部分

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        //最后一个参数 版本号一旦比之前大 就会调用onUpgrade
        val dbHelper = MyDatabaseHelper(this, "BookStore.db", 2)
        createDatabase.setOnClickListener {
            dbHelper.writableDatabase
        }

        //显然 addData已经在xml布局里写好了
        addData.setOnClickListener {
            val db = dbHelper.writableDatabase //借助这个小宝贝对数据库操作

            val values1 = ContentValues().apply {
                put("name", "YCH NB")
                put("author", "CHC")
                put("pages", 114514)
                put("price", 20.20)
            }
            db.insert("Book", null, values1)

            val values2 = ContentValues().apply {
                put("name", "LAT NB")
                put("author", "GZA")
                put("pages", 114)
                put("price", 20.19)
            }
            db.insert("Book", null, values2)
        }
        
		//new lines
        //显然 updateData已经在xml布局里写好了
        updateData.setOnClickListener {
            val db = dbHelper.writableDatabase //借助这个小宝贝对数据库操作

            val values = ContentValues()
            values.put("price", 10.99)
            db.update(
                "Book",
                values,
                "name = ?",
                arrayOf("YCH NB")
            ) //后面两个参数指定更新哪几行 第三个参数中的?是占位符 由第四个参数数组指定
        }
    }
}
```



### 删除数据

直接看代码 new lines部分

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        //最后一个参数 版本号一旦比之前大 就会调用onUpgrade
        val dbHelper = MyDatabaseHelper(this, "BookStore.db", 2)
        createDatabase.setOnClickListener {
            dbHelper.writableDatabase
        }

        //显然 addData已经在xml布局里写好了
        addData.setOnClickListener {
            val db = dbHelper.writableDatabase //借助这个小宝贝对数据库操作

            val values1 = ContentValues().apply {
                put("name", "YCH NB")
                put("author", "CHC")
                put("pages", 114514)
                put("price", 20.20)
            }
            db.insert("Book", null, values1)

            val values2 = ContentValues().apply {
                put("name", "LAT NB")
                put("author", "GZA")
                put("pages", 114)
                put("price", 20.19)
            }
            db.insert("Book", null, values2)
        }

        //显然 updateData已经在xml布局里写好了
        updateData.setOnClickListener {
            val db = dbHelper.writableDatabase //借助这个小宝贝对数据库操作

            val values = ContentValues()
            values.put("price", 10.99)
            db.update(
                "Book",
                values,
                "name = ?",
                arrayOf("YCH NB")
            ) //后面两个参数指定更新哪几行 第三个参数中的?是占位符 由第四个参数数组指定
        }
		
        //new lines
        //显然 deleteData已经在xml布局里写好了
        deleteData.setOnClickListener {
            val db = dbHelper.writableDatabase

            db.delete("Book", "pages > ?", arrayOf("1000"))
        }
    }
}
```



### 查询数据

使用**query**方法

参数很多 参考下图

![query](https://s1.ax1x.com/2020/07/23/UbX22q.png)

不要怕 多数情况下只要传入少数几个参数 

调用该方法后会返回一个**Cursor**对象 查询到的数据都得从他中取出



i.e.

直接看代码 new lines部分

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        //最后一个参数 版本号一旦比之前大 就会调用onUpgrade
        val dbHelper = MyDatabaseHelper(this, "BookStore.db", 2)
        createDatabase.setOnClickListener {
            dbHelper.writableDatabase
        }

        //显然 addData已经在xml布局里写好了
        addData.setOnClickListener {
            val db = dbHelper.writableDatabase //借助这个小宝贝对数据库操作

            val values1 = ContentValues().apply {
                put("name", "YCH NB")
                put("author", "CHC")
                put("pages", 114514)
                put("price", 20.20)
            }
            db.insert("Book", null, values1)

            val values2 = ContentValues().apply {
                put("name", "LAT NB")
                put("author", "GZA")
                put("pages", 114)
                put("price", 20.19)
            }
            db.insert("Book", null, values2)
        }

        //显然 updateData已经在xml布局里写好了
        updateData.setOnClickListener {
            val db = dbHelper.writableDatabase //借助这个小宝贝对数据库操作

            val values = ContentValues()
            values.put("price", 10.99)
            db.update(
                "Book",
                values,
                "name = ?",
                arrayOf("YCH NB")
            ) //后面两个参数指定更新哪几行 第三个参数中的?是占位符 由第四个参数数组指定
        }

        //显然 deleteData已经在xml布局里写好了
        deleteData.setOnClickListener {
            val db = dbHelper.writableDatabase

            db.delete("Book", "pages > ?", arrayOf("1000"))
        }
		
        //new lines
        //显然 queryData已经在xml布局里写好了
        queryData.setOnClickListener {
            val db = dbHelper.writableDatabase

            val cursor = db.query("Book", null, null, null, null, null, null)
            //Cursor移到第一行
            if (cursor.moveToFirst()) {
                do {
                    //这四行不解释了 字面意思 爷要上床了
                    val name = cursor.getString(cursor.getColumnIndex("name"))
                    val author = cursor.getString(cursor.getColumnIndex("author"))
                    val pages = cursor.getInt(cursor.getColumnIndex("pages"))
                    val price = cursor.getInt(cursor.getColumnIndex("price"))

                    Log.d("MainActivity", "book name is $name")
                    Log.d("MainActivity", "book author is $author")
                    Log.d("MainActivity", "book pages is $pages")
                    Log.d("MainActivity", "book price is $price")
                } while (cursor.moveToNext()) //循环变粒每一行
            }
            cursor.close() //别忘了关闭
        }
    }
}
```



### 使用SQL操作数据库

添加

```kotlin
db.execSQL("insert into Book (name, author, pages, price) values(?, ?, ?, ?)",
          arrayOf("The Da Vinci Code", "Dan Brown", "454", "16.96"))
```



更新、删除类似 不写了



查询使用别的函数

```kotlin
val cursor = db.rawQuery("select * from Book", null)
```



使用这种方式还是之前的全看个人喜好



## 7.5 实践

### 使用事务

我们都知道SQLite是支持事务的 ~~泻药，不知道~~

事务的特性可以让一系列操作要么全部完成，要么一个都不完成



```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        //最后一个参数 版本号一旦比之前大 就会调用onUpgrade
        val dbHelper = MyDatabaseHelper(this, "BookStore.db", 2)
        createDatabase.setOnClickListener {
            dbHelper.writableDatabase
        }
        
        //Something
        
        //newlines
        //显然 replaceData已经在xml布局里写好了
        //以下是事务的标准用法
        replaceData.setOnClickListener {
            val db = dbHelper.writableDatabase

            db.beginTransaction() // 开启事务
            try {
                db.delete("Book", null, null)
//                去掉这里的注释 删除和添加都不会成功执行
//                if (true) {
//                    throw NullPointerException()
//                }
                val values = ContentValues().apply {
                    put("name", "Game of Thrones")
                    put("author", "George Martin")
                    put("pages", 720)
                    put("price", 20.05)
                }
                db.insert("Book", null, values)
                db.setTransactionSuccessful() //事务已经执行成功
            } catch (e: Exception) {
                e.printStackTrace()
            } finally {
                db.endTransaction() //结束事务
            }
        }
    }
}
```



### 升级数据库的最佳写法

之前的写法很傻 删库重建 我们学习更好的写法现在

第一个需求 要添加一张新的表Category

```kotlin
class MyDatabaseHelper(val context: Context, name: String, version: Int) :
    SQLiteOpenHelper(context, name, null, version) {
    //text 表示文本类型 real 表示浮点数类型
    //以下把建表语句定义成字符串
    private val createBook = "create table Book (" +
            " id integer primary key autoincrement," +
            "author text," +
            "price real," +
            "pages integer," +
            "name text)"

    private val createCategory = "create table Category (" +
            "id integer primary key autoincrement," +
            "category_name text," +
            "category_code integer)"

    override fun onCreate(p0: SQLiteDatabase) {
        //在此调用下述方法建库建表
        p0.execSQL(createBook)
        p0.execSQL(createCategory)
        Toast.makeText(context, "Create succeed", Toast.LENGTH_SHORT).show()
    }
	
    //change here
    override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
        if (oldVersion <= 1) {
            db.execSQL(createCategory)
        }
    }

}
```



又来了个需求 要在Book和Category之间建立联系 要在Book中添加category_id

```kotlin
class MyDatabaseHelper(val context: Context, name: String, version: Int) :
    SQLiteOpenHelper(context, name, null, version) {
    private val createBook = "create table Book (" +
            " id integer primary key autoincrement," +
            "author text," +
            "price real," +
            "pages integer," +
            "name text," +
            "category_id integer)" // change here

    private val createCategory = "create table Category (" +
            "id integer primary key autoincrement," +
            "category_name text," +
            "category_code integer)"

    override fun onCreate(p0: SQLiteDatabase) {
        p0.execSQL(createBook)
        p0.execSQL(createCategory)
        Toast.makeText(context, "Create succeed", Toast.LENGTH_SHORT).show()
    }

    override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
        if (oldVersion <= 1) {
            db.execSQL(createCategory)
        }
        //add lines
        if (oldVersion <= 2) {
            db.execSQL("alter table Book add column category_id integer")
        }
    }

}
```



# Kotlin拓展

## 简化SharedPreferences的用法

整个花活

```kotlin
//通过扩展函数的方式向小宝贝中增加open方法 并且接受一个函数参数 他就是一个高阶函数啦

fun SharedPreferences.open(block: SharedPreferences.Editor.() -> Unit) {
    //由于open函数有小宝贝的上下文 可以直接调用edit方法获取小宝贝.Editor
    val editor = edit()
    editor.block() //用block对函数类型参数进行调用
    editor.apply() //提交
}
```

使用花活

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
		//saveButton已经在xml布局里写好
        saveButton.setOnClickListener {
            //可以和上面7.3-储存
            getSharedPreferences("data", Context.MODE_PRIVATE).open { //这个功能已在Google提供的KTX扩展中实现了 这里open换成edit就行
                putString("name", "Tom")
                putInt("age", 28)
                putBoolean("married", false)
            }
        }
    }
}
```



## 简化ContentValues

整个花活

```kotlin
fun cvOf(vararg pairs: Pair<String, Any?>) = ContentValues().apply { //vararg可变参数列表
    for (pair in pairs) {
        val key = pair.first
        val value = pair.second
        when (value) {
            is Int -> put(key, value)
            is Long -> put(key, value)
            is String -> put(key, value)
            // More types...
            null -> putNull(key)
        }
    }

}
```

使用花活

```kotlin
val values = cvOf("name" to "Game of Thrones", "author" to "George Martin",
                 "pages" to 720, "price" to 20.85)  /这个功能已在Google提供的KTX扩展中实现了 这里cvOf换成contentValuesOf就行
db.insert("Book",null,values)
```





















































