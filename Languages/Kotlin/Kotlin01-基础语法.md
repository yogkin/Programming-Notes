# 1 数据类型与语句

---
## 1 数据类型与变量定义

### 定义变量

- 可变使用 `var`（variable） 和 常量使用 `val`（value）
- val 与 Java 中的 final 对应
- 成员变量必须显式指定初始值，否则需要使用 `lateinit` 修饰变量，lateinit 不能用于基本类型和可空类型
- const：在 companion 对象或顶级作用域中可以使用 `const val` 定义字面常量，支持基本类型和String
- 编译器常量和运行时常量：val 是运行期常量，const val 是编译器常量

一个 val 变量必须在定义块执行时被初始化而且只能一次。但是如果编译器能够确保唯一的初始化声明能够其中一个被执行，你可以根据情况用不同的值初始化变量：

```kotlin
//这样初始化是合法的
val message: String
if (condition()) {
    message = "Success"
} else {
    message = "Failed"
}
```

### 数字类型

- 支持的数字数据类型
- 不同数字类型不能隐式转换，使用`toXXX`方法进行转换，比如`toInt()`
- 字面量支持二进制、十进制、十六进制，不支持八进制
- 2进制以 `0b `开头：`0b00001011`
- 数字类型位操作：`shl、shr、ushr、and、or、xor、inv`
- 当需要可空引用时，像数字、字符会被装箱。装箱操作不会保留同一性
- 允许除浮点0 ：`3 / 0.0 = Float.NaN`
- Kotlin 中不支持隐式类型转换

### 字符类型

- **它们不能直接当作数字**，字符字面值用单引号括起来: `'1'`，特殊字符可以用反斜杠转义，
- 编码其他字符要用 Unicode 转义序列语法：`'\uFF00'`。

### 字符串

- `"""`括起来的字符串会保留原来的换行格式，其中的特殊字符不需要转义
- 字符串模板的使用方法：使用`${xxx}`引用其他变量
- 字符串提供了很多扩展方法：`substringBeforeLast(); substringAfterLast(); trimMargin()`
- Kotlin 中 String 的 split 方法被改写了，它提供了同名方法的多个重载版本

```kotlin
    val list1 = "12.345-6.A".split("\\.".toRegex())
    println(list1)
    val list2 = "12.345-6.A".split(".","-")/指定多个分隔符
    println(list2)
结果:
[12, 345-6, A]
[12, 345, 6, A]
```

### 数组类型

- 数组在 Kotlin 中使用 Array 类来表示， 它定义了 get （按照运算符重载约定这会转变为 `[]`）和 set 函数 和 size 属性，以及一些其他有用的成员函数
- 与 Java 不同的是，Kotlin 中数组是**不型变**的（invariant）。这意味着 Kotlin 不让我们把 `Array<String>` 赋值给 `Array<Any>`
- Kotlin 也有无装箱开销的专门的类来表示原生类型数组: `ByteArray、 ShortArray、IntArray` 等等

### Range 类型

- **区间本质上就是两个值之间的间隔**，如果能迭代出区间中所有的值，那么这样的区间就称为数列
- `..`操作符表示区间表达式，重载的是`rangeTo`函数
- Range是一种数据结构，表示一个范围：比如 1 到5用 `1..5` 表示
- 使用 `in 和 !in` 运算符来检测某个数字是否在指定区间内
- `..`不仅可以创建数字区间，还可以创建字符区间，比如`'A'.. 'F'`
- 区间表达式由具有操作符形式 .. 的 `rangeTo` 函数辅以` in 和 !in` 形成。
- 区间是为任何可比较类型定义的，但对于整型原生类型，它有一个优化的实现。
- 整型区间（`IntRange、 LongRange、 CharRange`）有一个额外的特性：它们可以迭代。 编译器负责将其转换为类似 Java 的基于索引的 for-循环而无额外开销。
- 如果需要倒序迭代数字也很简单。可以使用标准库中定义的 `downTo()` 函数
- 以不等于 1 的任意步长迭代数字？ 当然没问题， `step()` 函数有助于此
- 要创建一个不包括其结束元素的区间，可以使用 `until` 函数
- `rangeTo()`：整型类型的 rangeTo() 操作符只是调用 *Range 类的构造函数
- `downTo()`：为任何整型类型对定义的
- `reversed()`：为每个 `*Progression` 类定义的，并且所有这些函数返回反转后的数列
- `step()`：扩展函数 step() 是为每个 `*Progression` 类定义的， 所有这些函数都返回带有修改了 `step` 值（函数参数）的数列。
- 区间如何工作：区间实现了该库中的一个公共接口：`kotlin.range.ClosedRange<T>`。

```kotlin
fun range(){
    for(i in 1..10 step 2){
        println(i)
    }
     for(i in 1.rangeTo(10) step 2){
            println(i)
     }
}
```

---
## 2 语句

### 流程控制

- `if`：if表达式取代了三元运算符，kotlin中没有三元运算符
- `for` 循环
- `while` 循环
- `when`：kotlin没有 `switch`，而 when 是增强版的 switch
- `when` 结构中可以使用任意对象，检查的条件是内容是否相等
- `when` 使用时可以不带参数的
- 使用 when 处理枚举或密封类时，如果全部情况都作为了分支，则可以省略最后的 else 分支
- 从流程中返回：定义标签`loop1@`，标签返回 `returnloop1@`

