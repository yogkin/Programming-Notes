# Lambda 编程

---
## 1 高阶函数

- 函数式编程提供了另一种解决问题的方法：**把函数当作值来对待**，可以直接传递函数，而不需要先声明一个类在传递这个类的实例，使用 lambda 表达式后，代码会更加简介，都不需要声明函数了。
- 高阶函数：高阶函数是将函数用作参数或返回值的函数。

```kotlin
private fun <T> lock(lock: Lock, body: () -> T): T {
    lock.lock()
    try {
        return body.invoke()
    } finally {
        lock.unlock()
    }
}
```

---
## 2 函数类型

- 函数类型：对于接受另一个函数作为参数的函数，我们必须为该参数指定函数类型。
- 比如`(T, T) -> Boolean)`的类型为 接收两个T类型，返回一个Boolean
- 可以定义一个变量，指向一个函数类型，比如：` val toStr: (num: Int) -> String = {  it.toString() }`


---
## 2 Lambda表达式（LambdaExpression）

- `lambda` 表达式总是被大括号括着
- `lambda` 参数（如果有的话）在` -> `之前声明（参数类型可以省略），函数体（如果存在的话）在` ->` 后面。
- 如果 `lambda` 是该调用的是唯一参数，则调用中的圆括号可以完全省略。
- 如果函数的最后一个参数是一个函数，并且你传递一个`lambda` 表达式作为相应的参数，可以在圆括号之外指定它。
- 如果函数字面值只有一个参数， 那么它的声明可以省略（连同` ->`），其名称是 `it`。
- Kotlin 和 Java的一个重要的不同点是：Kotlin 没有严格限制在局部类中只能访问 final 变量。也可以从 lambda 中修改变量

代码示例：
```kotlin
    val list = listOf(1, 2, 3)
    list.map {
        {
            it.toString()
        }
    }.forEach {
        println(it)
    }
```

**在 Java 中使用函数类型**：其背后的原理是，函数类型被声明为普通的接口，一个函数类型的变量是 FunctionN 接口的一个实现，`kotlin.jvm.functions` 包中定义了一些列接口 `Function1->Function2->Function22` 用于实现 Kotlin 中的各种函数声明，目前最多到 Function22，就是说最多支持22个参数的函数类型，不过谁会声明一个超过 22 个参数的函数呢？

- Java 8 的 lambda 会被自动转换为函数类型的值。


---
##  4 函数复合

函数复合：组合多个函数成一个新的函数，形式为 `f(g(x))`。

```
//f(g(x))   m(x) = f(g(x))
val add5 = { i: Int -> i + 5 } // g(x)
val multiplyBy2 = { i: Int -> i * 2 } // f(x)

//定义扩展以支持组合函数，Function1->kotlin.jvm.functions.Function1，表示单参数函数
private infix fun <P1, P2, R> Function1<P1, P2>.andThen(function: Function1<P2, R>): Function1<P1, R> =
        fun(p1: P1): R = function.invoke(this.invoke(p1))

private infix fun <P1, P2, R> Function1<P2, R>.compose(function: Function1<P1, P2>): Function1<P1, R> =
        fun(p1: P1): R = this.invoke(function.invoke(p1))

//测试
fun main(args: Array<String>) {
    println(multiplyBy2(add5(8))) // (5 + 8) * 2

    val add5AndMultiplyBy2 = add5 andThen multiplyBy2
    val add5ComposeMultiplyBy2 = add5 compose multiplyBy2
    println(add5AndMultiplyBy2(8)) // m(x) = f(g(x))
    println(add5ComposeMultiplyBy2(8)) // m(x) = g(f(x))
}

```

---
## 5  匿名函数

- 一般`lambda` 表达式语法缺少指定函数的返回类型的能力。在大多数情况下，这是不必要的。因为返回类型可以自动推断出来。然而，如果确实需要显式指定，可以使用另一种语法：`匿名函数`
- 匿名函数的返回类型推断机制与正常函数一样：对于具有表达式函数体的匿名函数将自动推断返回类型，而具有代码块函数体的返回类型必须显式 指定（或者已假定为 Unit）。
- 匿名函数参数总是在括号内传递。 允许将函数留在圆括号外的简写语法仅适用于 `lambda `表达式。
- `Lambda`表达式和匿名函数之间的另一个区别是非局部返回的行为。一个不带标签的` return` 语句 总是在用` fun `关键字声明的函数中返回。这意味着` lambda `表达式中的` return `将从包含它的函数返回，而匿名函数中的 `return` 将从匿名函数自身返回。

---
## 6 自执行闭包

