# 自动重试的场景

很多的接口都设计了token用于验证客户端的身份与状态，但是为了安全起见，token是有生命周期的，也就是说，一段时间内没有使用有效的token访问服务器，则token会被服务器置为无效状态。当发生token失效情况该怎么办呢？

 - 抛给用户，不可能，用户并不知道你token是什么鬼
 - 不告诉用户token失效，而是报一个网络异常，让用户再次刷新，用户体验不好
 - 自动重试
 
看来只有自动重试才是唯一的选择，但是怎么实现自动重试呢？


## 1 使用RxJava实现自动重试
 
 使用RxJaa的retryWhen操作符：

```java
    public class AutoRetry<T> implements Observable.Transformer<T, T> {
    
        @Override
        public Observable<T> call(final Observable<T> tObservable) {
            return tObservable.retryWhen(new Func1<Observable<? extends Throwable>, Observable<?>>() {
                @Override
                public Observable<?> call(Observable<? extends Throwable> observable) {
                    return observable
                            .flatMap(new Func1<Throwable, Observable<?>>() {
    
                                private AtomicInteger mAtomicInteger = new AtomicInteger(0);
    
                                @Override
                                public Observable<?> call(Throwable throwable) {
                                    int count = mAtomicInteger.incrementAndGet();
                                    if (count > 3 || !(ApiHelper.isNeedRetry(throwable))) {
                                        return Observable.error(throwable);
                                    } else {
                                        Timber.d("自动重试第%d次", count);
                                        return AccountManager.getInstance().autoLogin(throwable);
                                    }
                                }
                            });
                }
            });
        }
    }
```

如果个别接口，还是比较优雅的，但是如果每个接口都需要重试逻辑那也是烦操的，难不成每个网络请求都写一个compose(new AutoRetry())。那么还有没有其他方法呢？



# 方式2：在CallAdapter中加入自动重试

这方法应该算是一劳永逸的，但是没有研究CallAdapter的话可能感觉比较复杂，暂时没有研究

# 方式3：在拦截器中实现自动重试

拦截器的使用方式很简单，我们可以使用ApplicationInterceptor，在拦截器中优先读取网络结果，如果服务器返回的接口是ok的，那就构建恢复Response返回，如果不ok(比如token失效)，那么可以在拦截器更新token，然后再此请求之前的接口。具体的做法如下：




需要注意的是，Response的body只能被读取一次，在读取之后需要重新构建Response才能返回：

```java
return response.newBuilder()
                .body(ResponseBody.create(response.body().contentType(), response.body().string()))
                .build();
```

详细可以看：[retrofit/issues/1072](https://github.com/square/retrofit/issues/1072#issuecomment-139097973)


## 方式4：authenticator验证

这种方式需要服务器配合，详细可以看okhttp wiki
