---
title: Android-11
categories:
- Android

---

基于《第一行代码——Android》@郭霖



# 第十一章：使用网络技术

## 11.1 WebView的用法

WebView控件让我们能够在APP中内嵌一个浏览器

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
		//webView已经在xml布局里写好
        webView.settings.javaScriptEnabled = true //settings可以设置一些浏览器属性
        webView.webViewClient = WebViewClient() //当需要从一个网页跳转到另一个网页时，目标网页仍然在WebView中显示，而非打开系统浏览器
        webView.loadUrl("https://okrara.cn") //传入网址即可展示相关内容
    }
}
```

连接网络需要权限声明，具体怎么做很简单，不说了



## 11.2 使用HTTP访问网络

WebView帮助我们完成了大量工作，为了直观地看出HTTP是如何工作的，我们通过手动发送HTTP请求的方式更加深入地理解这个过程

### 使用HttpURLConnection

首先需要获取HttpURLConnection的实例，一般只需创建一个URL对象，并传入目标的网络地址，然后调用以下 **openConnection**方法即可

```kotlin
val url=URL("https://okrara.cn")
val connection = url.openConnection() as HttpURLConnection
```



接着，我们可以设置一下HTTP请求所用的方法，常用的有两个 **POST**和 **GET**

```kotlin
connection.requestMethod = "GET"
```

接下来就可以进行一些自由定制

```kotlin
connection.connectTimeout = 8000 //连接超时毫秒数
connection.readTimeout = 8000 //读取超时毫秒数
```

之后再调用 **getInputStream**方法获取到服务器返回的输入流，剩下的任务就是对输入流进行读取

```kotlin
val input = connection.inputStream
```

最后可以调用 **disconnect**方法关闭这个HTTP连接

```kotlin
connection.disconnect()
```



i.e.

这里的 **ScrollView**控件第一次使用，借助它可以以滚动的形式查看屏幕外的内容

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:id="@+id/sendRequestBtn"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Send Request" />

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <TextView
            android:id="@+id/responseText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

    </ScrollView>


</LinearLayout>
```



基本符合之前介绍的流程

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        sendRequestBtn.setOnClickListener {
            sendRequestWithHttpURLConnection()

        }
    }

    private fun sendRequestWithHttpURLConnection() {
        //开启线程发起网络请求
        thread {
            var connection: HttpURLConnection? = null
            try {
                val response = StringBuilder()
                val url = URL("https://okrara.cn")
                connection = url.openConnection() as HttpURLConnection
                connection.connectTimeout = 8000
                connection.readTimeout = 8000
                val input = connection.inputStream
                //对获取到的输入流进行读取
                val reader = BufferedReader(InputStreamReader(input))
                reader.use {
                    reader.forEachLine {
                        response.append(it)
                    }
                }
                showResponse(response.toString())
            } catch (e: Exception) {
                e.printStackTrace()
            } finally {
                connection?.disconnect()
            }
        }
    }

    private fun showResponse(response: String) {
        runOnUiThread {
            //在这里进行UI操作
            responseText.text = response
        }
    }
}
```



那如果是提交数据给服务器呢？

看看下面的例子就能明白

注意提交的每条数据之间用**&**分开，每条数据都是键值对形式

```kotlin
connection.requestMethod = "POST"
val output = DataOutputStream(connection.outputStream)
output.writeBytes("username=admin&password=123456")
```



### 使用OkHttp

这是一个第三方的开源库，它比刚刚我们使用的更好用

首先添加依赖，这会添加两个库，OkHttp和Okio，后者是前者的通信基础 版本号请按照官网更改

```
dependencies {
    ...
    implementation 'com.squareup.okhttp3:okhttp:4.1.0'
    ...
}
```



接着需要先创建一个 **OkhttpClient**实例

```kotlin
val client = OkHttpClient()
```

接下来如果想要发起一条HTTP请求，就要创建一个 Request对象

```kotlin
val request = Request.Builder()
					 .url("https://okrara.cn")					 
					 .build()