自执行闭包就是在定义闭包的同时直接执行闭包。
```
{
        println("hh")
}()

//不推荐使用直接执行闭包，取而代之的应该是 run
run{
  println("hh")
}
```

---
## 7 带接收者的函数字面值

Kotlin 提供了一种能力，调用一个函数字面值时，可以指定一个 **接收者对象(receiver object)**，在这个函数字面值的函数体内部，你可以调用接收者对象的方法，而不必指定任何限定符。这种能力与扩展函数很类似，在扩展函数的函数体中，你也可以访问接收者对象的成员。

这个特性对 Java 不可用：调用 lambda 内部的不同对象的方法，而无需额外的修饰符的能力。这样的 lambda 叫做带有接收器的 `lambda(lambdas with receivers)`，Kotlin标准库中的 with 和 apply 函数就是这样的函数：

```kotlin
     //当接收者类型可以从上下文推断时，lambda 表达式可以用作带接收者的函数字面值。
    class HTML {
        fun body() {

        }
    }

    //接收者是HTML
    fun html(init: HTML.() -> Unit): HTML {
        val html = HTML()  // 创建接收者对象
        html.init()        // 将该接收者对象传给该 lambda
        return html
    }

    //此时可以推断出接收者是HTML
    html {
        // 带接收者的 lambda 由此开始
        body()// 省略HTML直接调用该接收者对象的一个方法
    }
```


---
## 8 科里化

把一个多元函数变成多个单元函数调用链的过程

```kotlin
private fun log(tag: String, target: OutputStream, message: Any?) {
    target.write("[$tag] $message\n".toByteArray())
}

//柯里化函数：精简模式
private fun logA(tag: String) = fun(target: OutputStream) = fun(message: Any?) = target.write("[$tag] $message\n".toByteArray())

//柯里化函数：精简模式原理
private fun logA(tag: String): (target: OutputStream) -> (message: Any?) -> Unit {
    return fun(target: OutputStream): (message: Any?) -> Unit {
        return fun(message: Any?) {
            target.write("[$tag] $message\n".toByteArray())
        }
    }
}

//通用型：Function3表示任意该类型的函数
private fun <P1, P2, P3, R> Function3<P1, P2, P3, R>.curried() = fun(p1: P1) = fun(p2: P2) = fun(p3: P3) = this(p1, p2, p3)

//调用过程：
    log("benny", System.out, "HelloWorld")//调用单个函数
    logA("benny")(System.out)("HelloWorld Again.")//形成调用链
    val consoleLogWithTag = (::log.curried())("benny")(System.out)
    consoleLogWithTag("HelloAgain Again.")
```

---
## 9 偏函数

指定多参数函数的某些参数而得到新函数，实现方式是函数扩展

```kotlin
private fun <P1, P2, R> Function2<P1, P2, R>.partial2(p2: P2) = fun(p1: P1) = this(p1, p2)
private fun <P1, P2, R> Function2<P1, P2, R>.partial1(p1: P1) = fun(p2: P2) = this(p1, p2)

private val makeString = fun(byteArray: ByteArray, charset: Charset): String {
    return String(byteArray, charset)
}

private val makeStringFromGbkBytes = makeString.partial2(charset("GBK"))

//使用偏函数
    val bytes = "我是中国人".toByteArray(charset("GBK"))
    val stringFromGBK = makeStringFromGbkBytes(bytes)
```

---
## 10 集合的函数式 API

Kotlin 提供了大量的函数式编程API，filter，map，all, any, count，find，groupBy，flatMap，flatten 等

```kotlin
    File("build.gradle")
            .readText()
            .toCharArray()
            .filterNot(Char::isWhitespace)
            .groupBy { it }
            .map {
                it.key to it.value.size
            }
            .forEach(::println)
```

- 注意有时候把 filter 放在流式操作的前面可以减少对象的转换，因为已经过滤了一遍。

---
## 11 惰性求值：序列(asSequence)

类似 map 和 filter 这些函数会立即创建临时集合。这意味着每一步的临时结果都被保存在一个临时列表中。序列给了你一个可选的方法来完成这样的计算。这样可以避免创建一次性的临时对象。

集合懒操作的入口是 Sequence 接口。这个接口仅表示：一系列可以逐个枚举的元素。Sequence 只提供了一个方法:`iterator`。你可以用它来从序列中获取元素值。

序列操作分为两类：`中间操作和最终操作`。中间操作返回另一个序列。这个序列知道如何变换原始序列的集合。最终操作返回一个结果。这个结果可以是集合、集合元素、数字或者任意其他通过初始集合变换得到的序列

