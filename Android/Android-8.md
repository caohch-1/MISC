---
title: Android-8
categories:
- Android


---

基于《第一行代码——Android》@郭霖

# 第八章: 探究ContentProvider

## 8.1 ConetentProvider简介

实现安全的跨程序共享



## 8.2 运行时权限

### 权限机制详解

权限基本分成两类 普通权限和危险权限

目前为止的危险权限：

用户一旦同意某个权限申请之后，同组的其他权限也会被系统自动授权

![危险权限](https://s1.ax1x.com/2020/07/24/Uj3FPK.png)



### 在程序运行时申请权限

不太好叙述 直接看代码 都是一些规范的写法

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        makeCall.setOnClickListener {
            if (ContextCompat.checkSelfPermission(
                    this,
                    android.Manifest.permission.CALL_PHONE
                ) != PackageManager.PERMISSION_GRANTED
            //以上判断是否给了我们权限
            ) {
                //没有权限 调用以下方法请求权限
                ActivityCompat.requestPermissions(
                    this,
                    arrayOf(android.Manifest.permission.CALL_PHONE),
                    1
                )
            } else {
                //有权限 直接打电话
                call()
            }

        }
    }

    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        when (requestCode) {
            1 -> {
                if (grantResults.isNotEmpty() &&
                    grantResults[0] == PackageManager.PERMISSION_GRANTED
                ) {
                    call()
                } else {
                    Toast.makeText(this, "You denied the permission", Toast.LENGTH_SHORT).show()
                }
            }
        }
    }

    private fun call() {
        try {
            val intent = Intent(Intent.ACTION_CALL)
            intent.data = Uri.parse("tel:10086")
            startActivity(intent)
        } catch (e: SecurityException) {
            e.printStackTrace()
        }
    }
}
```



## 8.3 访问其他程序中的数据

### ContentResolver 的基本用法

如果想要访问 **ContentProvider**中的共享数据，就一定要借助 **ContentResolver**类，可以通过 **Context**中的 **getContentResolver**方法获取该类实例

**ContentResolver**提供了增删查改功能，以查询函数 **query**为例子，我们看看他的参数列表

![query参数](https://s1.ax1x.com/2020/07/24/UjteE9.jpg)



详细说说第一个参数 **内容Uri**

它由两部分组成 **authority**和 **path**

前者一般是 **包名** 

后者一般是 **表名**

字符串头部还要加上协议声明

后面还可以加上id 也支持通配符 *和#

i.e. **content://com.example.app.provider/tablex/idx**



得到 **内容Uri**后还要解析成 **uri对象**

```kotlin
val uri = Uri.parse("content://com.example.app.provider/tablex")
```



现在就可以借助这个小宝贝使用 **query**方法查询了

```kotlin
val cursor = contentResolver.query (
uri,
projection,
selection,
selectionArgs,
sortOrder)
```



查询完成后返回一个 **Cursor**对象，这时就可以把数据从中逐个读取了

思路很简单，和上一张差不多

```kotlin
while (cursor.moveToNext()) {
    val column1 = cursor.getString(cursor.getColumnIndex("column1"))
    val column2 = cursor.getInt(cursor.getColumnIndex("column2"))
}
cursor.close()
```



以上就是查询啦

再来看看增删改

```kotlin
val values = contentValuesOf("column1" to "text", "column2" to 1)
contentResolver.insert(uri, values)
```

```kotlin
val values = contentValuesOf("column1" to "")
contentResolver.update(uri, values, "column1 = ? and column2 = ?", arrayOf("text", "1"))
```

```kotlin
contentResolver.delete(uri, "column2 = ?", arrayOf("1"))
```



### 读取系统联系人

直接看代码

```kotlin
package com.example.contactstest

import android.Manifest
import android.content.pm.PackageManager
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.provider.ContactsContract
import android.widget.ArrayAdapter
import android.widget.Toast
import androidx.core.app.ActivityCompat
import androidx.core.content.ContextCompat
import kotlinx.android.synthetic.main.activity_main.*

