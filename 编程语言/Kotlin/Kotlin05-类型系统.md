# Kotlin 类型系统

---
## 可空性

- Kotlin 对可空类型是显示支持的
- 类型的含义：类型就是数据的分类，决定了该类型可能的值，以及在该值上可能完成的操作——维基百科
- 虽然 Java 对 NEP 提供了一些解决工具，比如 Nullable 等注解、IDEA 的代码检查、但这些都没有从根本上解决 NEP 问题，Java8 的 Optional 是代码变得冗余以及会产生一些运行时性能的开销
- Kotlin 的可空类型在运行时并没有任何区别，所以不会产生任何开销
- 安全调用运算符 `s?.toUpperCase()` 相当于 `if(s !=null ) s.toUpperCase() else null`
- Elvis 运算符 `s?.length? : 0`
- return 和 throw 等于语句都是表达式，可以与 Elvis 运算符一起使用，比如 `val a = persion.company?: throw IllegalArgumentException()`
- 安全转换 `val person = p as? Person`，as? 尝试把 p 转换为 Persion 类型，如果值是不合适的类型则返回 null
- 安全转换也可以与 Elvis 一起使用，比如 `val otherPerson = o as Person?: return false`
- 非空断言：`!!`，如果非空断言发生错误，其异常信息只会告诉你哪行发生异常，如果最好不要在同一行使用多个非空断言
- 内建函数 `let` 让处理可空表达式变得更加容易：`emaile?.{ sendEmail(it) }`
- 某些类型需要被延迟初始化，kotlin 提供了 lateinit 关键字提供支持，但是 lateinit 不支持可空类型
- lateinit 常见的用法是用于依赖注入
- 可空类型同样支持扩展，而且让代码可以在可能为 null 的类型上调用方法，只有扩展方法能做到：`T?.method() = fun ...`
- 在可空类型的扩展上，this 可以为 null
- 一般情况下，应该把扩展函数定义在不可为空的类型上，如果后期需要再做修改
- 参数类型的可空性，Kotlin 中所有泛型类和泛型函数的类型参数默认都是可空的
- 要使类型产生非空，必须要为它指定一个非空上界：`fun <T:Any> print(t:T){ }`


---
## 可空性和 Java

- Java 中的 `@Nullable String` 会被 Kotlin  当作 `String?`，而 `@NotNull Sring` 就是 `String`
- Kotlin 支持多种风格的可空性注解，比如JSR-305、Android Anotation，JetBrains工具注解
- 平台类型：Java 类型在 Kotlin 中表现为平台类型，既可以把它当作非空类型，也可以把它当作可空类型
- 平台类型需要开发者自己根据情况进行选择，倒是变量是非空类型还是可空类型
- 为什么需要平台类型，如果以最安全的方式处理，认为所有的 Java 类型都是可空类型，那么在编码上会产生非常多的冗余的null检查，比如 Java 中的`List<String>` 在 Kotlin 中会被认为是 `List<String?>?`，每次访问都需要进行null检查，所以让开发者自己做出
- 当在 Kotlin 中继承 Java 类时，可以把参数类型和返回值类型定义成可空类型，也可以定义为非空类型
- 当在 Kotlin 中继承 Java 类时，如果把参数类型和返回值类型定义成非空类型，Kotlin 编译器会为每个定义成非空类型的变量生成非空断言

---
## 基本数据量类型

- 大多数情况下，Kotlin 中的 Int 类型，都会被编译为 Java 的基本类型 int，唯一不可行的例外是泛型类，当然其他基本类型处理方式一样
- 当在 Kotlin 中使用 Java 声明时，Java 基本基本类型会被当成非空类型，而不是平台类型，因为基本类型不可能为空
- Kotlin 中的可空类型不能用 Java 的基本类型表示，这意味着 Kotlin 中任何时候，只要使用了基本数据类型的可空版，都会被编译为 Java 基本类型的包装类型
- Kotlin 不会自动的从一种数组转换成另一种，为了避免意外行为，Kotlin 要求我们必须显示进行类型转换，尤其是在比较装箱值的时候

```kotlin
val x = 1
val list = listOf(1L, 2L, 3L)
// 假设Kotlin支持隐式类型转换，也许可以这么写
x in list
```