```

之后调用 **newCall**方法来创建一个 **Call**对象，并调用它的 **execute**方法来发送请求并获取服务器返回的数据

```kotlin
val response = client.newCall(request).execute()
```

Response对象就是服务器返回的数据了，我们还可以获得返回的具体内容

```kotlin
val responseData = response.body?.string()
```

如果是发起POST请求，会比GET复杂一些，首先需要构造一个**Request Body**对象来存放待提交的参数

```kotlin
val requestBody = FormBody.Builder()
						  .add("username","admin")
						  .add("password","123456")
						  .build()
```

然后在 **Request.Builder**中调用 **post**方法，并将 **RequestBody**传入

```kotlin
val request = Request.Builder()
  					 .url("https://www.okrara.cn")
					 .post(requestBody)
					 .build()
```

接下来就和GET一样了，调用 **execute**方法



i.e.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        sendRequestBtn.setOnClickListener {
            sendResponseWithOkHttp()

        }
    }

    //Something

    private fun sendResponseWithOkHttp() {
        thread {
            try {
                val client = OkHttpClient()
                val request = Request.Builder()
                    .url("https://okrara.cn")
                    .build()
                val response = client.newCall(request).execute()
                val responseData = response.body?.string()
                if (responseData != null) {
                    showResponse(responseData)
                }
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }
    }
}
```



我在这里遇到了如下error

```kotlin
Expected Android API level 21+ but was 30
```

将**build.gradle(Module:app)**中的**targetSdkVersion**从30变成29即可通过编译



## 11.3 解析XML格式数据

在网络上传输数据最常用的两种格式： **XML**和 **JSON**

开始这段学习前，先要获取一段xml格式的数据

书上演示了win下通过Apache搭建简单Web服务器的方法

我这里是在Ubuntu20.04环境下，比win简单不少

```bash
sudo apt-get install apache2
sudo /etc/init.d/apache2 start
```

在**/var/www/html**下创建 **get_data.xml**

```xml
<apps>
    <app>
        <id>1</id>
        <name>Google Maps</name>
        <version>1.0</version>
    </app>
    <app>
        <id>2</id>
        <name>Chrome</name>
        <version>2.1</version>
    </app>
    <app>
        <id>3</id>
        <name>Google Play</name>
        <version>2.3</version>
    </app>
</apps>
```

OK搞定



### Pull解析方式

**MainActivity.kt**

不难看懂

```kotlin
class MainActivity : AppCompatActivity() {
    //SOmething

    private fun sendResponseWithOkHttp() {
        thread {
            try {
                val client = OkHttpClient()
                val request = Request.Builder()
                    .url("http://10.0.2.2/get_data.xml") //10.0.2.2对虚拟机而言就是计算机本机的IP地址
                    .build()
                val response = client.newCall(request).execute()
                val responseData = response.body?.string()
                if (responseData != null) {
                    parseXMLWithPull(responseData)
                }
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }
    }

    private fun parseXMLWithPull(xmlData: String) {
        try {
            val factory = XmlPullParserFactory.newInstance()
            val xmlPullParser = factory.newPullParser()
            xmlPullParser.setInput(StringReader(xmlData))
            var eventType = xmlPullParser.eventType
            var id = ""
            var name = ""
            var version = ""
            while (eventType != XmlPullParser.END_DOCUMENT) {
                val nodename = xmlPullParser.name
                when (eventType) {
                    //开始解析某个节点
                    XmlPullParser.START_TAG -> {
                        when (nodename) {
                            "id" -> id = xmlPullParser.nextText()
                            "name" -> name = xmlPullParser.nextText()
                            "version" -> version = xmlPullParser.nextText()
                        }
                    }
                    //完成解析某个节点
                    XmlPullParser.END_TAG -> {
                        if ("app" == nodename) {
                            Log.d("MainActivity", "id is $id")
                            Log.d("MainActivity", "name is $name")
                            Log.d("MainActivity", "version is $version")
                        }
                    }
                }
                eventType = xmlPullParser.next()
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
}
```

众所周知，HTTP由安全隐患，为了让程序使用HTTP，进行如下配置

创建 **src/main/res/xml/network_config.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
</network-security-config>
```



解下里修改 **AndroidManifest.xml**来启用配置文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.networktest">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        ...
        android:networkSecurityConfig="@xml/network_config">
		...
    </application>

</manifest>
```



