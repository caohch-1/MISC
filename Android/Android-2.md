---
title: Android-2
categories:
- Android
---

基于《第一行代码——Android》@郭霖

# 第二章：Kotlin入门

## 2.3 基础

### 变量

val  **不可变**变量

var  **可变**变量

```kotlin
val a = 10 // 自动推导类型为int
val a: Int = 10 // 显示声明
```

```kotlin
val list = listOf("a", "b", ...)  // 不可插入，删除等
val list = mutableListOf("a", "b", ...) // 可以...
//类似的 setOf() mutableSetOf() mapOf()
val map = mapOf("a" to 1, "b" to 2, ...) // to不是关键字 是infix函数 第九章学习
```



### fun 函数声明

与c++不同在于 先value后type

```kotlin
fun function_name(param1: type, param2: type, ...): return_type {
	//Do Something
}
```
一个语法糖
```kotlin
//when there is only one line in the function, we can declare as follw
fun function_name(param1: type, param2: type, ...): return_type = /*** one line code***/
fun function_name(param1: type, param2: type, ...) = /*** one line code***/
```



### PS

AS的自动补全会自动导包 尽量不要纯手敲 ~~显然我是懒狗~~





## 2.4 比基础高一点点的基础

### **if语句**

可以有**返回值**

```kotlin
val value = if (condition) {
	return_value1
} else {
	return_value2
}
```



### **when语句**

(我愿称之为switch pro)

同样具有返回值

```kotlin
val value = when (param) {
    condition1 -> //Do Something
    condition2 -> //Do Something
    ...
    else -> //Do Something
}
```

可以不带参数（很少使用）

i.e.

```kotlin
fun getSocre(name: String) = when {
    name.startsWith("Kaid") -> 86
    name == "Kali" -> 77
    name == "Bandit" -> 87
    else -> 0
}
```



### **创建一个区间**

```kotlin
val range = 0..10 // 0,1,2,3,4,5,6,7,8,9,10
val range = 0 until 10 // 0,1,2,3,4,5,6,7,8,9
val range = 0 until 10 step 2 // 0,2,4,6,8
val range = 10 downTo 0 // 10,9,8,7,6,5,4,3,2,1,0
```



### **for语句**

基本没啥

类似于Python

```kotlin
for (i in 一个区间) {
    //Do Something
}
```



### **while语句**

~~真的~~没啥





## 2.5 面向小宝贝

### **继承**

Kotlin中类默认**不能被继承**

同时 一个子类只能有**一个**父类

主构造函数没有函数体 但可以通过 **init** 干活

```kotlin
//加上 open 就可以啦啦啦啦
open class Father(val name: String, val age: Int) {
    //主构造函数
    init {
        //Something
    }
    //Something
}
```

值得注意的是子类中的name和age**不可以**声明为val或者var

同时 Father后面**必须**跟着括号 **即使没有参数** 这表示调用主构造函数 (除非没有主构造函数 在后文)

```kotlin
open class Father(val name: String, val age: Int) {
    //Something
}

class Son(val city: String, name: String, age: Int) : Father(name, age) {
    //Something
}
```

次构造函数(几乎用不到)

```kotlin
class Son(val city: String, name: String, age: Int) : Father(name, age) {
    //次构造函数
    constructor(name: String, age: Int) : this("", name, age) {
        //Something
    }
}
```

```kotlin
val son1 = Son("Shanghai", "Kaid", 20)
// 次构造函数
val son2 = Son("Kali", 21)
```

类中只有次构造函数 此时Father后面没有括号

```kotlin
class Son : Father {
    constructor(name: String, age: Int) : super(name, age) {
        //Something
    }
}
```



### 接口

似乎类似于c++抽象类???

虽然是Kotlin是**单**继承结构 但是可以**多**接口

接口(下面的Base)后面没有括号 因为没有主构造函数

```kotlin
interface Base {
    fun something1()
    //Something
}

class Son(val city: String, name: String, age: Int) : Father(name, age), Base {
    override fun something1() {
        //Something
    }
}
```



### 可见性修饰符

private

public 默认

protected 

internal 同一模块中的类可见（关于模块开发 会在后面学习）



### Kotlin类特色   ~~魔法~~

数据类

一行代码完成

自动重写equals() hashCode() toString()

```kotlin
data class DataClass(param1: type, param2: type, ...)
```



单例类

全局中只能有一个实例 ~~哈哈 完全没听说过呢 不愧是我~~

一行代码完成

自动重写一些东西

```kotlin
object Singleton {
    //Something
}
```





## 2.6 Lambda

### 一些规矩

```kotlin
{ param1: type, param2: type, ... -> //Something }
```

当lambda参数是**函数最后一个参数**时 可以写在函数括号外面

当lambda参数是**函数唯一一个参数**时 函数的括号可以忽略

**type**有时候可以不写 毕竟Kotlin的推导功能很出色  ~~对此 我还是觉得写会比较好~~

当lambda**本身只有一个参数** 可以用 **it** 来代替参数名



### 几个常用API

- **filter()**  顾名思义
- **any()** and **all()**  返回boolen 判断是否 有至少一个/全部 满足条件
- **len()**   i.e. obj.let {obj2 -> //Something}   这里obj2就是obj  let()把调用他的对象作为参数传入lambda表达式中 可以用于空指针检测 





## 2.7 空指针检测

安卓系统崩溃率最高的异常就是 **NullPointerException**

Kotlin默认所有 参数和变量 **不可为空** 不然不过编译

在 **type** 后面加 **?** 就允许是空啦       ~~????~~

**?.**  操作符 不空干活空了下班

**?:**  操作符 左右接受表达式 左空返左空了返右 

i.e.

```kotlin
if (a != null) {
    a.doSomenthing()
}
//用?.的魔法
a?.doSomething()
```

```kotlin
val c = if (a != null) {
    a
} else {
    b
}
//Kotlin使出了?: 代码量受到了大量伤害
val c = a ?: b
```

```kotlin
//结合?. and ?: 的例子( 展现懒狗的力量 )
fun getTextLength(text: String) = text.?length ?: 0
```



**!!** 非空断言工具 写在对象后面 表示你**十分确定**这个对象不为空 (因为Kotlin的空指针检测机制并非永远很智能 sometimes会卡我们编译)





## 2.8 内嵌字符串 ~~没啥~~

```kotlin
val a: Int =10
println("a = ${a}")
//or
println("a = $a")
```

