```kotlin
people.map(Person::name).filter { it.startsWith("A") }

people.asSequence()// 1 把初始集合转换为序列
.map(Person::name)// 2 序列支持跟集合同样的API
.filter { it.startsWith("A") } //
.toList()// 3 把结果序列转换为列表
```

### 创建序列

对集合调用 `asSequence()`用于创建一个序列。另一个方式是使用 `generateSequence()` 函数

```kotlin
private fun stringSquence(args: Array<String>) {
    val naturalNumbers = generateSequence(0) { it + 1 }
    val numbersTo100 = naturalNumbers.takeWhile { it <= 100 }
    println(numbersTo100.sum())
}
```

另一个常见的情形是序列的父类。如果一个元素拥有它的父类（比如人类或者Java文件），也许你会对许多序列的所有祖先感兴趣：

```kotlin
fun File.isInsideHiddenDirectory() = generateSequence(this) {
    println("generate：$it")
    it.parentFile
}.any {
    println("any：$it")
    it.isHidden
}

fun main(args: Array<String>) {
    val file = File(".").absoluteFile
    println(file.isInsideHiddenDirectory())
}

打印结果：

any：E:\code\studio\my_github\Repository\Kotlin\HelloKotlin\.
generate：E:\code\studio\my_github\Repository\Kotlin\HelloKotlin\.
any：E:\code\studio\my_github\Repository\Kotlin\HelloKotlin
generate：E:\code\studio\my_github\Repository\Kotlin\HelloKotlin
any：E:\code\studio\my_github\Repository\Kotlin
generate：E:\code\studio\my_github\Repository\Kotlin
any：E:\code\studio\my_github\Repository
generate：E:\code\studio\my_github\Repository
any：E:\code\studio\my_github
generate：E:\code\studio\my_github
any：E:\code\studio
generate：E:\code\studio
any：E:\code
generate：E:\code
any：E:\
true
```

---
## 12 中缀表示法

在满足以下条件时,函数也可以通过中缀符号进行调用：它们是成员函数或者是扩展函数且只有一个参数，使用infix关键词进行标记。

具体条件：

- 1，成员函数或扩展函数
- 2，只有一个参数
- 3，用`infix` 关键字标注

应用场景： DSL

```kotlin
private infix fun Int.pow(x: Int): Int {
    return Math.pow(this.toDouble(), x.toDouble()).toInt()
}

private fun testInfix() {
    val a = 3
    var b = 3.pow(5)
    //顾名思义，中缀表示法，pow以中缀的形式放在调用对象与参数之间，没有函数调用的形式。
    var c = 2 pow 3

    //这里step也是中缀表示法，而..是区间操作符
    for (i in 1..69 step 3) {

    }
}
```

---
## 13 成员引用

- 使用`::`操作符可以引用成员函数，这种表达式称之为成员引用
- 构造方法也是可以引用的：`::String`
- 绑定引用：Kotlin 1.1 支持绑定引用

```
val p = Persion("A",12)
//kotlin 1.0
val personAgeFunction = Persion::age
personAgeFunction(p)

//kotlin 1.1
val personAgeFunction = p::age//引用直接绑定了 p 对象
personAgeFunction()
```

---
## 14 使用 Java 的函数式接口

Kotlin 的 lambda 可以完全和 Java API 进行互操作。

```Java
/* Java */
public class Button{
    public void setOnClickListener(OnClickListener l){ ... }
}
public interface OnClickListener{
    voidonClick(View v);
}
```

Java 8之前的版本中，你必须创建一个匿名类的实例并把它当做一个参数传递，在Kotlin中，你可以传递lambda

```kotlin
button.setOnClickListener { view -> ... }
```

这是有效的，因为 OnClickListener 接口只有一个抽象方法。这样的接口叫做函数式接口（functional interfaces），或者叫 `SAM接口`。其中 SAM 表示单一抽象方法(single abstract method)。Java API 导出都是像 Runnable 和 Callable 这样的函数式接口，以及接口中的方法。Kotlin 允许你在调用接收函数式接口作为参数的 Java 方法时使用 lambda。这确保你的 Kotlin 代码保持整洁和地道。

### 把 lambda 当做参数传递给 Java 方法

可以把 lambda 传递给任何一个需要一个函数接口的 Java 方法

```kotlin
/* Java */void postponeComputation(int delay, Runnable computation);
```

在Kotlin中，你可以调用它并向它传递一个作为参数的lambda。编译器将会自动把它转换为一个Runnable的实例：

