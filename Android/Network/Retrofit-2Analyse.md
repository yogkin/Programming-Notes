# Retorfit 源码分析

---
## 1 Retrofit

Retrofit 是整个库的门面，提供了同一个的API，而 Retrofit.Builder 是一个构建器，用于创建一个 Retrofit

Retrofit 内部保存了我们通过 Builder 设置的各种工厂对象和基本属性，所以 Retrofit 为内部的对象提供了以下功能：

- 根据类型和注解查找 CallAdapter
- 根据类型和注解查找 Converter，包括`RequestBody to 实体对象`、`ResponseBody to 实体对象`、`String to 实体对象`
- 获取Callback执行器，这个其实是从 Platform 对象获取的
- 最重要的功能，create方法，根据我们定义的的service接口创建一个动态代理实现，这个代理最终是通过 `ServiceMethod.parseAnnotations(Retrofit, Method);` 方法生成的

---
## 2 Call 与 OkHttpCall

### Call 的定义与作用

Call 是一个接口，其抽象一个请求的调度，其定义如下：

```java
public interface Call<T> extends Cloneable {
  Response<T> execute() throws IOException;//同步调度返回 retrofit中定义的响应
  void enqueue(Callback<T> callback);//异步调度，Callback将会在UI线程回调
  boolean isExecuted();//是否已经执行了
  void cancel();//取消这个调度
  boolean isCanceled();//是否已经取消
  Call<T> clone();//复制一个调度，复制后又可以调度了
  Request request();//这个调度的原始HTTP请求
}
```

- 默认情况下 Retrofit 只支持 Serivce 接口中定义 Call 类型的返回值，如果要支持其他的返回值类型（比如 RxJava），必须为 Retrofit 添加对应的 CallAdaper，这个 CallAdaper 的任务就是对 Call 进行包装，这里体验了 Retotfit 对适配器模式的应用
- 不管怎么包装，Call 都是最终的请求发起者，其他所有的 CallAdaper 都是对 Call 的再包装
- 正是因为 CallAdaper 和 Call 的抽象，才让 Retrofit 能够支持 Service 接口定义众多风格的返回类型

### OkHttpCall

Retrofit 使用的是 OkHttp作为其网络引擎，所以 OkHttpCall 是 Call 的请求实现。

- OkHttpCall 内部通过 RequestFactory 创建 OkHttp 的 Request
- OkHttpCall 用于执行 OkHttp 的网络请求，然后会将请求结果解析为 Retrofit 中定义的 Response

---
## 3 Callback

Callback 是 Retrofit 中定义的异步请求回调：

```java
public interface Callback<T> {
  void onResponse(Call<T> call, Response<T> response);
  void onFailure(Call<T> call, Throwable t);
}
```

---
## 4 HttpServiceMethod

HttpServiceMethod 继承了 ServiceMethod，顾名思义，ServiceMethod 表示一个服务方法，即对应我们在 Service 上定义的方法（Method），ServiceMethod 只定义了一个抽象方法 invoke。HttpServiceMetho 是 ServiceMethod 的唯一子类：

- HttpServiceMethod 也是通过构建者模式创建的，HttpServiceMethod.Builder 用于获取创建 HttpServiceMethod 必要的对象，并对 Method 上定义的 Annotation 和返回值类型进行必要的检测，比如 TypeVariable、WildcardType 等类型是不支持的
- Builder 在创建 HttpServiceMethod 时会创建 RequestFactory 实例，RequestFactory 用于根据 Method 和 Method 上的 Annotation 创建一个 OkHttpRequest
- 当调用 HttpServiceMethod 的 invoke 方法时，HttpServiceMethod 会调用内部成员 `CallAdapter.adapt(OkHttpCall)` ，adapt 用于适配原始的 Call
- OkHttpCall 用于根据 `RequestFactory、方法参数args、CallFactory、ResponseConverter` 包装一个 OkHttp 的 Call


---
## 5 RequestFactory

RequestFactory 用于解析我们在 Service 上定义的方法（Method），然后创建一个 OkHttp 的 Request。其内部逻辑相对复杂。


---
## 6 CallAdapter 和 CallAdapter.Factory

```java
public interface CallAdapter<R, T> {

  //返回此适配器在将HTTP响应主体转换为Java对象
  Type responseType();

  //返回适配的类型
  T adapt(Call<R> call);

  abstract class Factory {

    public abstract @Nullable CallAdapter<?, ?> get(Type returnType, Annotation[] annotations, Retrofit retrofit);

    //提取泛型参数的上限，比如 index 1 of  Map<String, ? extends Runnable> returns  Runnable
    protected static Type getParameterUpperBound(int index, ParameterizedType type) {
      return Utils.getParameterUpperBound(index, type);
    }

    //从 type 中提取原始类类型，比如  List<? extends Runnable> 返回  List.class
    protected static Class<?> getRawType(Type type) {
      return Utils.getRawType(type);
    }

  }
}
```

- CallAdapter 是一个适配器，用于包装原始的 Retofit Call，应用的是适配器模式
- CallAdapter.Factory 用于创建一个 CallAdapter，应用都是工程模式
- 对于 Android 平台，Retfoit 提供了 ExecutorCallAdapterFactory，ExecutorCallAdapterFactory 仅仅是把回调分发到主线程

---
## 7 Converter

```java
public interface Converter<F, T> {

  T convert(F value) throws IOException;

  abstract class Factory {

    public @Nullable Converter<ResponseBody, ?> responseBodyConverter(Type type,
        Annotation[] annotations, Retrofit retrofit) {
      return null;
    }

    public @Nullable Converter<?, RequestBody> requestBodyConverter(Type type,
        Annotation[] parameterAnnotations, Annotation[] methodAnnotations, Retrofit retrofit) {
      return null;
    }

    public @Nullable Converter<?, String> stringConverter(Type type, Annotation[] annotations,
        Retrofit retrofit) {
      return null;
    }

    protected static Type getParameterUpperBound(int index, ParameterizedType type) {
      return Utils.getParameterUpperBound(index, type);
    }

    protected static Class<?> getRawType(Type type) {
      return Utils.getRawType(type);
    }
  }
}
```

- Converter.Factory.requestBodyConverter()，RequestBody to 实体对象
- Converter.Factory.responseBodyConverter()，ResponseBody to 实体对象
- Converter.Factory.stringConverter()，String to 实体对象

---
## 8 Utils

泛型处理等工具方法

---
## 引用

- [Retrofit分析-经典设计模式案例](http://www.jianshu.com/p/fb8d21978e38)
- [Retrofit分析-漂亮的解耦套路](http://www.jianshu.com/p/45cb536be2f4)



