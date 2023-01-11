在我的pids项目中，需要通过百度翻译平台获取线路中所有站点名的英文。不过百度翻译文档是没有提供SDK的，因此需要通过网络连接的方式向百度翻译发起请求再获取翻译结果。在这次迷你实验中会学习OkHttp和Retrofit，并使用他们分别完成网络连接的过程。

注册并获取百度翻译的api的步骤就不说了，可见[Android 百度翻译API（详细步骤+源码）](https://blog.csdn.net/qq_38436214/article/details/110117486?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522167305490416800211544574%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=167305490416800211544574&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-3-110117486-null-null.142^v70^control,201^v4^add_ask&utm_term=%E7%99%BE%E5%BA%A6%E7%BF%BB%E8%AF%91&spm=1018.2226.3001.4187)。

第一步，不要忘了要在AndroidManifest中添加网络访问权限
```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

为了方便叙述，先把要用到的依赖说一下。既然要用到okhttp和retrofit，这俩的依赖是一定要添加的；网络请求是耗时过程，必然不允许在主线程中执行网络请求操作。虽然okhttp有提供异步访问的方式，不过使用协程能让代码看起来更简洁。所以把协程的依赖也添加上了；翻译的结果是一段以json格式的字符串，使用gson库来处理会比较方便，那也加上吧。不过retrofit内部提供了对gson的支持，这需要添加retrofit的gson依赖
```groovy
implementation "com.squareup.okhttp3:okhttp:4.10.0"  
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.5.2"  
implementation 'com.google.code.gson:gson:2.10.1'  
implementation 'com.squareup.retrofit2:retrofit:2.9.0'  
implementation 'com.squareup.retrofit2:converter-gson:2.6.1'
```

首先把UI设置好
[[]]

然后开始实现逻辑。首先明确一下这个实验的任务，概括起来就三个步骤嘛：当okhttp或者retrofit的按钮按下时后，首先获取百度翻译的URL，然后向该URL发起请求，最后用得到的数据刷新界面
```kotlin
buttonOkhttp.setOnClickListener {  
    val url = TranslateUtil.getURL(getText())  
    CoroutineScope(Dispatchers.Main).launch {  
        val result = useOkHttp(url)  
        refreshUI(result)  
    }  
}

buttonRetrofit.setOnClickListener {  
    CoroutineScope(Dispatchers.Main).launch {  
        val result = useRetrofit(getText())  
        refreshUI(result)  
    }  
}
```

先来看okhttp。第一步获取url。百度翻译对url格式有要求，包含q、appid、from、to、salt、sign这些参数，具体细节应该前往百度翻译文档查看
```Kotlin
fun getURL(text: String): String {  
    val salt = Random().nextInt(100)  
    val sign = getMD5(appId + text + salt + key)  
    return "$httpsSite?appid=$appId&q=$text&from=zh&to=en&salt=$salt&sign=$sign"  
}  
  
fun getMD5(text: String): String {  
    val digest = MessageDigest.getInstance("MD5").digest(text.toByteArray())  
    val builder = StringBuilder()  
    for (bit in digest) {  
        if (bit.and(0xFF) < 0x10) {  
            builder.append("0")  
        }  
        builder.append(Integer.toHexString(bit.and(0xFF)))  
    }  
    return builder.toString()  
}
```

第二步发起请求。首先我使用了`withContext(Dispatchers.IO)`为当前协程切换了线程——不需要为线程切换而感到烦心，在离开withContext()的代码块后，协程就会恢复到了之前的Main线程

okhttp的访问十分简单，构建Request实例后，使用OkHttpClient实例发起`newCall().execute()`，最终得到Response实例，也就是http响应结果，我需要的json数据就在其中。从json中提取数据的步骤就交给Gson了。
```kotlin
private suspend fun useOkHttp(url: String): TranslateResult {   
    var result: TranslateResult?  
    withContext(Dispatchers.IO) {  
        val client = OkHttpClient()  
        val request = Request.Builder().url(url).build()  
        val response = client.newCall(request).execute()  
        result = Gson().fromJson(response.body?.string(), TranslateResult::class.java)  
    }  
    return result!!  
}
```

不过在使用Gson之前，我得先构建一个类，它必须有和json中字段相同的参数名。至于json的样式就得在百度翻译的文档中查看了。

构造好类之后，Gson才能如期工作。使用`Gson.fromJson()`后，Gson会将json中的内容一一填入我的类中
```kotlin
data class TranslateResult(val from: String, val to: String, val trans_result: List<TextResult>, val error_code: String, val error_message: String)  
  
data class TextResult(val src: String, val dst: String
```

第三步是数据刷新，就不说了吧。

先把Retrofit的使用叙述完再测试代码吧。Retrofit将访问网络的操作视为一种接口，所以首先要声明一个接口，并声明一个方法并声明其目标返回值，以供在访问网络时调用。向服务器索要数据通常使用Http的GET方法，GET注解中的内容表示要访问的URL中表示文件位置或者请求参数的一部分，注意不可以为空。Query注解表示url中`?`之后的部分，可以对照前面okhttp部分生成的url比对一下。
```kotlin
interface TranslateService {  
    @GET("?from=zh&to=en")  
    fun getTranslate(  
        @Query("appid") appId: String,  
        @Query("q") text: String,  
        @Query("salt") salt: String,  
        @Query("sign") sign: String  
    ): Call<TranslateResult>  
}
```

在构建Retrofit实例时既需要设置url（网站部分，即上一步url中未包含的部分），也需要设置将json转对象的工具。Retrofit实例使用`create()`将接口创造为服务，然后通过`execute()`执行该服务所提供的方法得到响应转换后的对象
```kotlin
private suspend fun useRetrofit(text: String):TranslateResult{  
    var result: TranslateResult?  
    withContext(Dispatchers.IO){  
        val salt = Random().nextInt(100)  
        val sign = TranslateUtil.getMD5(TranslateUtil.appId + getText() + salt + TranslateUtil.key)  
        val retrofit = Retrofit.Builder()  
            .baseUrl(TranslateUtil.httpsSite)  
            .addConverterFactory(GsonConverterFactory.create())  
            .build()  
        val service = retrofit.create(TranslateUtil.TranslateService::class.java)  
        result = service.getTranslate(TranslateUtil.appId, getText(), salt.toString(), sign).execute().body()!!
    }  
    return result!!  
}
```

我之前知道Retrofit是基于okhttp的，所以一直觉得应该要比okhttp更加方便才对，不过在这个实验中好像没什么特别突出的优势

最后来测试一下结果。“美的大道”是广州地铁7号线的终点站，地铁公司中它的英文是“Meidi Dadao”，搓搓手，看一下百度翻译的结果会是什么样的。
[[]]

嗯，有点尴尬，但好像确实也没错。其实在使用百度翻译前也想过它无法判断我的输入是地方名还是物体名，比如广州还有一个站叫黄沙，但是它的翻译结果是“sand”。

接下来就是技术无关、而且不太科学的闲聊时间了。

我在之后想到了一个新的方法，那就是向翻译暗示这是一个站名，做法是将翻译内容设为“下站黄沙”而不只是“黄沙”，这个做法是有效的，翻译结果变成了“Next Station Huangsha”。这样的话我就可以直接通过substring方法提取我只需要的站名。

但很快就否决了这个方法，因为“下站”的翻译结果在多次尝试中就出现了4个版本：“Next station”、“Next Stop”、“The Next Station is”、“The Next Stop is”。这可不妙啊

随后想到了另一个方法是将翻译内容设置为“黄沙站”，这样翻译结果就是“Huangsha Station”，来个substring()截取一下变成“Huangsha”皆大欢喜。不过如果翻译内容为“广州南站站”的话，截取后的结果就只有“Guangzhou South Railway”了。所以我做了一个十分不科学的做法，以缓解翻译不太准确的问题
```kotlin
for ((idx, name) in list.withIndex()) {  
    if (origin[idx].last() == '站') builder.append(name)  
    else builder.append(name.substring(0, name.length - 8))  
    builder.append(',')  
}
```
[[]]


参考：
[Android 百度翻译API（详细步骤+源码）](https://blog.csdn.net/qq_38436214/article/details/110117486?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522167305490416800211544574%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=167305490416800211544574&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-3-110117486-null-null.142^v70^control,201^v4^add_ask&utm_term=%E7%99%BE%E5%BA%A6%E7%BF%BB%E8%AF%91&spm=1018.2226.3001.4187)
[百度翻译开放平台](https://fanyi-api.baidu.com/product/113)
[okhttp官网](https://square.github.io/okhttp/)
[Android上的kotlin协程](https://developer.android.google.cn/kotlin/coroutines#kts)
[Retrofit](https://square.github.io/retrofit/)
[使用Retrofit调用百度翻译](https://blog.csdn.net/qq_43691887/article/details/123930857?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522167323763016800180628764%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=167323763016800180628764&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-1-123930857-null-null.142^v70^control,201^v4^add_ask&utm_term=retrofit%E7%99%BE%E5%BA%A6%E7%BF%BB%E8%AF%91&spm=1018.2226.3001.4187)