```kotlin
//方式 1，为整个程序创建一个`Runnable`实例
postponeComputation(1000) { println(42) }

//方式 2，当显式声明一个对象时，每次调用都会创建一个新的实例
postponeComputation(1000, object : Runnable
    { overridefunrun() {
        println(42)
    }
})
```

上面两种情况的处理方式不同：

- 当显式声明一个对象时，每次调用都会创建一个新的实例
- 使用lambda，情况就不一样了：如果lambda没有访问定义它的函数的变量，对应的匿名类实例将会在调用之间进行重用

### lambda 和添加/移除侦听器

意，由于处在匿名对象内部，lambda 中没有 this关键字：无法在被转换的 lambbda 中引用匿名类实例。从编译器的角度来看，lambda 是一块代码，不是一个对象。你不能把它引用为一个对象。lambda 中的 this 引用指向包围它的类。 如果你的事件侦听器在处理事件时，需要将自身取消订阅，你无法为此使用 lambda。相反的，你应该使用一个匿名对象来实现侦听器。在匿名对象中，this 关键字指向该对象的实例。你可以把它传递给移除侦听器的 API


---
## 15 内联函数：inline、noinline、rossinline、reified

### 使用内联函数提升性能

- 使用高阶函数会带来一些运行时的效率损失：每一个函数都是一个对象，并且会捕获一个闭包。
- `inline` 修饰符影响函数本身和传给它的 `lambda` 表达式：所有这些都将内联 到调用处。
- 内联可能导致生成的代码增加，但是如果我们使用得当（不内联大函数），它将在 性能上有所提升，尤其是在循环中的“超多态（megamorphic）”调用处。
- 编译器完全支持内联跨模块的函数或者是第三方库定义的函数，也可以在Java中调用绝大部分内联函数，但Java中这些函数不会被内联，而是被便以为普通的函数
- 内联的集合操作：Kotlin 中集合操作函数，比如 filter 等都是内联的，可以大胆的使用它们，而不需要关心性能问题

在调用内联函数的时候可以传递`函数类型的变量`作为参数，在这种情况下 lamdba 的代码无法被内联，因为 lamdba 的代码在内联函数被调用点是不可用的。只有 synchronized 的函数体被内联了，lamdba 才会被正常调用：

```kotlin
inline fun<T> synchronized(lock:Lock, action:()->T){
    ...
}

class LockOwner(val lock:Lock){
    fun runUnderLock(body:()->Unit){
        //这里 body 无法被内联
        synchronized(lock, body)
    }
}
```

### 内联函数的限制

鉴于内联函数的运作方式，不是所有使用 lambda 的函数都可以被内内联，当函数被内联的时候，作为参数的 lambda 表达式的函数体会直接替换为最终生成的代码中，这将函数体中的对应（lambda）参数的作用，如果（lambda） 被调用，这样的参数很容易地被内联，但如果（lambda）参数在某个地方被保存起来，以便后面可以继续调用，lambda 表达式的代码将不能被内联，因为必须要有一个包含这些代码的对象存在。

一般来说，参数如果被直接调用或者作为参数传递给另一个内联的函数，它是可以被内联的，否则编译器会禁止参数被内联并给出错误信息：`Illegal usage of inline-parameter`。

一个很好的例子：

```
fun <T,R> Sequence<T>.map(transform:(T)-> R):Sequence<R>{
    //transform 没有被直接调用，而是作为构造函数的参数传递给了TransformingSequence
    return TransformingSequence(this,transform)
}
```

### 禁用函数参数的内联

1. 如果传给一个内联函数的参数是 lambda 表达式，而且想要控制某些 lambda 参数不被内联，可以用 noinline 修饰符标记哪些不想被内联的函数
2. **noinline的优势**：可以内联的 lambda 表达式只能在内联函数内部调用或者作为可内联的参数传递， 但是 noinline 的可以以任何我们喜欢的方式操作：`存储在字段中`、`传送它`等等。
3. 需要注意的是，`如果一个内联函数没有可内联的函数参数并且没有具体化的类型参数`，编译器会产生一个警告，因为内联这样的函数很可能并无益处（如果你确认需要内联，则可以用 @Suppress("NOTHING_TO_INLINE") 注解关掉该警告）。

```kotlin
private class NoinlineSub {
    var func: (() -> Unit)? = null
}

private inline fun noinlineSample(inlined: () -> Unit, /*使用noinline修饰*/noinline notInlined: () -> Unit) {
    val a = NoinlineSub()
    //a.func = inlined //由于inlined是内联lambda表达式，所以不能赋值给NoinlineSub的func字段。
    a.func = notInlined
    inlined.invoke()
    notInlined.invoke()
}
```

### 非局部返回