### SAX解析方式

这种方式复杂一些，但在语义方面更加清晰

通常情况下我们新建一个类继承自 **DefaultHandler**并重写父类的5个方法

```kotlin
package com.example.networktest

import org.xml.sax.Attributes
import org.xml.sax.helpers.DefaultHandler

class ContentHandler : DefaultHandler() {
    //开始解析XML时调用
    override fun startDocument() {
        super.startDocument()
    }
	//开始解析某个节点时调用
    override fun startElement(
        uri: String?,
        localName: String?,
        qName: String?,
        attributes: Attributes?
    ) {
        super.startElement(uri, localName, qName, attributes)
    }
	//获取节点内容时调用
    override fun characters(ch: CharArray?, start: Int, length: Int) {
        super.characters(ch, start, length)
    }
	//完成解析某个节点时调用
    override fun endElement(uri: String?, localName: String?, qName: String?) {
        super.endElement(uri, localName, qName)
    }
	//完成整个XML解析时调用
    override fun endDocument() {
        super.endDocument()
    }
}
```





i.e.

```kotlin
class ContentHandler : DefaultHandler() {
    private var nodeName = ""
    private lateinit var id: StringBuilder
    private lateinit var name: StringBuilder
    private lateinit var version: StringBuilder

    override fun startDocument() {
        id = StringBuilder()
        name = StringBuilder()
        version = StringBuilder()
    }

    override fun startElement(
        uri: String,
        localName: String,
        qName: String,
        attributes: Attributes
    ) {
        //记录当前节点名
        nodeName = localName
        Log.d("ContentHandler", "uri is $uri")
        Log.d("ContentHandler", "localName is $localName")
        Log.d("ContentHandler", "qName is $qName")
        Log.d("ContentHandler", "attributes is $attributes")
    }

    override fun characters(ch: CharArray?, start: Int, length: Int) {
        //根据当前节点名判断将内容添加到哪一个StringBuilder对象中
        when (nodeName) {
            "id" -> id.append(ch, start, length)
            "name" -> name.append(ch, start, length)
            "version" -> version.append(ch, start, length)
        }
    }

    override fun endElement(uri: String?, localName: String?, qName: String?) {
        if ("app" == localName) {
            Log.d("ContentHandler", "id is ${id.toString().trim()}")
            Log.d("ContentHandler", "name is ${name.toString().trim()}")
            Log.d("ContentHandler", "version is ${version.toString().trim()}")
            //最后要把StringBuilder清空
            id.setLength(0)
            name.setLength(0)
            version.setLength(0)
        }
    }

    override fun endDocument() {
    }
}
```



**MainActivity.kt**

```kotlin
class MainActivity : AppCompatActivity() {
    //Something

    private fun sendResponseWithOkHttp() {
        thread {
            try {
                val client = OkHttpClient()
                val request = Request.Builder()
                    .url("http://10.0.2.2/get_data.xml")
                    .build()
                val response = client.newCall(request).execute()
                val responseData = response.body?.string()
                if (responseData != null) {
                    //change one line here
                    parseXMLWithSAX(responseData)
                }
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }
    }
    
	//Something
    
    private fun parseXMLWithSAX(xmlData: String) {
        try {
            val factory = SAXParserFactory.newInstance()
            val xmlReader = factory.newSAXParser().xmlReader
            val handler = ContentHandler()
            //将ContentHandler的实例设置到XMLReader中
            xmlReader.contentHandler = handler
            //开始执行解析
            xmlReader.parse(InputSource(StringReader(xmlData)))
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
}
```



## 11.4 解析JSON格式数据

### 使用JSONObject

很容易看懂，不多做解释

