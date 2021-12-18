---
title: Android-5
categories:
- Android
---

基于《第一行代码——Android》@郭霖

# 第五章：探究Fragment

## 5.1 What's this

一种可以嵌入在Activity中的UI片段

在平板上广泛使用



## 5.2使用方法

### 简单用法

#### Step1: 创建布局

创建 **src/main/res/layout/left_fragment.xml** （同样的还有个right 不赘述）

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:text="Button" />

</LinearLayout>

```



#### Step2: 创建类

创建 **src/main/java/com/example/fragmenttest/LeftFragment.kt** （同样的还有个right 不赘述）

加载布局 不多解释

```kotlin
class LeftFragment : Fragment() {
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.left_fragment, container, false)
    }
}
```



#### Step3: 修改Main布局

 修改 **src/main/res/layout/activity_main.xml**

使用<fragment>标签在布局中添加**Fragment**，通过**name**属性显式声明要添加的**Fragment**类名

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">

    <fragment
        android:id="@+id/leftFrag"
        android:name="com.example.fragmenttest.LeftFragment"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1" />

    <fragment
        android:id="@+id/rightFrag"
        android:name="com.example.fragmenttest.RightFragment"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1" />

</LinearLayout>
```



### 动态添加

#### Step1: 创建待添加Fragment的实例

创建 **src/main/java/com/example/fragmenttest/AnotherRightFragment.kt**

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#ffff00"
    android:orientation="vertical">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:text="This another right fragment"
        android:textSize="24sp" />

</LinearLayout>

```

创建 **src/main/java/com/example/fragmenttest/AnotherRightFragment.kt**

```kotlin
package com.example.fragmenttest

import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.fragment.app.Fragment

class AnotherRightFragment : Fragment() {
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.another_right_fragment, container, false)
    }
}
```



#### Step2: 改变Main布局

修改 **src/main/java/com/example/fragmenttest/AnotherRightFragment.kt**

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">

    <fragment
        android:id="@+id/leftFrag"
        android:name="com.example.fragmenttest.LeftFragment"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1" />

    <FrameLayout
        android:id="@+id/rightLayout"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1" />

</LinearLayout>
```



#### Step3: 实现点击按钮改变布局

修改 **src/main/java/com/example/fragmenttest/MainActivity.kt**

```kotlin
package com.example.fragmenttest

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import androidx.fragment.app.Fragment
import kotlinx.android.synthetic.main.left_fragment.*

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
		
        button.setOnClickListener {
            replaceFragment(AnotherRightFragment())
        }
        replaceFragment(RightFragment())
    }

    private fun replaceFragment(fragment: Fragment) {
        //获取 **FragmentManager** ：调用**getSupportFragmentMangaer**方法获取
        val fragmentManager = supportFragmentManager
        //开启一个事务：通过调用**beginTransaction**方法开启
        val transaction = fragmentManager.beginTransaction()
        //向容器内添加或替换**Fragment**，一般使用**replace**方法实现 参数为容器id 和Fragment实例
        transaction.replace(R.id.rightLayout, fragment)
        //提交事务：通过调用**commit**方法
        transaction.commit()
    }
}
```



### 实现返回栈

**FragmentTransaction** 中提供了一个 **addToBackStack**方法 可以帮助实现

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        button.setOnClickListener {
            replaceFragment(AnotherRightFragment())
        }
        replaceFragment(RightFragment())
    }

    private fun replaceFragment(fragment: Fragment) {
        val fragmentManager = supportFragmentManager
        val transaction = fragmentManager.beginTransaction()
        transaction.replace(R.id.rightLayout, fragment)
        //added line
        transaction.addToBackStack(null)
        transaction.commit()
    }

```



### Fragment和Activity之间的交互

#### 在Activity中调用Fragment

```kotlin
val fragment = supportFragmentManager.findFragmentById(R.id.leftFrag) as LeftFragment
//kotlin-android-extensions插件提供给懒狗的
val fragment = leftFrag as LeftFragment
```



#### 在Fragment中调用Activity

```kotlin
if (activity != null) {
    val mainActivity = activity as MainActivity
}
```



#### 在Fragment中调用其他Fragment

好哥哥 动动脑子 看看上面两条 想到了么



## 5.3 生命周期

### 状态和回调

状态与Activity类似 见下图 

![Fragment生命周期](https://developer.android.com/images/fragment_lifecycle.png)

看看几个回调方法

**onAttach( )**: 当Fragment和Activity建立联系时调用

**onCreateView( )**: 为Fragment创建视图（加载布局）时调用

**onActivityCreated( )**: 确认与Fragment相关联的Activity已经创建完毕时调用

**onDestoryView( )**: 当与Fragment关联的视图被移除时调用

**onDetach( )**: 当Fragment和Activity解除关联时调用



## 5.4: 动态加载的技巧

### 限定符

平板左右显示两页内容 而手机是一页 限定符来判断并进而动态加载

修改 src/main/res/**layout**/activity_main.xml 

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">

    <fragment
        android:id="@+id/leftFrag"
        android:name="com.example.fragmenttest.LeftFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_weight="1" />

</LinearLayout>
```



创建文件夹 src/main/res/**layout-large**

并在该目录下创建 **activity_main.xml** 是的和 **layout** 目录下的那个xml名字一样

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">

    <fragment
        android:id="@+id/leftFrag"
        android:name="com.example.fragmenttest.LeftFragment"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1" />

    <fragment
        android:id="@+id/rightFrag"
        android:name="com.example.fragmenttest.RightFragment"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="3" />

</LinearLayout>
```



最后去掉 **MainActivity.kt** 里的 **replaceFragment** 相关代码 即可

可能有点迷惑 但是看完下面的表结合以上操作 应该就能理解了

![限定符](https://s1.ax1x.com/2020/07/20/U4OZE6.png)



### 最小宽度限定符

新问题出现了 large到底是指多大 

如果要灵活为不同机型适配 使用最小宽度限定符

创建目录 **src/main/res/layout-sw600dp**

并在该目录下创建 **activity_main.xml** 

梅开二度 不多作解释



## 5.5 实践 简易的新闻应用

看书和码吧 懒得粘过来了



# Kotlin拓展

## 扩展函数

扩展函数指不修改某个类源码情况下，仍然可以打开这个类，向该类添加新的函数

基本语法结构

```kotlin
fun ClassName.methodName(param1:Type, param2:Type): Type {
    return Type
}
```



i.e. 给String库 增加数字母功能

```kotlin
fun String.lettersCount() : Int{
    var count =0;
    for (char in this) {
        if (char.isLetter()) {
            count++
        }
    }
    return count
}
```



## 运算符重载

基本语法

```kotlin
class Obj {
    operator fun plus(obj: Obj):Obj {
        //Something
    }
}
```

i.e.

```kotlin
class Money(val value: Int) {
    operator fun plus(money: Money): Money {
        val sum = value + money.value
        return Money(sum)
    }

    operator fun plus(newValue: Int): Money {
        val sum = value + newValue
        return Money(sum)
    }
}
```



![语法糖和实际调用函数](https://s1.ax1x.com/2020/07/20/U5ZwD0.png)