- 非局部返回：在 Kotlin 中，我们可以只使用一个正常的、非限定的` return` 来退出一个命名或匿名函数。 这意味着要退出一个 `lambda` 表达式，我们必须使用一个标签，并且 在 lambda 表达式内部禁止使用裸 `return`，因为 `lambda` 表达式不能使包含它的函数返回
- 但是如果 lambda 表达式传给的函数是内联的，该 return 也可以内联，所以它是允许的，这种返回（位于 lambda 表达式中，但退出包含它的函数）称为非局部返回
- 匿名函数默认使用局部返回，总结：fun 就近原则

```kotlin
private fun inlineReturn1() {

    noinlineLock(ReentrantLock()) {
        //禁止使用裸 return，因为 lambda 表达式不能使包含它的函数返回：
        // 错误：不能使 `foo` 在此处返回
        //return
    }

    //内联函数可以直接从函数返回，因为运行时它将会被内联到该函数体内
    lock(ReentrantLock()) {
        return
    }
}

private fun inlineReturn2() {
    //但是如果 lambda 表达式传给的函数是内联的，该 return 也可以内联，所以它是允许的
    //这种返回（位于 lambda 表达式中，但退出包含它的函数）称为非局部返回
    fun hasZeros(ints: List<Int>): Boolean {
        //forEach是是一个内联扩展
        ints.forEach {
            if (it == 0) return true // 从hasZeros 返回
        }
        return false
    }
}

//名函数默认使用局部返回，fun 就近原则
fun lookForAlice(people: List<Person>) {
    people.forEach(fun (person) {
        if (person.name == "Alice") return
        println("${person.name} is not Alice")
    })
}
```

### 内联属性

inline 修饰符可用于没有幕后字段的属性的访问器。 可以标注独立的属性访问器，在调用处，内联访问器如同内联函数一样内联。

```kotlin
private class Pa {}
private class Pb {}

private fun getPa(): Pa {
    return Pa()
}

private class InlinePropertySub {

    val foo: Pa
        inline get() = getPa()
}
```

### 自另一个执行上下文的 lambda 表达式参：rossinline

一些内联函数可能调用这种函数——`传给它们的不是直接来自函数体、而是来自另一个执行上下文的 lambda 表达式参数`。例如来自局部对象或嵌套函数。在这种情况下，该 lambda 表达式中也不允许非局部控制流。 为了标识这种情况，该 lambda 表达式参数需要 用 `crossinline` 修饰符标记

```kotlin
private inline fun crossinlineSample(crossinline body: () -> Unit) {
    val f = object : Runnable {
        //body的执行上下文是Runnable
        override fun run() = body()
    }
}
```


### 具体化的类型参数：reified

有时候我们需要**访问一个作为参数传给我们的一个类型**，这时可以使用具体化的类型参数：`reified`，使用 reified 修饰代码类型参数的函数，在函数体内，可以像使用实际类型那样使用类型参数声明，因为它将会被内联。

```kotlin
//在这里我们向上遍历一棵树并且检查每个节点是不是特定的类型。 这都没有问题，但是调用处不是很优雅
private fun <T> TreeNode.findParentOfType1(clazz: Class<T>): T? {
    var p = parent
    while (p != null && !clazz.isInstance(p)) {
        p = p.parent
    }
    @Suppress("UNCHECKED_CAST")
    return p as T?
}

//内联函数支持具体化的类型参数，于是我们可以这样写：
//使用 reified 修饰符来限定类型参数，现在可以在函数内部访问它了， 几乎就像是一个普通的类一样。
//由于函数是内联的，不需要反射，正常的操作符如 !is 和 as 现在都能用了
//普通的函数（未标记为内联函数的）不能有具体化参数。 不具有运行时表示的类型
private inline fun <reified T> TreeNode.findParentOfType2(): T? {
    var p = parent
    while (p != null && p !is T) {
        p = p.parent
    }
    return p as T?
}

private fun testfindParentOfType() {
    val treeNode: TreeNode? = null
    treeNode?.findParentOfType1(DefaultMutableTreeNode::class.java)
    treeNode?.findParentOfType2<TreeNode>()
}
```

### 何时使用函数声明内联

- 不要随意的使用 inline， **使用 inline 只能提高带有 lambda 参数的函数的性能**，其他情况需要额外的度量和研究，对于普通函数调用，JVM 已经提供了强大的内联支持，它会分析代码的执行，并在任何时候通过内联通过带来好处的时候将函数调用内联，这是在将字节码转换为机器码时自动完成的。
- 如果一个函数的代码很长，不应该使用 inline，这样会过度增加字节码的长度。