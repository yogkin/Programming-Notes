# Retrofit 使用

- Retrofit是一个简化Http请求的库，官方的定义是：A type-safe **HTTP client** for Android and Java，特点就是用接口和注解来描述http请求，定义即实现。Retrofit使用的是REST API。
- 从2.0版本开始较之前的版本变化很大，其原理是通过反射接口中方法的注解，根据注解获取的信息为接口创建一个代理对象，实际调用的是代理对象的方法

---
## 1 相关资料

Retrofit1.9 资料

- [Retrofit1.9介绍](http://www.tuicool.com/articles/NnuIva)
- [源码解析](http://www.cnblogs.com/angeldevil/p/3757335.html)
- [使用教程与源码解析](http://www.cnblogs.com/laiqurufeng/p/4484916.html)

Retorfit2.0 资料

- [Retrofit2.0有史以来最大改进-中文](https://realm.io/cn/news/droidcon-jake-wharton-simple-http-retrofit-2/)
- [Retrofit 2.0 + OkHttp 3.0 配置](https://drakeet.me/retrofit-2-0-okhttp-3-0-config)
- [Retrofit2 完全解析 探索与okhttp之间的关系](http://blog.csdn.net/lmj623565791/article/details/51304204)
- [你真的会用Retrofit2吗?Retrofit2完全教程](http://www.jianshu.com/p/308f3c54abdd)
- [深入浅出 Retrofit，这么牛逼的框架你们还不来看看？](http://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=2653577186&idx=1&sn=1a5f6369faeb22b4b68ea39f25020d28&scene=1&srcid=06039K4A2eGkHPxLbKED09Mk#wechat_redirect)


---
## 2 Retrofit 2 介绍

以下内容整理自 [Retrofit2.0有史以来最大改进-中文](https://realm.io/cn/news/droidcon-jake-wharton-simple-http-retrofit-2/)

### URL的定义

```
    Gson gson = new GsonBuilder().setDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'")
                .serializeNulls()
                .create();

    Retrofit retrofit = new Retrofit.Builder()
            .baseUrl("http://gank.avosapps.com/api/")
            .client(okHttpClient)
            .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
            .addConverterFactory(GsonConverterFactory.create(gson))
            .build();

    GanKService service =  retrofit.create(GanKService.class);
```

BaseUrl

```
        @GET("data/福利/20/{page}")
        Call<MeiZiList.MeiZi> getMeiZi(@Path("page") int pageNumber);
    
        @GET("/data/video/20/{page}")
        Call<MeiZiList.MeiZi> getMeiZi(@Path("page") int pageNumber);
    
        @GET("http://gank.com/data/福利/20/{page}")
        Call<MeiZiList.MeiZi> getMeiZi(@Path("page") int pageNumber);
```

这三种定义最终得到的Url是不一样的，如果page=1，那么分别是

```
    http://gank.avosapps.com/api/data/福利/20/1;
    
    http://gank.avosapps.com/data/video/20/1；
    
    http://gank.com/data/福利/20/1
```

可见：第一种方式是用于直接拼接的，也就是相对路径，第二种方式注解Url前面有"/",表示绝对路径，会从com后面开始拼接，第三种方法就是完全替代，忽略在Retrofit中设置的BaseUrl了

### 动态的Url

还有一种方式用于实现动态的url，就是一个url注解，这种情况适用于当一系列数据中，下一页的数据地址从上一次请求的header中返回

```
    @GET
    Call<List<Contributor>> repoContributorsPaginate(@Url String url);
```

### 同步与异步，可取消的Call

#### 同步与异步

1.9之前如果需要同步异步，需要定义两个方法，现在只需要定义一个，返回一个Call对象，用Call来操作

定义方法：

```
        @GET("data/福利/20/{page}")
        Call<MeiZiList> getMeiZi(@Path("page") int pageNumber);
```

同步操作为：

```
       GanKService ganKService = retrofit.create(GanKService.class);
    
            Call<MeiZiList> meiZi = ganKService.getMeiZi(1);
            try {
                
                Response<MeiZiList> execute = meiZi.execute();
    
            } catch (IOException e) {
                e.printStackTrace();
            }
```

异步操作为：

```
     GanKService ganKService = retrofit.create(GanKService.class);
    
            Call<MeiZiList> meiZi = ganKService.getMeiZi(1);
            meiZi.enqueue(new Callback<MeiZiList>() {
                @Override
                public void onResponse(Response<MeiZiList> response, Retrofit retrofit) {
                    
                }
    
                @Override
                public void onFailure(Throwable t) {
    
                }
            });
```

按照安卓的习惯，一般使用异步操作，而且不需要捕获异常，**注意**一个Call只能被执行一次，多次执行会产生异常，如果要执行一个相同的请求，可以clone一个相同的call，代价很低。


#### 取消一个call

2.0之前的版本不支持取消一个正在执行的请求，现在可以轻易的做到了

```
    call.cancel();
```

### Converters

1.9 默认使用Gson来进行对象的序列化与反序列化，现在Converters从Retrofit中移除，如果不显性的声明一个可用的 Converter 的话，Call的泛型只能定义为ResponseBody：

```
        @GET("http://10.2.20.13:8080/day21FileUpload/test1.json")
        Call<ResponseBody> getMeiZiString();

        public void onResponse(Response<ResponseBody> response, Retrofit retrofit) {
             //可以从response.body().string()中获取响应
            Log.d(TAG, "response.body():" + response.body().string());
        }
```

官方提供了一些Converters的是实现：

```
    Gson: com.squareup.retrofit2:converter-gson:2.0.0
    Jackson: com.squareup.retrofit2:converter-jackson:2.0.0
    Moshi: com.squareup.retrofit2:converter-moshi:2.0.0
    Protobuf: com.squareup.retrofit2:converter-protobuf:2.0.0
    Wire: com.squareup.retrofit2:converter-wire:2.0.0
    Simple XML: com.squareup.retrofit2:converter-simplexml:2.0.0
```

当然也可以实现自己的Converter.Factory

```
    public class TestConverter extends Converter.Factory {
    
        private TestConverter() {
            super();
        }
    
        public static TestConverter create() {
            return new TestConverter();
        }
        
        // json-->实例化反象
        @Override
        public Converter<ResponseBody, ?> responseBodyConverter(Type type, Annotation[] annotations, Retrofit retrofit) {
    
        }

        //对象-->json(当直接拿一个javaBean作为请求参数时，会调用此方法把对象序列化成可用参数)
        @Override
        public Converter<?, RequestBody> requestBodyConverter(Type type, Annotation[] parameterAnnotations, Annotation[] methodAnnotations, Retrofit retrofit) {
    
        }
    
        @Override
        public Converter<?, String> stringConverter(Type type, Annotation[] annotations, Retrofit retrofit) {
    
        }
    }
```

现在一个Retrofit可以添加多个Converters，用jake-wharton的话来说就是：
>如果你遇到这种情况：一个 API 请求返回的结果需要通过 JSON 反序列化，另一个 API 请求需要通过 proto 反序列化，唯一的解决方案就是将两个接口分离开声明,之所以搞得这么麻烦是因为一个 REST adapter 只能绑定一个 Converter 对象。

多个Converters如果工作？

```
    SomeProtoResponse —> Proto? Yes!
    SomeJsonResponse —> Proto? No! —> JSON? Yes!
```

>原理很简单，其实就是对着每一个 converter 询问他们是否能够处理某种类型。我们问 proto 的 converter： “Hi, 你能处理 `SomeProtoResponse` 吗？”，然后它尽可能的去判断它是否可以处理这种类型。我们都知道：Protobuff 都是从一个名叫 message 或者 message lite 的类继承而来。所以，判断方法通常就是检查这个类是否继承自 message。在面对 JSON 类型的时候，首先问 proto converter，proto converter 会发现这个不是继承子 Message 的，然后回复 no。紧接着移到下一个 JSON converter 上。JSON Converter 会回复说我可以！

需要注意的是：因为 JSON 并没有什么继承上的约束。所以我们无法通过什么确切的条件来判断一个对象是否是 JSON 对象。以至于 JSON 的 converters 会对任何数据都回复说：我可以处理！这个一定要记住， **JSON converter 一定要放在最后**，不然会和你的预期不符。

现在converter的很高效的：
>现在这已经高效的多，因为我们实际上可以查询那些 adapter。举个例子，GSON 有一个type adapter, 当我们请求 GSON Converter Factory，询问它是否可以处理某种请求的时候， converter factory 就开始查询这个 adapter, 它将以缓存的形式存在，当我们再次查询的时候，这个 adapter 就可以直接被使用了。这是一个非常小的成功，极力避免了不断地查询带来的损耗。
call adapter 有相同的模式。我们询问一个 call adapter factory 它是否可以处理某个类型，它将会以相同的方式回应。（例如：它会返回 null 来表达否）。它的API 是非常简单的。

### CallAdapter 更多可替换的执行机制

比如一个方法返回Call
```
    interface GitHubService {
      @GET("/repos/{owner}/{repo}/contributors")
      Call<List<Contributor>> repoContributors(
          @Path("owner") String owner,
          @Path("repo") String repo);
```
但是也可以定义下面这些方法：
```
      @GET("/repos/{owner}/{repo}/contributors")
      Observable<List<Contributor>> repoContributors2(
          @Path("owner") String owner,
          @Path("repo") String repo);
    
      @GET("/repos/{owner}/{repo}/contributors")
      Future<List<Contributor>> repoContributors3(
          @Path("owner") String owner,
          @Path("repo") String repo);
    }
```
CallAdapter跟converter 的工作原理很像

>通过返回类型来判断需要调用哪个 exection。比如说：返回为 Call 的类型， 我们的整个执行机制会问：“Hey，你知不知道如何处理 Call ？”　如果是 RxJava，它就会说：“我不知道，我只知道　Observable 的处理方法。”。　随后，我们又问内部的 converter，他刚好回答说：“是的！我会！”。

```
    call —> RxJava? No! —> Call? Yes!
    
    Observable —> RxJava? Yes!

    Future —> RxJava? **No!** —> Call? **No!** —> Throw!
```

可以看到Retrofit2.0依然支持RxJava，但是需要添加：RxJavaCallAdapterFactory：

```
    compile com.squareup.retrofit2:adapter-rxjava:2.0.0
```

当然也可实现自己的CallAdapterFactory


### 现在必须依赖OKHttp

>Retrofit 2 现在开始依赖了 OkHttp 了，而且这部分不再支持替换。这是一件比较有争议的事情。但是希望我能证明为什么这是一个对的决定。
OkHttp 现在很小而且很聚焦，有很多好用的 API 接口。我们在 Retrofit 2 里都有对 OkHttp 的接口映射，也基本具备了我们需要的所有的特性，包括提到的所有的抽象类们。这些都超赞！这是压缩 Retrofit 库大小的一个法宝。我们最终减小了Retrofit 60% 的体积，同时又具有了更多的特性

OKHttp依赖OkIO进行io操作，okio是一个很高效的io操作库

### 如何处理回调

#### 参数化的 Response 类型

 Response 对象增加了曾经一直被我们忽略掉的重要元数据：响应码（the reponse code），响应消息（the response message），以及读取相应头（headers）。
```
     class Response<T> {
      int code();
      String message();
      Headers headers();
    
      boolean isSuccess();
      T body();
      ResponseBody errorBody();
      com.squareup.okhttp.Response raw();
    }
```

>同时还提供了一个很方便的函数来帮助你判断请求是否成功完成，其实就是检查了下响应码是不是 200。然后就拿到了响应的 body 部分，另外有一个单独的方法获取 error body。基本上就是出现一个返回码，然后调用相对应的函数的。只有当响应成功以后，我们会去做反序列化操作，然后将反序列化的结果放到 body 回调中去。如果出现了返回了网络成功响应（返回码：200）却最终返回 false 的情况，我们实际上是无法判断返回到底是什么的，只能将`ResponseBody`（简单封装的了 content-type，length，以及 raw body部分） 类型交给你去处理

#### 对象序列化错误依然会调用 onResponse

当响应成功以后，会把数据解析为对象，然后将解析得到的对象赋值给response的body，就算解析出错，onResponse也会被调用，此时body为null
```
      public void onResponse(Response<MeiZiList> response, Retrofit retrofit) {
    
                    if (response.isSuccess()) {
                        MeiZiList body = response.body();
    
                        if (body == null) {
                            ResponseBody responseBody = response.errorBody();
                        }
                    }
```

### 如果response存在什么问题，比如404什么的，onResponse也会被调用

可以从`response.errorBody().string()`中获取错误信息的主体

### 日志系统被移除，使用OkHttp的interceptor

>**日志功能还没有完成**，在 Retrofit 1 里是有日志的，但是在 Retrofit 2 里面没有。依赖 OkHttp 的一个优点是你实际上可以使用一个 interceptor 来实现实际的底层的请求和响应日志。因此，对于原始请求和响应，我们并不需要它，但是我们很可能需要日志来记录 Java 类型。

>最后，在我有空的时候，我想在 Retrofit 2 里支持 **WebSocket**。在 2.0 里很可能无法实现，但是我想在后续的2.1 版本里会加入支持。

```
    OkHttpClient client = new OkHttpClient();
    client.interceptors().add(..);
    
    Retrofit retrofit = new Retrofit.Builder()
        .baseUrl("https://api.github.com")
        .client(client)
        .build();
```

注意，如果在拦截器中获取了Response，需要重新设置

```
    ......
    if (response.body() != null) {
            // 深坑！
            // 打印body后原ResponseBody会被清空，需要重新设置body
            ResponseBody body = ResponseBody.create(contentType, bodyString);
            return response.newBuilder().body(body).build();
        } else {
            return response;
        }
```

### 添加依赖，开始使用

```
    dependencies {
      compile 'com.squareup.retrofit2:retrofit:2.0.0'
      compile 'com.squareup.retrofit2:converter-gson:2.0.0'
      compile 'com.squareup.retrofit2:adapter-rxjava:2.0.0'
      compile 'com.squareup.okhttp3:logging-interceptor:3.2.0'
    }
```

---
## 3 Retrofit 具体使用

### GET

```
        @GET("data/福利/20/{page}")
        Call<MeiZiList> getMeiZi(@Path("page") int pageNumber , @Query("type") String type , @QueryMap(encoded=true) Map<String ,String> stringStringMap);
```

- 注解get表示使用get方式请求数据
- @Path用于动态替换url中的对应的路径
- @Query用于在url后面添加查询参数，如 baseUrl?key1=value1&...
- @QueryMap类似，encoded表示是否进行urlencoder

### POST

#### 表单数据

```
    @FormUrlEncoded
    @POST("user/fuelcard/delete")
    Call<BaseResponse> delFuelcard(@Field("cardId") String cardId);
```

- FormUrlEncoded表示提交表单数据，每一键值对用Field表示
- 如果键值对较多，可以使用@FieldMap

#### Body

```
    @POST("/users/new")
    Call<User> createUser(@Body User user);
```

- 该对象将会被`Retroofit`实例指定的转换器转换，如果没有添加转换器，则只有`RequestBody`可用

#### Multipart

```
    @Multipart
    @PUT("/user/photo")
    Call<User> updateUser(@Part("photo") RequestBody photo, @Part("description") RequestBody description);
```

当函数有`@Multipart`注解的时候，将会发送`multipart`数据，Parts都使用`@Part`注解进行声明，Multipart parts要使用Retrofit的众多转换器之一或者实现`RequestBody`来处理自己的序列化。

#### 文件上传

```
        @Multipart
        @POST("http://10.2.20.13:8080/day21FileUpload/servlet/RegistServlet")
        Call<String> upDateMeiZiUserData( @Part("photo\"; filename=\"123.jpg\" ") RequestBody file , @Part("username") String name);

      File file = new File(Environment.getExternalStorageDirectory(), "123.jpg");
            RequestBody fileBody =  RequestBody.create(MediaType.parse("multipart/form-data"), file);
    
            mTestService.getMeiZiUserData(fileBody , "ztiany").enqueue(new Callback<String>() {
                @Override
                public void onResponse(Response<String> response, Retrofit retrofit) {
    
                }
    
                @Override
                public void onFailure(Throwable t) {
    
                }
            });
```

或者使用下面方式：

```
    @Multipart
    @POST("http://10.2.20.13:8080/day21FileUpload/servlet/RegistServlet")
    Observable<Response<PersonalInfo>> upDateMeiZiUserData(@PartMap Map<String, RequestBody> params);
    
    注意对于File的RequestBody需要这样创建：
    
    可以使用File或者byte[]。
    
    使用byte[]：
    RequestBody photo = RequestBody.create(MediaType.parse("image/*"), BitmapUtils.bitmapToBytes(bitmap));
    使用文件：
    RequestBody photo = RequestBody.create(MediaType.parse("image/*"), file);
    
    
    添加到map中
    params.put("photo\"; filename=\"" + file.getName() + "", fileRequestBody);
    同时添加文件和字段时：
    params.put("username",  RequestBody.create(null, "abc"));
```

### Header

#### 静态

```
    @Headers("Cache-Control: max-age=640000")
    @GET("/widget/list")
    Call<List<Widget>> widgetList();

    @Headers({
        "Accept: application/vnd.github.v3.full+json",
        "User-Agent: Retrofit-Sample-App"
    })
    @GET("/users/{username}")
    Call<User> getUser(@Path("username") String username);
```

需要注意的是：header不能被互相覆盖。所有具有相同名字的header将会被包含到请求中。

#### 动态

可以使用`@Header`注解动态的更新一个请求的header。必须给`@Header`提供相应的参数，如果参数的值为空header将会被忽略，否则就调用参数值的`toString()`方法并使用返回结果

```
    @GET("/user")
    Call<User> getUser(@Header("Authorization") String authorization)
```

### 文件下载，使用 `@Streaming`

在使用 Retrofit 下载文件的时候，默认情况下，Retrofit 在处理结果前会将整个 Server Response 读进内存，这在接收 JSON 或者 XML 等数据时不会存在什么问题，但如果是一个非常大的文件，就可能造成 `OutOfMemory`异常。

为了避免Retrofit将非常大的数据直接读进内存中，需要添加一个特殊的注解：

```
    @Streaming
    @GET
    Call<ResponseBody> downloadFileWithDynamicUrlAsync(@Url String fileUrl);
```

如果使用了 `@Streaming`，注意不要在 UI 线程读取网络内容，否则 Android 将会抛出`android.os.NetworkOnMainThreadException`异常。可以使用 RxJava 来解决这个问题。具体参考 [Retrofit 2 - 如何从服务器下载文件](http://www.jianshu.com/p/92bb85fc07e8)

### 其他

Retrofit 2.0 之前参数不能传 null 值，现在是可以的。