class MainActivity : AppCompatActivity() {
    private val contactsList = ArrayList<String>()
    private lateinit var adapter: ArrayAdapter<String>

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        adapter = ArrayAdapter(this, android.R.layout.simple_list_item_1, contactsList)
        //contactsView已经在xml布局里写好了
        contactsView.adapter = adapter
        
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_CONTACTS) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this, arrayOf(Manifest.permission.READ_CONTACTS), 1)
        } else {
            readContacts()
        }
    }

    override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<out String>, grantResults: IntArray) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        when (requestCode) {
            1 -> {
                if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    readContacts()
                } else {
                    Toast.makeText(this, "You denied the permission", Toast.LENGTH_SHORT).show()
                }
            }
        }
    }

    private fun readContacts() {
        //查询联系人数据
        //没有按照之前的说法做Uri对象是因为我们可以直接按照如下方式获取封装好的对象
        contentResolver.query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI, null, null, null, null)?.apply {
            while (moveToNext()) {
                //获取联系人姓名
                val displayName = getString(getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME))
                //获取联系人手机号
                val number = getString(getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER))
                contactsList.add("$displayName\n$number")
            }
            adapter.notifyDataSetChanged()
            close()
        }
    }
}
```



## 8.4 创建自己的ContentProvider

### 创建步骤

可以通过新建一个类去继承 **ContentProvider**来实现

它当中有六个抽象方法 必须重写

```kotlin
class MyProvider : ContentProvider() {
	//通常会在这里完成对数据库的创建和升级等操作，返回true表示ContentProvider初始化成功，返回false则表示失败
    override fun onCreate(): Boolean {
        return false
    }
    
	//uri参数用于确定查询哪张表，projection参数用于确定查询哪些列，selection和selectionArgs参数用于约束查询哪些行，sortOrder参数用于对结果进行排序，查询的结果存放在Cursor对象中返回
    override fun query(uri: Uri, projection: Array<String>?, selection: String?, selectionArgs: Array<String>?, sortOrder: String?): Cursor? {
        return null
    }
    
	//uri参数用于确定要添加到的表，待添加的数据保存在values参数中。添加完成后，返回一个用于表示这条新记录的URI
    override fun insert(uri: Uri, values: ContentValues?): Uri? {
        return null
    }
    
	//uri参数用于确定更新哪一张表中的数据，新数据保存在values参数中，selection和selectionArgs参数约束更新哪些行，受影响的行数将作为返回值返回
    override fun update(uri: Uri, values: ContentValues?, selection: String?, selectionArgs: Array<String>?): Int {
        return 0
    }
    
	//uri参数用于确定删除哪一张表中的数据，selection和selectionArgs参数用于约束删除哪些行，被删除的行数将作为返回值返回
    override fun delete(uri: Uri, selection: String?, selectionArgs: Array<String>?): Int {
        return 0
    }
    
	//根据传入的内容URI返回相应的MIME类型
    override fun getType(uri: Uri): String? {
        return null
    }

}
```



i.e.

```kotlin
class MyProvider : ContentProvider() {
    private val table1Dir = 0
    private val table1Item = 1
    private val table2Dir = 2
    private val table2Item = 3

    private val uriMatcher = UriMatcher(UriMatcher.NO_MATCH)

    init {
        uriMatcher.addURI("com.example.app.provider", "table1", table1Dir)
        uriMatcher.addURI("com.example.app.provider", "table1/#", table1Item)
        uriMatcher.addURI("com.example.app.provider", "table2", table2Dir)
        uriMatcher.addURI("com.example.app.provider", "table2", table2Item)
    }
	
    //别的几个就不写了 类似
    override fun query(
        uri: Uri,
        projection: Array<out String>?,
        queryArgs: Bundle?,
        cancellationSignal: CancellationSignal?
    ): Cursor? {
        when (uriMatcher.match(uri)) {
            table1Dir -> {
                //查询table1中的数据
            }
            table1Item -> {
                //查询table1中的单条数据
            }
            table2Dir -> {
                //查询table2中的所有数据
            }
            table2Item -> {
                //查询table2中的单条数据
            }
        }
    }
	