```kotlin
class MainActivity : AppCompatActivity() {
	//Something
    private fun sendResponseWithOkHttp() {
        thread {
            try {
                val client = OkHttpClient()
                val request = Request.Builder()
                    .url("http://10.0.2.2/get_data.json") //10.0.2.2对虚拟机而言就是计算机本机的IP地址
                    .build()
                val response = client.newCall(request).execute()
                val responseData = response.body?.string()
                if (responseData != null) {
                    parseJSONWithJSONObject(responseData)
                }
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }
    }
    
	//Something
    
    private fun parseJSONWithJSONObject(jsonData: String) {
        try {
            val jsonArray = JSONArray(jsonData)
            for (i in 0 until jsonArray.length()) {
                val jsonObject = jsonArray.getJSONObject(i)
                val id = jsonObject.getString("id")
                val name = jsonObject.getString("name")
                val version = jsonObject.getString("version")
                Log.d("MainActivity", "id is $id")
                Log.d("MainActivity", "name is $name")
                Log.d("MainActivity", "version is $version")
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
}
```

### 

### 使用GSON

更加简单，但是要先添加依赖

```
implementation 'com.google.code.gson:gson:2.8.5'
```

GSON强大之处在于可以将一段JSON格式的字符串自动映射成一个对象

比如如下一段JSON数据

```json
{"name":"Tom","age":20}
```

可以定义一个Person类，并且加入name和age字段，然后写如下代码

```kotlin
val gson = Gson()
val person = gson.fromJson(jsonData, Person::class.java)
```

如果要解析的是一段JSON数组，如下

```json
[{"name":"Tom","age":20},{"name":"Jack","age":25},{"name":"Lily","age":22}]
```

这时我们需要借助 **TypeToken**将期望解析成的数据类型传入 **fromJson**方法中

```kotlin
val typeOf = object : TypeToken<List<Person>>() {}.type
val people = gson.fromJson<List<Person>>(jsonData, typeOf)
```



以上是基本用法，下面举个例子看看

```kotlin
class MainActivity : AppCompatActivity() {
	//Something
    private fun sendResponseWithOkHttp() {
        thread {
            try {
                val client = OkHttpClient()
                val request = Request.Builder()
                    .url("http://10.0.2.2/get_data.json") //10.0.2.2对虚拟机而言就是计算机本机的IP地址
                    .build()
                val response = client.newCall(request).execute()
                val responseData = response.body?.string()
                if (responseData != null) {
                    parseJSONWithGSON(responseData)
                }
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }
    }
    
	//Something
    //我们已经创建了一个App类并拥有三个字段
    private fun parseJSONWithGSON(jsonData: String) {
        val gson = Gson()
        val typeOf = object : TypeToken<List<App>>() {}.type
        val appList = gson.fromJson<List<App>>(jsonData, typeOf)
        for (app in appList) {
            Log.d("MainActivity", "id is ${app.id}")
            Log.d("MainActivity", "name is ${app.name}")
            Log.d("MainActivity", "version is ${app.version}")
        }
    }
}
```



## 11.5 网络请求回调的实现方式

通常情况下我们应该将通用的网络操作提取到一个公共的类里，并提供一个通用方法

新建一个 Object

```kotlin
object HttpUtil {

    fun sendHttpRequest(address: String): String {
        var connection: HttpURLConnection? = null
        try {
            val response = StringBuilder()
            val url = URL(address)
            connection = url.openConnection() as HttpURLConnection
            connection.connectTimeout = 8000
            connection.readTimeout = 8000
            val input = connection.inputStream
            val reader = BufferedReader(InputStreamReader(input))
            reader.use {
                reader.forEachLine {
                    response.append(it)
                }
            }
            return response.toString()
        } catch (e: Exception) {
            e.printStackTrace()
            return e.message.toString()
        } finally {
            connection?.disconnect()
        }
    }
}
```

以后每当需要发起一条HTTP请求就可以这样写

```kotlin
val address = "https://okrara.cn"
val response = HttpUtil.sendHttpRequest(adress)
```

需要注意，网络请求通常是耗时操作，可能阻塞主线程

在 **sendHttpRequest**里开启一个线程也不可行，因为此处服务器响应的数据是无法返回的

这时，我们就需要使用编程语言的**回调机制**

下面就学习一下回调机制到底如何使用

首先定义一个接口

