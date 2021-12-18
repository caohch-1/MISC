---
title: Android-9
categories:
- Android



---

基于《第一行代码——Android》@郭霖

# 第九章：运用手机多媒体

## 9.1 将程序运行到手机上

物理机连接电脑 USB调试



## 9.2 使用通知

### 创建通知渠道

首先需要一个 **NotificationManager**对通知进行管理，可以通过调用 **Context**的 **getSystemService**方法获取

该方法接收一个字符串参数用于确定获取系统的哪个服务

```kotlin
val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
```

接下来使用 **NotificationChannel**类构建一个通知渠道，并调用 **NotificationManager**的 **createNotificationChannel**方法完成创建

由于是Android 8.0中新增的的API，因此还需要先判断版本才行

```kotlin
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.0) {
    val channel = NotificationChannel(channelId, channelName, importance)
    manager.createNotificationChannel(channel)
}
```



### 通知的基本用法

首先使用 **Builder**构造器来创建 **Notification**对象

```kotlin
val notification = NotificationCompat.Builder(context, channelId).build()
```



更丰富一点

```kotlin
val notification = NotificationCompat.Builder(context, channelId)
	  .setContentTitle("This is content title")
	  .setContentText("THis is content text")
	  .setSmallIcon(R.drawable.samll_icon)  //只能使用纯alpha图层的图片进行设置
	  .setLargeIcon(BitmapFactory.decodeResource(getResource(), R.drawable.large_icon))
	  .build()
```



以上工作完成后，调用 **NotificationManager**的 **notify**方法就可以显示通知了

该方法接收两个参数 第一个是 Id 第二个是Notification对象

```kotlin
manager.notify(1, notification)
```



i.e.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel("normal", "Normal", NotificationManager.IMPORTANCE_DEFAULT)
            manager.createNotificationChannel(channel)
        }
        sendNotice.setOnClickListener {
            val notification = NotificationCompat.Builder(this, "normal")
                    .setContentTitle("This is content title")
                    .setContentText("THis is content text")
                    .setSmallIcon(R.drawable.small_icon)  //只能使用纯alpha图层的图片进行设置
                    .setLargeIcon(BitmapFactory.decodeResource(resources, R.drawable.large_icon))
                    .build()
            manager.notify(1, notification)
        }
    }
}
```





目前我们的通知点上去没啥反应 使用**PendingIntent**，可以理解为在合适时机触发的**Intent**

回过头看 **NotificationCompat.Builder**，它可以连缀一个 **setContentIntent**方法

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel("normal", "Normal", NotificationManager.IMPORTANCE_DEFAULT)
            manager.createNotificationChannel(channel)
        }
        sendNotice.setOnClickListener {
            val intent = Intent(this, NotificationActivity::class.java)
            //第二个参数一般用不到 传0即可 第三个参数Intent对象 第四个确定PendingIntent行为 一般传入0
            val pi = PendingIntent.getActivity(this, 0, intent, 0)

            val notification = NotificationCompat.Builder(this, "normal")
                    .setContentTitle("This is content title")
                    .setContentText("THis is content text")
                    .setSmallIcon(R.drawable.small_icon)  //只能使用纯alpha图层的图片进行设置
                    .setLargeIcon(BitmapFactory.decodeResource(resources, R.drawable.large_icon))
                    //添加PendingIntent
                    .setContentIntent(pi)
                    //点击通知后通知自动消失
                    .setAutoCancel(true)
                    .build()
            manager.notify(1, notification)
        }
    }
}
```



### 进阶用法

#### setStyle方法

构建富文本的通知内容 

该方法接收一个 **NotificationCompat.Style**参数

完全显示长文本

```kotlin
.setStyle(NotificationCompat.BigTextStyle().bigText("ceafguaergvabuvgragfaru" +
                            "egfueraguearfuagruifgtgfeagfeuriavuierugierufvgreugfuivbyufuberugfuergu" +
                            "ergeruiguieqrgeruiwgiurgewurguewrgiuewr" +
                            "gewruigerugerugerugergfuierigewrugewruguieuioerwguewrg"))
```

也可以显示图片

```kotlin
.setStyle(NotificationCompat.BigPictureStyle().bigPicture(
    BitmapFactory.decodeResource(
        resources, R.drawable.big_image)))
```