    //用于获取Uri对象对应的MIME类型 一个内容URI所对应的MIME字符串由三部分组成
    //1.必须由vnd开头
    //2.以路径结尾，则后接android.cursor.dir/；以id结尾，则后接着android.cursor.item/
    //3.z最后iuy接上vnd.<authority>.<path>
    override fun getType(uri: Uri) = when (uriMatcher.match(uri)) {
        table1Dir -> "vnd.android.cursor.dir/vnd.com.example.app.provider.table1"
        table1Item -> "vnd.android.cursor.item/vnd.com.example.app.provider.table1"
        //还有两个类似
        else -> null
    }
}
```



### 实现跨程序数据共享

使用上一章的 **DatabaseTest**项目修改

先把 **MyDatabaseHelper**中使用 **Toast**的删除 跨程序访问时不能直接使用 **Toast**

接着New->Other->Content Provider  **authority**指定为 **com.example.databasetest.provider**

```kotlin
class DatabaseProvider : ContentProvider() {
    private val bookDir = 0 //访问book中所有数据
    private val bookItem = 1 //访问book中单条数据
    private val categoryDir = 2 //访问category中所有数据
    private val categoryItem = 3 //访问category中单条数据
    private val authority = "com.example.databasetest.provider"
    private var dbHelper: MyDatabaseHelper? = null

    //对UriMatcher进行初始化操作 将期望匹配的几种URI格式加入
    private val uriMatcher by lazy { //懒加载代码块中的代码只有当uriMatcher变量首次被调用时彩绘执行
        // 并会把代码块中最后一行代码的返回值赋值给uriMatcher
        val matcher = UriMatcher(UriMatcher.NO_MATCH)
        matcher.addURI(authority, "book", bookDir)
        matcher.addURI(authority, "book/#", bookItem)
        matcher.addURI(authority, "category", categoryDir)
        matcher.addURI(authority, "category/#", categoryItem)
        matcher
    }

    override fun onCreate() = context?.let { //借助?.和let判断 getContext方法返回值是否为空 如果为空就使用?:返回false
        dbHelper = MyDatabaseHelper(it, "BookStore.db", 2) //创建MyDatabaseHelper实例
        true //表示初始化成功
    } ?: false

    override fun query(
        uri: Uri, projection: Array<String>?, selection: String?,
        selectionArgs: Array<String>?, sortOrder: String?
    ) = dbHelper?.let {
        //查询数据
        val db = it.readableDatabase //获取SQLiteDatabase实例
        val cursor = when (uriMatcher.match(uri)) { //根据Uri判断参数判断用户想要查哪张表
            bookDir -> db.query("Book", projection, selection, selectionArgs, null, null, sortOrder)
            categoryDir -> db.query(
                "Category",
                projection,
                selection,
                selectionArgs,
                null,
                null,
                sortOrder
            )
            bookItem -> {
                val bookId = uri.pathSegments[1]
                db.query("Book", projection, selection, selectionArgs, null, null, sortOrder)
            }
            categoryItem -> {
                val categoryId =
                    uri.pathSegments[1] //该方法会将内容URI权限之后的部分以"/"进行分割 并放在一个字符串列表 这里取出的就是id
                db.query("Category", projection, selection, selectionArgs, null, null, sortOrder)
            }
            else -> null
        }
        cursor
    }

    override fun insert(uri: Uri, values: ContentValues?) = dbHelper?.let {
        //添加数据
        val db = it.writableDatabase //获取SQLiteDatabase实例
        val uriReturn = when (uriMatcher.match(uri)) {
            bookDir, bookItem -> {
                val newBookId = db.insert("Book", null, values)
                Uri.parse("content://$authority/book/$newBookId") //insert方法要求返回一个能够表示这条新增数据的Uri
            }
            categoryDir, categoryItem -> {
                val newCategoryId = db.insert("Category", null, values)
                Uri.parse("content://$authority/category/$newCategoryId")
            }
            else -> null
        }
        uriReturn
    }