```kotlin
interface HttpCallbackListener {
    fun onFinish(response: String){
        // 得到服务器返回的具体内容
    }//当服务器成功响应我们的请求时调用
    fun onError (e:Exception) {
        //对异常进行处理
    }//网络操作出问题时调用
}
```

接着修改 **HttpUtil**

```kotlin
object HttpUtil {

    fun sendHttpRequest(address: String, listener: HttpCallbackListener) {
        thread {
            var connection: HttpURLConnection? = null
            try {
                val response = StringBuilder()
                val url = URL(address)
                connection = url.openConnection() as HttpURLConnection
                connection.connectTimeout = 8000
                connection.readTimeout = 8000
                val input = connection.inputStream
                val reader = BufferedReader(InputStreamReader(input))
                reader.use {
                    reader.forEachLine {
                        response.append(it)
                    }
                }
                //回调
                listener.onFinish(response.toString())
            } catch (e: Exception) {
                e.printStackTrace()
                //回调
                listener.onError(e)
            } finally {
                connection?.disconnect()
            }
        }
    }
}
```



上述方法总体上比较麻烦，使用 **OkHttp**会变简单

```kotlin
object HttpUtil {
	//Something
    
    fun sendOkHttpRequest(address: String, callback: okhttp3.Callback) {
        val client = OkHttpClient()
        val request = Request.Builder()
            .url(address)
            .build()
        client.newCall(request).enqueue(callback) //enqueue在内部已经帮我们开好线程，并将最终的请求结果回调到Callback中
    }
}
```



那么在调用时可以这么写

```kotlin
HttpUtil.sendOkHttpRequest(adress,object : Callback{
    override fun onResponse(call: Call,response:Response) {
        // 得到服务器返回的具体内容
        val responseData = response.body?.string()
    }
    
    override dun onFailure(call:Call,e:Exception) {
        //处理异常
    }
})
```



## 11.6 最好用的网络库：Retrofit

#### 基本用法

先添加依赖

由于Retrofit会借助GSON将JSON数据转换成对象，因此同样需要一个App类

```kotlin
class App(val id: String, val name: String, val version: String)
```

接下来，我们可以根据服务器接口的功能进行归类，创建不同种类的接口文件，并在其中定义对应具体服务器接口的方法

i.e.

```kotlin
interface AppService {
    @GET("get_data.json") //表示调用getAppData方法时会发起一条GET请求，地址就是注解中传入的具体参数，相对路径即可
    fun getAppData(): Call<List<App>>
}
```

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        getAppDataBtn.setOnClickListener {
            val retrofit = Retrofit.Builder()
                .baseUrl("http://10.0.2.2/")  //指定根目录
                .addConverterFactory(GsonConverterFactory.create()) //指定解析数据时用的转换库
                .build()
            val appService = retrofit.create(AppService::class.java)
            appService.getAppData().enqueue(object : Callback<List<App>> {
                override fun onResponse(call: Call<List<App>>, response: Response<List<App>>) {
                    val list = response.body()
                    if (list != null) {
                        for (app in list) {
                            Log.d("MainActivity", "id is ${app.id}")
                            Log.d("MainActivity", "name is ${app.name}")
                            Log.d("MainActivity", "version is ${app.version}")
                        }
                    }
                }

                override fun onFailure(call: Call<List<App>>, t: Throwable) {
                    t.printStackTrace()
                }
            })
        }
    }
}
```



#### 处理复杂的接口地址类型

为了方便举例，先定义一个Data类

```kotlin
class Data(val id:String, val content:String)
```

先从简单的看起，比如服务器的地址接口如下所示

```
GET http://example.com/get_data.json
```

接口地址是静态的，永远不会改变，对应到Retrofit中使用如下写法

```kotlin
interface ExampleService {
    @GET("get_data.json")
    fun getData():Call<Data>
}
```

但是接口不可能总是静态的，可能是如下情况

```
GET http://example.com/<page>/get_data.json
```

对应的Retrofit写法

```kotlin
interface ExampleService {
    @GET("{page}/get_data.json")
    fun getData(@Path("page") page:Int):Call<Data>
}
```

还有很多服务器会要求我们传入一系列参数

```
GET http://example.com/get_data.json?u=<user>&t=<token>
```

对应写法

```kotlin
interface ExampleService {
    @GET("get_data.json")
    fun getData(@Query("u") user:String, @Query("t") token:String) : Call<Data>
}
```

POST等提交数据的该怎么写呢，比如如下接口地址

```
POST http://example.com/data/create
{"id":1,"content":"The description for this data."}
```

对应的写法

```kotlin
interface ExampleService {
    @POST("data/create")
    fun createData(@Body data: Data): Call<ResponseBody>
}
```

最后，还有一些服务器接口可能还会要求我们在HTTP请求的header中指定参数，比如

```
GET http://example.com/get_data.json
User-Agent: okhttp
Cache-Control: max-age=0
```

对应写法（静态）

```kotlin
interface ExampleService {
    @Headers("User-Agent: okhttp","Cache-Control: max-age=0")
    @POST("data/create")
    fun createData: Call<Data>
}
```

对应动态写法

```kotlin
interface ExampleService {
    @POST("data/create")
    fun getData(@Headers("User-Agent") userAgent:String,
               @Headers("Cache-Control") cacheControl:String): Call<Data>
}
```



#### Retrofit构建器的最佳写法

创建一个单例类 **ServiceCreator**

```kotlin
object ServiceCreator {
    private const val BASE_URL = "http://10.0.2.2/"