上面`x in list`的结果为 false，Kotlin 要求我们做显示的类转换，如果代码中同时用到了不同类型的数组类型，就必须显示地转换这些变量，以避免意向不到的行为

---
## 其他基本类型

- Any 和 Any? 类型，Any 是 Kotlin 中所有非空类型的超类型
- 在底层，Any 对应 Java 中的 java.lang.Object，Any 因此了某些方法(wait, notify)，但是可以通过显示转换为 Object 来调用
- Unit 类型：Kotlin 的 void，如果 Kotlin 函数使用 Unit，则 Java 中覆盖这个函数应该使用 void
- Nothing 类型：表示这个函数用于不会返回
- 返回 Nothing 的函数可以放在 Elvis 预算符的右边来做先决条件检查：`val address = company.address? : fail("no address")`，加上 fail 抛出异常，addrss 会被认为是非空类型，因为 address 为空时已经抛出异常了

---
## 可空性和集合

-  `List<Int?>` 能持有 Int? 类型的元素
- String.toIntOrNull() 是一个方便的函数，如果字符串无法解析为整数，将会返回 null
- 便利一个包含可空值的集合并过滤调 null 值是非常常见的操作，因此 Kotlin 提供了标准函数 `filterNotNull()` 来完成它

---
## 只读集合和可变集合

- Kotlin 的集合设计和 Java 不同之处在于，Kotlin 把访问集合的接口和修改集合的接口分开了，这种集合在与最基础的接口中：`kotlin.collections.Collection`
-  `kotlin.collections.MutableCollection` 表示可变集合，它继承于 `kotlin.collections.Collection`
- 使用集合时需要牢记一个关键点：只读集合不一定是不可变的，有可能该集合的真实类型是 Mutable 的，所有只读集合也不总是线程安全的
- 只读集合和可变结合是怎么做到分离的？Kotlin 中声明了 Kotlin集合 与 Java集合是一样的：每个 Kotlin 接口都对应 Java集合接口的一个实例，这总说法没有错，在 Kotlin 和 Java之间转移并不需要转换，不需要包装也不需要拷贝数据，但是每一种 Java 集合接口在 Kotlin 中都有两种表示：一种是只读的，一种是可变的。
- 对于 ArrayList 和 HastSet 等，在 Kotlin 看来，它们分别继承自 MutableList接口和 MutableSet接口
- Kotlin(至少是1.0版中)的 setOf() 和 mapOf() 返回的都是 Java 标准类库中的实例，在底层它们都是可变的，但是不能完全依赖于此，Kotlin 的未来版本可能改变这一行为
- 如果有一个使用 `java.util.Collection` 作为形参的方法，在 Kotlin 中，可以传递任意的 `kotlin.collections.Collection` 和 `kotlin.collections.MutableCollection`，这对集合的可变性有总要影响，因为 Java 并不能区分只读集合与可变集合，即使 Kotlin 中把集合声明成只读的，Java 代码也能够修改这个集合。因此如果 Kotlin 函数中使用了集合，并传递给了 Java，开发者必须有责任使用正确的参数类型，这取决于开发者调用的 Java 代码是否会修改集合
- 同样，向 Java 传递非空元素集合时，也无法阻止 Java 向其中添加 null 元素。开发者需要采取特别的措施

---
## 作为平台类型的集合

Kotlin 把那些定义在 Java 中的类型看作平台类型，Kotlin 没有任何关于平台类型的可空性信息，所以编译器允许 Kotlin 代码将其视为可空或者非空，通用 Java 中声明的集合类型变量也会被视为平台类型，一个平台类型的集合本质就是可变性未知的集合。Kotlin 将其视为只读的或者可变的。

通用到底设置为哪种类型，需要开发者自行斟酌。

通用 Java 类型 `List<String>` 如果表示成了两种不同的 Kotlin 类型，一种是 `List<String>?`，另一种是 `MutableList<String?>`，为了做出正确的选择，开发者必须知道 Java 接口或类必须遵守的确切契约。

---
## 对象和基本类型的数组

- Kotlin 的数组是一个带有类型参数的类
- arrayOf 函数创建一个数组，包含的元素是为该函数指定的实参
- arrayOfNulls 函数创建一个元素可空的数组
- 和其他类型一样，数组类型的类型参数始终会变成对象类型