    override fun update(
        uri: Uri, values: ContentValues?, selection: String?,
        selectionArgs: Array<String>?
    ) = dbHelper?.let {
        //更新数据
        val db = it.writableDatabase //获取SQLiteDatabase实例
        val updatedRows = when (uriMatcher.match(uri)) {
            bookDir -> db.update("Book", values, selection, selectionArgs)
            bookItem -> {
                val bookId = uri.pathSegments[1]
                db.update("Book", values, "id = ?", arrayOf(bookId))
            }
            categoryDir -> db.update("Category", values, selection, selectionArgs)
            categoryItem -> {
                val categoryId = uri.pathSegments[1]
                db.update("Category", values, "id = ?", arrayOf(categoryId))
            }
            else -> 0
        }
        updatedRows
    } ?: 0

    override fun delete(uri: Uri, selection: String?, selectionArgs: Array<String>?) =
        dbHelper?.let {
            //删除数据
            val db = it.writableDatabase //获取SQLiteDatabase实例
            val deletedRows = when (uriMatcher.match(uri)) {
                bookDir -> db.delete("Book", selection, selectionArgs)
                bookItem -> {
                    val bookId = uri.pathSegments[1]
                    db.delete("Book", "id = ?", arrayOf(bookId))
                }
                categoryDir -> db.delete("Category", selection, selectionArgs)
                categoryItem -> {
                    val categoryId = uri.pathSegments[1]
                    db.delete("Category", "id = ?", arrayOf(categoryId))
                }
                else -> 0
            }
            deletedRows
        } ?: 0

    //和上一节写法一致 不解释了
    override fun getType(uri: Uri) = when (uriMatcher.match(uri)) {
        bookDir -> "vnd.android.cursor.dir/vnd.com.example.databasetest.provider.book"
        bookItem -> "vnd.android.cursor.item/vnd.com.example.databasetest.provider.book"
        categoryDir -> "vnd.android.cursor.dir/vnd.com.example.databasetest.provider.category"
        categoryItem -> "vnd.android.cursor.item/vnd.com.example.databasetest.provider.category"
        else -> null
    }
}
```



# Kotlin拓展

## 泛型的基本用法

定义一个泛型类

```kotlin
class Myclass<T> {
    fun method(param: T):T {
        return param
    }
}
```



定义一个泛型方法

```kotlin
class Myclass {
    fun <T> method(param: T):T {
        return param
    }
}
```



## 类委托和属性委托

类委托：

```kotlin
class MySet<T>(val helperSet: HashSet<T>) : Set<T> by helperSet { //除了以下新加、修改的功能 别的与HashSet一致
    fun helloworld()=println("Hello World")
    override fun isEmpty() = false
}
```



属性委托：

```kotlin
class MyClass {
    var p by Delegate()
}
```

**by**关键字连接左边的p属性和右边的 **Delegate**实例 

调用p属性的时候自动调用**Delegate**的 **getValue**

给p属性赋值的时候自动调用**Delegate**的 **setValue**

容易想到，我们还应该修改**Delegate**类

```kotlin
class Delegate {
    var propValue: Any? = null
    
    operator getValue(myClass:MyClass, prop: KProperty<*>): Any? {
        return propValue
    }
    
    operator setValue(myClass:MyClass, prop:KProperty<*>, value: Any?) {
        propValue= value
    }
}
```

以上是一种标准的代码实现模板



## 实现自己的lazy函数

```kotlin
//by lazy基本语法结构
val p by lazy{
    ...
}
```



正式开始

```kotlin
fun <T> later(block:() -> T) = Later(block)

class Later<T>(val block: () -> T){
    var value:Any? = null
    operator fun getValue(any:Any?,prop:KProperty<*>):T {
        value = block()
    }
	return value as T
}
```







