# Retrofit 扩展

Retrofit 扩展主要有两个方面，一个是自定义类型转换器，另一个就是 CallAdapter，让 Retrofit 支持返回不同类型的结果，比如 RxJava。

---
## 1 CallAdapter

Retrofit 默认只支持返回 Call 类型，通过 CallAdapter 却带来了无限的扩展，其他各种返回值类型都是对原始 Call 调用的封装。

- 如何从 Retorfit + RxJava 中拿到 header，参考 [getting-header-information-with-rxjava-and-retrofit](https://stackoverflow.com/questions/26851459/getting-header-information-with-rxjava-and-retrofit)
- 比如 GitHub 的分页连接是放在 header 中的，那么如何扩展 RxJavaAdaper 以支持这样的分页方式呢？RxCallAdaper 的核心类是 `CallExecuteOnSubscribe、CallEnqueueOnSubscribe、CallArbiter`，call 方法用于分发结果。

---
## 2 让 Retrofit 支持 Kotlin 协程

- [retrofit-coroutines](https://github.com/tinsukE/retrofit-coroutines)
- [retrofit2-kotlin-coroutines-adapter](https://github.com/JakeWharton/retrofit2-kotlin-coroutines-adapter)
- [kotlin-coroutines-retrofit](https://github.com/gildor/kotlin-coroutines-retrofit)
- [Kotlin Coroutines — Handling concurrency like a pro (Retrofit2 + Coroutines)](https://proandroiddev.com/kotlin-coroutines-handling-concurrency-like-a-pro-retrofit2-coroutines-31cd0611fd91)
- [Porting Retrofit2 sample to koltin coroutines sample](https://medium.com/@raghunandan2005/retrofit2-and-koltin-coroutines-sample-938a6842b0a1)


---
## 3 Retrofit + RxJava 如何对网络结果进行统一处理

网络请求结果可能有以下情况：

如果服务器返回的格式是规范的，比如：
```
{
    "code":200,
    "msg":"success",
    "data":{}//具体业务数据
}
```
code、msg是必然会返回的，code 的值也是规范的，data 是具体业务需要的数据，或者没有数据时可以不传，对于这种情况可以对网络结果处理进行统一的封装。有三种方式：

- 使用 OkHttp 的拦截器，直接在拦截器中对返回的数据进行预处理，把 data 从源 json 中剥离出来，对 code 进行统一处理，不是 success 的 code 则以异常的形式抛出（异常会传递到RxJava链的下游），这种方式适用于简单的业务场景，对于业务非常复杂的 app，可能存在非常多的业务异常，而且很多异常都是某个业务所特有的，那么这种情况下做统一处理就会让拦截器非常耦合，另外如果发生异常，如何把 msg 传递到具体的调用者那里呢，当然是使用 Exception。
- 使用 RxJava 的 compose 操作，扩展一个 Transformer，对网络结果进行统一的处理，这样每个接口调用都要 compose 一下，但是扩展性很强。Transformer 中可以检查业务数据、剥离源数据中的 data，还可以视情况对 Transformer 各种拓展。
- 扩展 Retrofit，包括修改源码，增加新的注解，扩展 CallAdapter 和 ConvertAdapter 等。

如果服务器返回的格式不是规范的，比如：

```
//情况 1，具体业务数据和通用数据分开的
{
    "code":200,
    "msg":"success",
    "data":{}//具体业务数据
}
//情况 2：具体业务数据和通用数据耦合在一起
{
    "code":200,
    "msg":"success",
    "name":"Ztiany",
    "age" : 12
}
```

对于这种情况，有两种解决方案：

- 使用 OkHttp 的拦截器，对于情况 2 的数据，可以将其纠正为情况 1 的数据格式，这样在具体的业务中就可以做统一的处理了。
- 强制要求后台返回统一的数据格式。

总结，没有银弹，只能针对具体的业务情况选择相对合适的方案。

<br><br>
另外关于这个问题，也有一些讨论：

- [RxJava+Retrofit，在联网返回后如何先进行统一的判断？](https://www.zhihu.com/question/39182019)
- [RxJava 与 Retrofit 结合的最佳实践](https://gank.io/post/56e80c2c677659311bed9841)

---
## 4 Retrofit 功能增强

如果觉得 Retrofit 提供的注解还不够强大怎么办，自己扩展呗，比如有些特定接口(不是所有的)需要传一些固定的字段的场景，可以添加 `@FixedField` 注解来防止重复写重复的代码，具体参考下面文章。

- [HackRetrofit](https://github.com/enbandari/HackRetrofit)
- [Android 下午茶：Hack Retrofit 之 增强参数](http://bennyhuo.leanote.com/post/Android-Hack-Retrofit "全文")
- [Android 下午茶：Hack Retrofit (2) 之 Mock Server](http://bennyhuo.leanote.com/post/Android-Hack-Retrofit-Mock-Server "全文")