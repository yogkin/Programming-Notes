# Kotlin 反射

可以说 Kotlin 的反射 API 有两套，一套是 kotlin 上的反射 api，另一套自然是在 kotlin 中使用 java 的发送 API。

---
## 1 Kotlin 反射 API

kotlin 反射 API 定义在 `kotlin.reflect`中。

---
## 2 Java 反射 API

在 kotlin 中同样可以使用 java 的反射 api，**效率上比 Kotlin 反射 API 快一些**，因为 kotlin 反射实际上是去读 类上标注的 MateData 注解上的信息，反编译任何一个 kotlin 类，都可以看到类上是添加了 MateData 注解的，这样可能 kotlin 反射能获取到更多的信息。

```java

@Metadata(
   mv = {1, 1, 11},
   bv = {1, 0, 2},
   k = 1,
   d1 = {"......"},
   d2 = {"Lcom/bennyhuo/github/utils/AdapterList;", "T", "Ljava/util/ArrayList;", "Lkotlin/collections/ArrayList;", "adapter", "Landroid/support/v7/widget/RecyclerView$Adapter;", "(Landroid/support/v7/widget/RecyclerView$Adapter;)V", "getAdapter", "()Landroid/support/v7/widget/RecyclerView$Adapter;", "add", "", "element", "(Ljava/lang/Object;)Z", "", "index", "", "(ILjava/lang/Object;)V", "addAll", "elements", "", "clear", "remove", "removeAll", "removeAt", "(I)Ljava/lang/Object;", "removeRange", "fromIndex", "toIndex", "set", "(ILjava/lang/Object;)Ljava/lang/Object;", "update", "production sources for module app"}
)
public final class AdapterList extends ArrayList {
   @NotNull
   private final Adapter adapter;
    
    ......
}
```

