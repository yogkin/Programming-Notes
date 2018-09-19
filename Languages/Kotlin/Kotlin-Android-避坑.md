[TOC]

## 1 与Retrofit配合使用时存在的问题


Kotlin会给泛型加上通配符，比如`Map<String, RequestBody>`在Java中会被认为是`Map<String, ？ extend RequestBody>`，但是在Retrofit规定接口中的参数不能有通配符，所以使用Kollin接口定于API时就可能会遇到这个问题：[Parameter type must not include a type variable or wildcard](https://github.com/square/retrofit/issues/1805)

解决方案是加上注解`@JvmSuppressWildcards`：

```kotlin
@PUT("/api/v3/{storeId}/categories/{categoryId}")
fun update(@Path("storeId") storeId: Int, @Path("categoryId") categoryId: Int?, @Query("token") token: String, @Body category: Map<String, @JvmSuppressWildcards Any>): Observable<OperationResultUpdated>
```

---
## 2 RxJava2的combineLatest泛型丢失

```kotlin
    private var userNameObservable: Observable<CharSequence>? = null
    private var passwordObservable: Observable<CharSequence>? = null

    fun initialize() {
        userNameObservable = RxTextView.textChanges(username).skip(1)
            .debounce(500, TimeUnit.MILLISECONDS)
        passwordObservable = RxTextView.textChanges(password).skip(1)
            .debounce(500, TimeUnit.MILLISECONDS) 
      }

    //下面代码编译无法通过，泛型推导失败
    Observable.combineLatest(userNameObservable,
            passwordObservable, 
            { u: CharSequence, p: CharSequence -> u.isNotEmpty() && p.isNotEmpty() })
```

参考：[Observable.combineLatest type inference in kotlin](https://stackoverflow.com/questions/42725749/observable-combinelatest-type-inference-in-kotlin)

---
## 3 Suppress

kotlin中的压制警告语法变量，具体如下：

-  `@Suppress("UNCHECKED_CAST")`

---
## 4 反编译 Kotlin

Kotlin 反编译时，在 IDEA 上进行 decompile 可能造成 IDEA 卡死，而使用 AndroidStuidio 则不会。


---
## 5 Gson 和 DataClass

DataClass 没有默认的构造函数，而 Gson 却可以通过 `Unsafe.allocateInstance()` 类绕过构造函数实例化对象，这样实例化出来的对象的字段都是没有被初始化的，基于这种情况，就可能存在问题：

```kotlin
//父类
abstract class PagingWrapper<T>{

    abstract fun getElements(): List<T>

    /*这里把 getElements 添加到 GitHubPaging中，是为了给上层使用*/
    val paging by lazy {
        GitHubPaging<T>().also { it.addAll(getElements()) }
    }
}

//基类
data class SearchRepositories(var total_count: Int,
                              var incomplete_results: Boolean,
                              var items: List<Repository>) : PagingWrapper<Repository>() {

    override fun getElements() = items

}
```

PagingWrapper 中 paging 是懒加载的，反编译之后会发现实现是这样的：

```kotlin
public abstract class PagingWrapper {

   @NotNull
   private final Lazy paging$delegate = LazyKt.lazy((Function0)(new Function0() {
      public Object invoke() {
         return this.invoke();
      }

      @NotNull
      public final GitHubPaging invoke() {
         GitHubPaging var1 = new GitHubPaging();
         var1.addAll((Collection)PagingWrapper.this.getElements());
         return var1;
      }
   }));

   @NotNull
   public abstract List getElements();

   @NotNull
   public final GitHubPaging getPaging() {
      Lazy var1 = this.paging$delegate;
      KProperty var3 = $$delegatedProperties[0];
      return (GitHubPaging)var1.getValue();
   }
}
```

layz 依赖于 paging$delegate 字段，但是如果是 Unsafe 通过 `Unsafe.allocateInstance()` 实例化 SearchRepositories 的话，paging$delegate 是没有被初始化的，它的值还是null，这样后面调用 paging 必然会抛出 NPE。