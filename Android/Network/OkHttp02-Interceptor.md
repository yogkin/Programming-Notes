# OkHttp 拦截器

拦截器(Interceptors)是一种强有力的途径，来**监控**，**改写**和**重试**HTTP访问。下面是一个简单的拦截器，对流出的请求和流入的响应记录日志


例如实现一个简单的Log拦截器：

```
    class LoggingInterceptor implements Interceptor {
      @Override public Response intercept(Interceptor.Chain chain) throws IOException {
        Request request = chain.request();
    
        long t1 = System.nanoTime();
        logger.info(String.format("Sending request %s on %s%n%s",
            request.url(), chain.connection(), request.headers()));
    
        Response response = chain.proceed(request);
    
        long t2 = System.nanoTime();
        logger.info(String.format("Received response for %s in %.1fms%n%s",
            response.request().url(), (t2 - t1) / 1e6d, response.headers()));
    
        return response;
      }
    }
```

在拦截器的实现中，对`chain.proceed(request)`的一次调用时是关键的一步。这个看起来简单的方法是HTTP工作实际发生的地方，产生一个满足请求的响应。多个拦截器可以链接起来。假如说你有一个压缩拦截器和一个求和校验拦截器：你将需要决定数据是先压缩然后求和校验，或者先求和校验再压缩。OkHttp使用列表来追踪拦截器，多个拦截器是被有序的调用。