#### 不同重要等级的具体效果

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel("normal", "Normal", NotificationManager.IMPORTANCE_DEFAULT)
            //change following 2 lines
            val channel2 = NotificationChannel("importance", "Importance", NotificationManager.IMPORTANCE_HIGH)
            manager.createNotificationChannel(channel2)
        }
        sendNotice.setOnClickListener {
            val intent = Intent(this, NotificationActivity::class.java)
            val pi = PendingIntent.getActivity(this, 0, intent, 0)

            val notification = NotificationCompat.Builder(this, "importance") //change "normal" to "importance"
                    .setContentTitle("This is content title")
                    .setContentText("wabibabi,wabibabu")
                    .setSmallIcon(R.drawable.small_icon)
                    .setLargeIcon(BitmapFactory.decodeResource(resources, R.drawable.large_icon))
                    .setContentIntent(pi)
                    .setAutoCancel(true)
                    .build()
            manager.notify(1, notification)
        }
    }
}
```



现在会变成最烦人的弹出横幅



## 9.3 调用摄像头和相册

#### 调用色相头

略微复杂 直接看书吧 这里展示代码

**src/main/java/com/example/cameraalbumtest/MainActivity.kt**

```kotlin
class MainActivity : AppCompatActivity() {
    val takePhoto = 1
    lateinit var imageUri: Uri
    lateinit var outputImage: File

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        takePhotoBtn.setOnClickListener {
            //创建File对象，用于储存照片
            outputImage = File(externalCacheDir, "output_image.jpg")
            if (outputImage.exists()) {
                outputImage.delete()
            }
            outputImage.createNewFile()
            imageUri = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
                FileProvider.getUriForFile(
                    this,
                    "com.example.cameraalbumtest.fileprovider",
                    outputImage
                )
            } else {
                Uri.fromFile(outputImage)
            }
            //启动相机
            val intent = Intent("android.media.action.IMAGE_CAPTURE")
            intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri)
            startActivityForResult(intent, takePhoto)
        }
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        when (requestCode) {
            takePhoto -> {
                if (resultCode == Activity.RESULT_OK) {
                    //显示拍摄的照片
                    val bitmap =
                        BitmapFactory.decodeStream(contentResolver.openInputStream(imageUri))
                    imageView.setImageBitmap(rotateIfRequired(bitmap))
                }
            }
        }
    }

    private fun rotateIfRequired(bitmap: Bitmap): Bitmap {
        val exif = ExifInterface(outputImage.path)
        val orientation =
            exif.getAttributeInt(ExifInterface.TAG_ORIENTATION, ExifInterface.ORIENTATION_NORMAL)
        return when (orientation) {
            ExifInterface.ORIENTATION_ROTATE_90 -> rotateBitmap(bitmap, 90)
            ExifInterface.ORIENTATION_ROTATE_180 -> rotateBitmap(bitmap, 180)
            ExifInterface.ORIENTATION_ROTATE_270 -> rotateBitmap(bitmap, 270)
            else -> bitmap
        }
    }

    private fun rotateBitmap(bitmap: Bitmap, degree: Int): Bitmap {
        val matrix = Matrix()
        matrix.postRotate(degree.toFloat())
        val rotatedBitmap =
            Bitmap.createBitmap(bitmap, 0, 0, bitmap.width, bitmap.height, matrix, true)
        bitmap.recycle() //将不需要的Bitmap对象回收
        return rotatedBitmap
    }
}
```

**src/main/AndroidManifest.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.cameraalbumtest">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="com.example.cameraalbumtest.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">

            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>
    </application>

</manifest>
```

**src/main/res/xml/file_paths.xml**

```xml
<?xml version="1.0" encoding="utf-8" ?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path
        name="my_images"
        path="/" />
</paths>
```



#### 从相册中选择图片

```kotlin
class MainActivity : AppCompatActivity() {
    val takePhoto = 1
    //add line here
    val fromAlbum = 2
    lateinit var imageUri: Uri
    lateinit var outputImage: File

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
		//Something
        
        //add lines here
        fromAlbumBtn.setOnClickListener {
            //打开文件选择器
            val intent = Intent(Intent.ACTION_OPEN_DOCUMENT)
            intent.addCategory(Intent.CATEGORY_OPENABLE)
            //指定显示图片
            intent.type = "image/*"
            startActivityForResult(intent, fromAlbum)
        }
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        when (requestCode) {
            //Something
            
            //add lines here
            fromAlbum -> {
                if (resultCode == Activity.RESULT_OK && data != null) {
                    data.data?.let { uri ->
                        //将选择的图片显示
                        val bitmap = getBitmapFromUri(uri)
                        imageView.setImageBitmap(bitmap)
                    }
                }
            }
        }
    }

    private fun getBitmapFromUri(uri: Uri) = contentResolver
        .openFileDescriptor(uri, "r")?.use {
            BitmapFactory.decodeFileDescriptor(it.fileDescriptor)
        }

    //Something
    }
}
```





## 9.4 播放多媒体文件

### 播放~~阴~~音频

![MediaPlayer](https://s1.ax1x.com/2020/07/25/Ux3xSK.png)

播放音频一般使用 **MediaPlayer**类实现 以上是它的一些常用的控制方法

首先创建一个 **MediaPlayer**对象

然后调用 **setDataSource**方法设置音频文件的路径

再调用 **prepare**使其进入准备状态

接下来调用 **start**开始播放音频

调用 **pause**会暂停播放

再调用 **reset**会停止播放



**看例子吧**

注意先创建**src/main/assets/music.mp3**

 ```kotlin
class MainActivity : AppCompatActivity() {
    private val mediaPlayer = MediaPlayer()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        initMediaPlayer()
        play.setOnClickListener {
            if (!mediaPlayer.isPlaying) {
                mediaPlayer.start()
            }
        }
        pause.setOnClickListener {
            if (!mediaPlayer.isPlaying) {
                mediaPlayer.pause()
            }
        }
        stop.setOnClickListener {
            if (!mediaPlayer.isPlaying) {
                mediaPlayer.reset()
                initMediaPlayer()
            }
        }
    }

    private fun initMediaPlayer() {
        val assetManager = assets
        val fd = assetManager.openFd("music.mp3")
        mediaPlayer.setDataSource(fd.fileDescriptor, fd.startOffset, fd.length)
        mediaPlayer.prepare()
    }

    override fun onDestroy() {
        super.onDestroy()
        mediaPlayer.stop()
        mediaPlayer.release()
    }
}
 ```

### 

### 播放视频

和音频类似 不多说了

![ViedoView](https://s1.ax1x.com/2020/07/25/UxG280.png)



# Kotlin拓展

**使用infix函数构建更可读的语法**

直接看例子

```kotlin
infix fun String.beginsWith(prefix: String) = startsWith(prefix)
//接下来我们就可以这么使用
if ("Hello Kotlin" beginsWith "Hello") {
    //Something
}
```



注意：

- infix函数不能是顶层函数
- 必须且只能接收一个参数





















