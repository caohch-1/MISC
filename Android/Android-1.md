---
title: Android-1
categories:
- Android
---

基于《第一行代码——Android》@郭霖

# 第一章:开始启程

## 1.3

### **如何使用 res文件夹 下的资源**

In Codes

```kotlin
R.string.app_name
```

In XML

```xml
@string/app_name
```



引用图片资源drawable/ 应用图标mipmap/ 布局文件layout/ 字符串string

i.e.

```xml
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        ...
    </application>
```



## 1.4

### **使用日志工具Log而不是println()**

```kotlin
Log.v() //verbose
Log.d() //debug
Log.i() //info
Log.w() //warn
Log.e() //error
```





