![](https://raw.githubusercontent.com/wiki/square/okhttp/interceptors@2x.png)

## Application Interceptors

拦截器被注册为应用(application)拦截器或者网络(network)拦截器。我们将用上面定义的`LoggingInterceptor`来展示这两者的区别。

要注册应用拦截器，在`OkHttpClient.interceptors()`方法返回的列表上调用`add()`方法：

```
    OkHttpClient client = new OkHttpClient();
    client.interceptors().add(new LoggingInterceptor());

    Request request = new Request.Builder()
        .url("http://www.publicobject.com/helloworld.txt")
        .header("User-Agent", "OkHttp Example")
        .build();
    
    Response response = client.newCall(request).execute();
    response.body().close();
```

代码中的URL`http://www.publicobject.com/helloworld.txt`重定向到`https://publicobject.com/helloworld.txt`, OkHttp会自动跟随这一重定向。我们的应用拦截器会被调用一次，方法`chain.proceed()`返回的响应是重定向后的响应：

```
    INFO: Sending request http://www.publicobject.com/helloworld.txt on null
    User-Agent: OkHttp Example

    INFO: Received response for https://publicobject.com/helloworld.txt in 1179.7ms
    Server: nginx/1.4.6 (Ubuntu)
    Content-Type: text/plain
    Content-Length: 1759
    Connection: keep-alive
```

我们可以通过`response.request().url()`同`request.url()`不同看出访问被重定向。这两个日志语句记录了两个不同了URL。

## Network Interceptors

注册网络拦截器的方法很类似。添加拦截器到`networkInterceptors()`列表，取代`interceptors()`列表：

```
    OkHttpClient client = new OkHttpClient();
    client.networkInterceptors().add(new LoggingInterceptor());

    Request request = new Request.Builder()
        .url("http://www.publicobject.com/helloworld.txt")
        .header("User-Agent", "OkHttp Example")
        .build();

    Response response = client.newCall(request).execute();
    response.body().close();
```

当我们运行这一段代码时，拦截器执行了两次。一次是初始的到`http://www.publicobject.com/helloworld.txt`的请求，另一次是重定向到`https://publicobject.com/helloworld.txt`的请求。

```
    INFO: Sending request http://www.publicobject.com/helloworld.txt on Connection{www.publicobject.com:80, proxy=DIRECT hostAddress=54.187.32.157 cipherSuite=none protocol=http/1.1}
    User-Agent: OkHttp Example
    Host: www.publicobject.com
    Connection: Keep-Alive
    Accept-Encoding: gzip
    
    INFO: Received response for http://www.publicobject.com/helloworld.txt in 115.6ms
    Server: nginx/1.4.6 (Ubuntu)
    Content-Type: text/html
    Content-Length: 193
    Connection: keep-alive
    Location: https://publicobject.com/helloworld.txt
    
    INFO: Sending request https://publicobject.com/helloworld.txt on Connection{publicobject.com:443, proxy=DIRECT hostAddress=54.187.32.157 cipherSuite=TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA protocol=http/1.1}
    User-Agent: OkHttp Example
    Host: publicobject.com
    Connection: Keep-Alive
    Accept-Encoding: gzip
    
    INFO: Received response for https://publicobject.com/helloworld.txt in 80.9ms
    Server: nginx/1.4.6 (Ubuntu)
    Content-Type: text/plain
    Content-Length: 1759
    Connection: keep-alive
```

上面的网络请求也包含更多的数据，例如由OkHttp添加的请求头`Accept-Encoding: gzip`，通知支持对响应的压缩。网络拦截器链有一个非空的连接，用来询问我们连接web服务器使用的IP地址和TLS配置。


### 选择应用拦截器还是网络拦截器

**Application Interceptors**

1.  不用担心中间过程的响应，例如重定向和重试。
2.  始终调用一次，即使HTTP响应来自于缓存。
3.  观察应用原始的意图。不用关注由OkHttp注入的头信息，例如`If-None-Match`。
4.  允许短路，不调用`Chain.proceed()`。
5.  允许重试，多次调用`Chain.proceed()`。

**Network Interceptors**

1.  能操作中间响应，例如重定向和重试。
2.  发生网络短路的缓存响应时，不被调用。
3.  观察将通过网络传输的数据。
4.  可以获取到携带请求的`connection`

## 改写请求

　　拦截器可以添加，移除或者替换请求头。它们也可以改变请求体。例如，如果你连接的web服务器支持，你可以用一个应用拦截器来压缩请求体。

```
    /** This interceptor compresses the HTTP request body. Many webservers can't handle this! */
    final class GzipRequestInterceptor implements Interceptor {
      @Override public Response intercept(Chain chain) throws IOException {
        Request originalRequest = chain.request();
        if (originalRequest.body() == null || originalRequest.header("Content-Encoding") != null) {
          return chain.proceed(originalRequest);
        }
    
        Request compressedRequest = originalRequest.newBuilder()
            .header("Content-Encoding", "gzip")
            .method(originalRequest.method(), gzip(originalRequest.body()))
            .build();
        return chain.proceed(compressedRequest);
      }
    
      private RequestBody gzip(final RequestBody body) {
        return new RequestBody() {
          @Override public MediaType contentType() {
            return body.contentType();
          }
    
          @Override public long contentLength() {
            return -1; // We don't know the compressed length in advance!
          }
    
          @Override public void writeTo(BufferedSink sink) throws IOException {
            BufferedSink gzipSink = Okio.buffer(new GzipSink(sink));
            body.writeTo(gzipSink);
            gzipSink.close();
          }
        };
      }
    }
```

### 改写响应

对应的，拦截器也可以改写响应头和改变响应体。这通常来讲比改写请求头更危险，因为它可能违背了web服务器的预期。如果你在一个微妙的情境下，并准备好去处理对应的后果，改写响应头是一种有力的方式来解决问题。例如，你可以修复服务器错误配置的缓存控制响应头，来优化对响应的缓存。

```
    /** Dangerous interceptor that rewrites the server's cache-control header. */
    private static final Interceptor REWRITE_CACHE_CONTROL_INTERCEPTOR = new Interceptor() {
      @Override public Response intercept(Chain chain) throws IOException {
        Response originalResponse = chain.proceed(chain.request());
        return originalResponse.newBuilder()
            .header("Cache-Control", "max-age=60")
            .build();
      }
    };
```

通常来讲，这种方式在补充相应的服务器问题修复时最有用。

## 可用性

OkHttp拦截器需要2.2版本以上。不幸的是，拦截器对`OkUrlFactory`，和任何依赖`OkUrlFactory`的库无效，包含Retrofit 1.8以下和Picasso 2.4版本以下。

## 总结

通过学习拦截器，大致上了解了拦截器的作用，以及`ApplicationInterceptor`或`NetworkInterceptor`的区别。`ApplicationInterceptor`在`OkHttpCore`处理请求前被调用，适合对请求做修改，而`NetworkInterceptor`在`OkHttpCore`处理请求后被调用，合适对401之类的响应码做处理或者改写服务器响应。