    private val retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(GsonConverterFactory.create())
        .build()

    inline fun <reified T> create(): T = create(T::class.java)
}
```

这样使用起来会很简单，比如想要获取一个 **AppService**接口的动态代理对象

```kotlin
val appService = ServiceCreator.create(AppService::class.java)
```



# Kotlin拓展：协程编写高效并发程序

## 基本用法

添加依赖

```
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.1.1'
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.1.1'
```

开启协程

```kotlin
fun main() {
    GlobalScope.launch {
        println("codes run in coroutine scope")
    }
}
```

**GlobalScope.launch**创建的都是顶层协程，有什么办法让协程中所有代码都运行完之后再结束应用程序呢？

借助 **runBlocking**关键字

```kotlin
fun main() {
    runBlocking {
        println("codes run in coroutine scope")
        delay(1500)
        println("codes run in coroutine scope finished")
    }
}
```



如何创建多个协程呢？

借助 **launch**函数

```kotlin
fun main() {
    runBlocking {
        launch{
            println("launch1")
            delay(1000)
            println("launch1 finished")
        }
        
        launch{
            println("launch2")
            delay(1000)
            println("launch2 finished")
        }
    }
}
```



如果要把一部分代码放到一个单独的函数中，可是这个函数中没有协程作用域，那该怎么调用delay这种挂起函数呢？

借助 **suspend**关键字，把函数声明为挂起函数，挂起函数之间可以互相调用

```kotlin
suspend fun printDot(){
    println(".")
    delay(1000)
}
```



但是又有新的问题挂起函数还是没有协程作用域，那无法调用launch函数，怎么办呢？

借助 **coroutineScope**函数来解决，该函数本身也是个挂起函数，它的特点是会继承外部的协程作用域并创建一个子作用域，示例写法如下

```kotlin
suspend fun printDot() = coroutineScope{
    launch{
        println(".")
        delay(1000)
    }
}
```



## 更多的作用域构建器

launch可以创建一个新的协程，却不能获取执行结果

借助 **async**函数就可以实现，该函数必须在协程作用域中才能调用，它会创建一个子协程并返回一个 **Deferred对象**

```kotlin
fun main() {
    runBlocking{
        val result = async {
            5+5
        }.await()
        println(result)
    }
}
```

注意，当调用 **await**方法时，如果代码块中的代码还没执行完，那么 **await**会将当前的协程阻塞住，直到可以获得执行结果



最后再来学习一个比较特殊的作用域构建器： **withContext**函数，该函数是一个挂起函数，大体可以将他理解成 **async**函数的一种简化版写法，实例写法如下

```kotlin
fun main() {
    runBlocking {
        val result = withContext(Dispatchers.Default) { //Dispatchers.Default是该函数的必要参数--线程参数，有三种值可选
            5+5
        }
        println(result)
    }
}
```































