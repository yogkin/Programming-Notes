#  与java互操作

Kotlin与java的交互： Kotlin 在设计时就考虑了Java 互操作性。
可以从 Kotlin 中自然地调用现存的 Java 代码，并且在 Java 代码中也可以很顺利地调用 Kotlin 代码。

---
## 1 Kotlin中调用Java

- 遵循 Java 约定的 getter 和 setter 的方法（名称以 get 开头的无参数方法和以 set 开头的单参数方法）在 Kotlin 中表示为属性。
- 如果 Java 类只有一个 setter，它在 Kotlin 中不会作为属性可见，因为 Kotlin 目前不支持只写（set-only）属性。
- 如果一个 Java 方法返回 void，那么从 Kotlin 调用时中返回 Unit。 万一有人使用其返回值，它将由 Kotlin 编译器在调用处赋值， 因为该值本身是预先知道的（是 Unit）
- 一些 Kotlin 关键字在 Java 中是有效标识符：in、 object、 is 等等。 如果一个 Java 库使用了 Kotlin 关键字作为方法，你仍然可以通过反引号（`）字符转义它来调用该方法
- 空安全和平台类型：Java 中的任何引用都可能是 null，这使得 Kotlin 对来自 Java 的对象要求严格空安全是不现实的。 Java 声明的类型在 Kotlin 中会被特别对待并称为平台类型。对这种类型的空检查会放宽， 因此它们的安全保证与在 Java 中相同
- 平台类型表示法：T! 表示“T 或者 T?”
- 已映射类型
- Kotlin 中的 Java 泛型
- Java 数组：与 Java 不同，Kotlin 中的数组是不型变的。这意味着 Kotlin 不允许我们把一个 `Array<String>` 赋值给一个 `Array<Any>`， 从而避免了可能的运行时故障。
- 对于每种原生类型的数组都有一个特化的类（IntArray、 DoubleArray、 CharArray 等等）来处理这种情况。 它们与 Array 类无关，并且会编译成 Java 原生类型数组以获得最佳性能。
- Java可变参数

---
## 2 Java中调用Kotlin

### null字段

 - Nullable 注解
 - NotNull 注解

### 几个常用注解

 - `@JvmField`属性便以为java字段
 - `@JvmStatic`对象方法便以为静态方法
 - `@JvmName `解决签名冲突
 - `@JvmOverloads `默认参数方法编译为多个重载方法

### 包级方法

 - 包：在 `org.foo.bar` 包内的` example.kt `文件中声明的所有的函数和属性，包括扩展函数， 都编译成一个名为 `org.foo.bar.ExampleKt `的 Java 类的静态方法

### NoArg和AllOpen插件

 - NoArg为data class生成无参构造方法
 - AllOpen为标注的类去掉final，允许被继承

### SAM(single abstract method)转换

- SAM转换条件：java单一方法接口
- 注意转换之后的实例变化：调用Java方法时，如果传入一个lambda，lambda会被转换为Runnable类型，每一次调用都会有一次新的转换，这种变换即SAM