### 表达式

- 类似 `if、when、try` 之类在 Kotlin 中都是有返回值的表达式，而不是语句。
- 而 Java 中的赋值操作表达式，在Kotlin中变成了语句，比如`return a = getInt()` 在 Kotlin 中是非法语句，因为赋值操作没有返回值。

### 类型检查与转换

- 类型检查用`is 或 !is`，比如 `obj is String`
- 智能类型转换：当且仅当一个变量在 is 检查之后不再改变时，智能类型转换才会起作用。当你对一个带有属性的类使用智能类型转换时，属性必须是一个val（不可变类型），同时它不能有自定义的访问器。否则，它不能验证每一个属性的访问是否会返回同样的值。
- 类型转换：一个指定类型的显式转换通过 `as` 关键词来表达的 `val n = e as Num`，`as?`用于进行安全的转换

### 使用 in 检查

- 可以使用 `in` 操作符来检查一个值是否在某个范围内，或者相反的，`!in`（操作符）来检查一个值是否不再某个范围内

### 空检查与 Elvis 操作符

- 当某个变量的值可以为 null 的时候，必须在声明处的类型后添加 ? 来标识该引用可为空。
- `Elvis操作符?:`：示例`obj?.getSome() ?: doSomething()`，`?:`符号会在符号左边为空的情况才会进行下面的处理，不为空则不会有任何操作。

```
var s1 : String? = null
var s2 : String = ""
```

### 对象比较

```
       ==比较：
           如果作用于基本数据类型的变量，则直接比较其存储的 “值”是否相等
           对于引用类型： a==b等同于a?.equals(b) ?: (b === null)
       equals
           equals方法不能作用于基本数据类型的变量
           如果没有对equals方法进行重写，则比较的是引用类型的变量所指向的对象的地址；
           诸如String、Date等类对equals方法进行了重写，比较的是所指向的对象的内容。
        === 和 !==
            a===b当且仅当a和b指向同一个对象时返回true
```

### 使用包级函数

- 包就是命名空间
- 与 Java 或 C# 不同，**在 Kotlin 中类没有静态方法**。在大多数情况下建议简单地使用包级函数。
- kotlin 没有对磁盘上源文件的布局强加任何限制，可以不按照包名来布局源文件

### try/catch/finally

- Kotlin 没有区分已检查和未检查的异常。所有异常都是不需要检查的

### 空安全：可空类型与非空类型

Kotlin 的类型系统旨在消除来自代码空引用的危险。许多编程语言（包括 Java）中最常见的陷阱之一是访问空引用的成员，导致空引用异常。在 Java 中，这等同于 `NullPointerException` 或简称 NPE。Kotlin 的类型系统旨在从我们的代码中消除 NullPointerException

NPE 的唯一可能的原因可能是：

1. 显式调用 `throw NullPointerException()`
2. 使用了 `!!` 操作符
3. 外部 Java 代码导致的
4. 对于初始化，有一些数据不一致（如一个未初始化的 this 用于构造函数的某个地方）


### 类别名

类型别名为现有类型提供替代名称。如果类型名称太长，你可以另外引入较短的名称，并使用新的名称替代原类型名。类型别名不会引入新类型。 它们等效于相应的底层类型。 当你在代码中添加 `typealias Predicate<T>` 并使用 `Predicate<Int>` 时，Kotlin 编译器总是把它扩展为 `(Int) -> Boolean`。 因此，当你需要泛型函数类型时，你可以传递该类型的变量，反之亦然

```kotlin
            typealias NodeSet = Set<Network.Node>
```

### this 表达式

- 在类的成员中，`this` 指的是该类的当前对象
- 在扩展函数或者带接收者的函数字面值中， `this `表示在点左侧传递的接收者参数。
- 如果` this` 没有限定符，它指的是最内层的包含它的作用域。要引用其他作用域中的 `this`，请使用 标签限定符
- 要访问来自外部作用域的`this`（一个类 或者扩展函数， 或者带标签的带接收者的函数字面值）我们使用`this@label`，其中 `@label` 是一个 代指` this` 来源的标签

```kotlin
private class A { // 隐式标签 @A

    inner class B { // 隐式标签 @B

        fun Int.foo() { // 隐式标签 @foo

            val a = this@A // A 的 this
            val b = this@B // B 的 this
            val c = this // foo() 的接收者，一个 Int
            val c1 = this@foo // foo() 的接收者，一个 Int

            val funLit = lambda@ fun String.() {
                val d = this // funLit 的接收者
            }

            val funLit2 = { s: String ->
                // foo() 的接收者，因为它包含的 lambda 表达式
                // 没有任何接收者
                val d1 = this
            }

        }
    }
}
```


---
## 3 Kotlin默认导入的包

有多个包会默认导入到每个 Kotlin 文件中：

```
kotlin.*
kotlin.annotation.*
kotlin.collections.*
kotlin.comparisons.*
kotlin.io.*
kotlin.ranges.*
kotlin.sequences.*
kotlin.text.*
```
