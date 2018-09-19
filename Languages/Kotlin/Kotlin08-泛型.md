# 泛型

---
## 1 泛型类型参数

Kotlin 始终要求类型实参要么被显示的说明，要么能够被编译器推断出来。

### 类型参数约束

类型参数约束可以限制作为泛型类和泛型函数的类型实参的类型，kotlin 中上界约束声明为：`fun<T:Numner> List<T>sum():T`，`:`代替了Java中的 extends 关键字。

一旦指定了类型 T 的上界，就可以把类型 T 的值当作上界的值使用，比如调用定义在上界类中的方法

```kotlin
/*泛型上限*/
private fun <T : Number> oneHalf(value: T): Double {
    //类型为 T 的value 可以使用 Number 上的方法
    return value.toDouble() / 2.0
}

/*泛型上限*/
private fun <T : Comparable<T>> max(first: T, second: T): T {
    return if (first > second) first else second
}

println(oneHalf(3))
println(max("kotlin", "java"))
```

极少数的情况下需要在一个类型参数上指定多个约束，下面代码限制了 T 同时是 CharSequence 和 Appendable的 子类，这是需要使用 `where` 关键字，参考下面代码：

```kotlin
fun <T> ensureTrailingPeriod(seq: T) where T : CharSequence, T : Appendable {
    if (!seq.endsWith('.')) {
        seq.append('.')
    }
}

fun main(args: Array<String>) {
    val helloWorld = StringBuilder("Hello World")
    ensureTrailingPeriod(helloWorld)
    println(helloWorld)
}
```

### 让类型形参非空

没有指定泛型上界的类型形参将会被认为是 `Any?`的，如果想要包装类型形参始终是非空类型，可以通过指定一个约束来实现。

```kotlin
//此时的 T 是可空的类型，在使用时需要进行判断
private class Producer<out T>

//此时的 T 是非空的类型
private class Producer<out T: Any>
```

---
## 2 运行时泛型：擦除和实例类型参数

JVM 上的泛型一般是通过泛型擦出类实现的，也就是说在运行时泛型类的类型实例是不被保留的，这样做是有好处的，可以减少应用程序使用的内存，因为不需要保存类型参数

### 运行时的泛型：类型检查和转换

一般而言，在 is 检查中不可能使用类型实参中的类型，下面代码无法编译：

```kotlin
//无法编译
if(value is List<String>){
    ...
}
//可以使用星投影来做这种检查，List<*>类比与Java中的List<?>
if(value is List<*>){
    ...
}
```

as 和 as? 转换中仍然可以使用一般的泛型类型，但是如果该类有正确的类型但是类型实参是错误的，转换也不会失败，因为在运行时转换发生的时候实参是未知的。

```kotlin
fun printSum(c: Collection<*>) {
    //如果 c 是 List<String> 类型，转换也可以正常工作，但是sum函数会导致 ClassCastException
    val intList = c as? List<Int> ?: throw IllegalArgumentException("List is expected")
    println(intList.sum())
}

fun main(args: Array<String>) {
    printSum(listOf(1, 2, 3))
}
```

### 声明带实化类型参数的函数

内联函数的实参可以被实化，意味着你可以在运行时引用实际的类型实参

```kotlin
inline fun <reified T> isA(value: Any) = value is T

fun main(args: Array<String>) {
    println(isA<String>("abc"))
    println(isA<String>(123))
}
```

为什么在 inline 函数中允许这样写 `e is T`，而普通的类或函数却不行，编译器把实现内联函数的字节码插入每一次调用发生的地方。

注意:

- inline 函数可以在 Java 中使用，但不会被内联
- 而 reified 类型的参数的 inline 函数不能在 Java 中调用
- 我们提供的内联只有在一种情况下有性能优势：即函数有函数类型的参数，且对于的实参 lambda 和函数一起被内联的时候
- 实化类型参数的函数的应用：代替类引用 

### 实化类型参数的限制

可以做的事：

- 用在类型检查和类型转换中：`is !is as as?`
- 使用 Kotlin 反射 API
- 获取相应的 java.lang.Class，`::class.java`
- 作为调用其他函数的类型实参

不能做的事：

- 创建指定为类型参数的类的实例
- 调用类型参数类的伴生对象的方法
- 调用带实化类型参数函数的时候使用非实化类型形参作为类型实参
- 把类、属性或者非内联函数的类型参数标记成 reified

---
## 3 变型：泛型和子类泛型

变型的概念描述了拥有相同基础类型和不同类型实参的(泛型)之间是如何关联的，例如`List<String> 和 List<Any>`之间如何关联。

### 为什么存在变型：给函数传递实参

如果一个函数接收 Any 类型的参数，那么把 String 传递给这个函数是绝对安全的，但是当 String 和 Any 成为 List 接口的类型实参之后，就没这么简单了，参考下面代码：

```kotlin
//示例1：
fun printCOntents(list:List<Any>){
    println(list.joinToString())
}

//在Kotlin中，这是安全的
println(listof("ABC","BAC"))

//示例2：
fun addAnswer(list:MutableList<Any>){
   list.add(42)
}

val strings = mutableList("abc","bac")
addAnswer(strings)//如果这行编译通过
println(strings.maxBy{ it.length })//这行抛出 ClassCastException：无法将整数转换为字符串
```

与 Java 一样，默认情况下泛型是不型变的，即`MutableList<String>` 不是 `MutableList<Any>`的子类型，为什么？因为这将导致列表中的元素类型不安全。但是在 Kotlin 中，上面代码因为 `List<Any>`是只读的列表，不能向列表中添加任何类型的元素，因此示例1 中这种传参是安全的，可以通过根据列表是否可变选择合适的接口来轻易地控制，如果函数接收的是只读列表，可以传递具有更具体的元素类型的列表。

---
### 类、类型、子类型

有时候我们会把`类型`和`类`当成相同的概念使用，但它们不一样。最简单的例子就是非泛型类，类的名称可以直接当作类型使用，例如`var x: String`就是声明了一个可以保存 String 类的实例的变量，但是注意，同样的类名也可以用来声明可空类型：`var x:String?`，这意味着每个 Kotlin 类都可以用于构造至少两种类型，泛型类的情况就更复杂了。

**类型术语**：

- 子类型：为了讨论类型之间的关系，需要熟悉子类型这个术语，任何时候如果需要的是类型A，你都能够使用类型B当作A的值，类型B就是类型A的子类型。
- 子类型：超类型是子类型的反义词

为什么一个类型是否是另一个类型的子类型那么重要？`编译器为每个变量赋值的时候或者给函数传参的时候都要做这项检查`，简单情况下，子类型和子类本质上意味着一样的事物，但可空类型提供了一个例子，说明子类型和子类不是同一事物。当我们开始谈论泛型类型时，子类型和子类之间的差异显得格外重要。例如把`List<String>`是否是`List<Any>`的子类型。

**变型概念**：一个泛型类，比如`MutableList<T>`，如果对于任意两种类型 A 和 B，`MutableList<A>`既不是`MutableList<B>`的子类型也不是它的超类型，它就被称为在该类型参数上是型变的。Java 中所有的类都是不型变的。


### 协变：保留子类型化关系

一个协变类是一个泛型类，以`Producer<T>`为例，对这种类来说，下面的描述是成立的，如果 A 是 B 的子类型，那么 `Producer<A>` 就是 `Producer<B>` 的子类型，我们说`子类型化被保留了`，例如`Producer<Cat>` 是 `Producer<Animal>` 的子类型，因为 Cat 是 Animal 的子类型。

在 Kotlin 中要声明某个类型参数上是可以协变的，在该参数类型的名称前加上 out 关键字即可：

```kotlin
interface Producer<out T>{
    fun produce():T
}
```
将一个类型声参数标记为协变的，在该`类型实参没有精确匹配到`函数中定义的类型形参时，可以让该类的值作为这些函数的实参传递，例如：

```kotlin
fun printCOntents(list:List<Any>){
    println(list.joinToString())
}

println(listof("ABC","BAC"))//List<String> pass to List<Any>
```

**不能把任何类都生命为协变的：这样不安全**：

- 让类在某个类型参数变为协变，限制了该类中对该类型参数使用的可能性，要保证类型安全，它只能用在所谓的 out 位置，意味着这个类只能生产类型 T的值，而不能消费它们

**in 位置 和 out 位置**：

- 考虑这样一个类，它声明了一个类型参数 T 并包含了一个使用 T 的函数，如果函数是把 T 当成返回类型，我们就说它在 out 位置，这种情况下，该函数`生产`类型 T 的值，如果 T 用做函数参数的类型，它就在 in 位置，这样的函数`消费`T 类型的值。
- 类的类型参数之前的 out 关键字要求所用使用 T 的地方只能把 T 放在 out 位置，而不能放在 in 位置，这个关键字约束了使用 T 的可能性，但这保证了对应子类型关系安全性

**关键字 out 的两层含义**：

- 子类型化会被保留，比如`Producer<Cat>` 是 `Producer<Animal>` 的子类型
- T 只能用在 out 位置

参下面代码：

```kotlin
//List 上的 E 声明在了 out 位置，List 只生产 E 类型的值
public interface List<out E> : Collection<E> {
    //...
    public fun subList(fromIndex: Int, toIndex: Int): List<E>//这里的T也在 out 位置
}

//MutableList 不能在 E 上声明成协变的，因为 E 用在了 in 位置，编译器强制实施这种限制
public interface MutableList<E> : List<E>, MutableCollection<E> {
    //...
    override fun add(element: E): Boolean
}
```

**一些细节：**

```kotlin
private open class Animal {
    fun feed() {}
}

private class Cat : Animal()

//构造方法的参数既不在 in 位置，也不在 out 置位，即使声明了 out，仍然可以在构造函数中使用，构造方法不是那中在实例创建后还能调用的方法，因此它不存在危险
private class Herd1<out T : Animal>(vararg animals: T)

/*然而如果声明了一个 var 参数，因为var默认生成了get/set, 即可变属性leadAnimal在out和in位置上都是用了T，而T只被声明为out，此时 T 不能被声明为 out*/
private class Herd2<out T : Animal>(val leadAnimal: T, vararg animals: T)
private class Herd3<T : Animal>(var leadAnimal: T, vararg animals: T)

/*位置规则只覆盖了类外部可见的API，即public/protected/internal的，私有方法的参数既不在in位置，也不在out位置，变型规则只会防止外部使用者对类的误用，但不会对类自己起作用，
此时 T 又能被声明为 out，因为*/
private class Herd4<out T : Animal>(private var leadAnimal: T, vararg animals: T)
```

---
### 逆变：反转子类型化关系

逆变的概念可以看作是协变的镜像，它与协变刚好相反。参考 Comparator 接口例子：

```kotlin
interface Comparator<in T>{
    fun compare(el:T,e2:T):Int
}
```

这个接口方法只是消费类型为 T 的值，这说明 T 只在 in 位置使用，因此它的声明之前用了 in 关键字。

参考下面代码：anyComparator可用于比较任意类型，sottedWith 接收的参数类型为：`Comparator<String>)`，由于逆变性，`Comparator<Any>` 成为了 `Comparator<String>` 的子类型

```kotlin
/*逆变性：Comparator这个接口只是为了消费T类型，这说明T只在in位置使用，因此它的声明之前添加了in关键字，现在Comparator可以用于比较任何类型*/
private val anyComparator = Comparator<Any> { e1: Any, e2: Any ->
    e1.hashCode() - e2.hashCode()
}

private fun testAnyComparator(){
    val strings = listOf("A","D","B")
    //sortedWith方法签名：sortedWith(comparator: Comparator<in T>): List<T>
    //sottedWith接收的参数类型为： Comparator<String>): List<T>
    strings.sortedWith(anyComparator)
}
```

如果 B 是 A 的子类型，那么 `Consumer<A>` 就是 `Consumer<B>` 的子类型，类型参数 A 和 B 换了位置，所以我们说子类型化被反转了。

**in 关键字的意思**

- 对应类型的值是`传递进来`给这个类的方法的，并且被这个类所消费。

**协变、逆变、不变型类对比**

协变 | 逆变 | 不变型
---|---|---
`Producer<out T>` | `Consumer<in T>` | `MutableList<T>`
类的子类型化被保留了：`Producer<Cat>`是`Producer<Animal>`的子类型 | 类的子类型化被反转了：`Consumer<Animal>`是`Consumer<Cat>`的子类型 | 没有子类型化
T 只能在 out 位置 | T 只能在 in 位置 | T 能在任意位置(可读写不变的泛型是不可变的，比如 `a >= 2 又要 a <= 2`，那么 `a = 2`)



**一个类可以在一个类型上协变，同时可以在另一个类型参数上逆变**

- 比如 标准库中的 Function1，Function1 消费 P1 类型且只生产 R 类型
```kotlin
public interface Function1<in P1, out R> : Function<R> {
    public operator fun invoke(p1: P1): R
}
```

---
### 使用点型变：在类型出现的地方指定变型

- 在类声明的时候就能够指定变型修饰符是很方便的，因为这些修饰符会应用到所有类被使用的地方，这种被称作 **声明点型变**
- Java 用完全不同的方式处理型变，通配符类型`? extends 和 super`，Java 中每次使用带类型参数的类型的时候，还可以指定这个类型参数是否可以用它的子类型或者超类型替换，这叫做 **使用点型变**


**声明点型变** vs **使用点型变**：声明点型变带来了更加间接的代码，因为只要指定一次变型修饰符，所有这个类的使用者都不需要再考虑这些，Java 中库的作者不得不一直使用通配符`Function<? super T , ? extends R>` 来创建安装用户期望运行的 API：

 ```java
public interface Stream<T>{
    <R> Stream<R> map(Function<? super T , ? extends R> mapper);
}
 ```

Kotlin 也支持使用点型变， 允许在类型参数出现的位置指定变型，即使在类型声明时它不能被声明为协变或逆变的：

```kotlin
//该方法要求两个参数泛型实参必须一致，但其实source只用于读取，destination只用于添加，这种情况下集合元素的类型不需要精确匹配，例如把一个字符串的集合拷贝到可以包含任意对象的集合中不存在问题
private fun <T> copyData1(source: MutableList<T>, destination: MutableList<T>) {
    for (item in source) {
        destination.add(item)
    }
}

//要让函数支持不同的类型，可以引入第二个泛型参数，为了能够进行拷贝，source中的元素类型应该是destination中元素类型的子类型
private fun <T : R, R> copyData2(source: MutableList<T>, destination: MutableList<R>) {
    for (item in source) {
        destination.add(item)
    }
}

private fun testCopyData() {
    val ints: MutableList<Int> = mutableListOf(1, 2, 3)
    val anyItems: MutableList<Any> = mutableListOf<Any>()
    copyData2(ints, anyItems)
    println(anyItems)
}
```

但是 Kotlin 提供了一种更优雅的表达方式。当函数的实现调用了那些类型参数只出现在 out 位置或者 in 位置的方法时，可以在函数定义上给特定用途的类型参数加上变型修饰符：

```kotlin
//可以个类型的用法加上 out 关键字：没有使用那些 T 用在 in 位置上的方法
private fun <T> copyData3(source: MutableList<out T>, destination: MutableList<T>) {
    for (item in source) {
        destination.add(item)
    }
}

private fun testCopyData() {
    val ints: MutableList<Int> = mutableListOf(1, 2, 3)
    val anyItems: MutableList<Any> = mutableListOf<Any>()
    copyData3(ints, anyItems)
    println(anyItems)
}
```

可以给类型声明中类型参数任意的方法执行变型修饰符，这些用法包括：`形参类型、局部变量类型、函数返回值类型`等等。这里发生的一切称为 **类型投影**，即 source 不是一个常规的 MutableList，而是一个投影（受限）的 MutableList，智能调用返回类型是泛型参数的那些方法，或者说只能在 out 位置使用它的方法，而且编译器禁止调用使用类型参数做实参类型的那些方法（在 in 位置使用类型参数）：

```kotlin
val list:Mutable<out Number> = ...
list.add(42)//无法编译
```

带 in 投影类型参数的数据拷贝函数：
```kotlin
//与copyData3效果一致
private fun <T> copyData4(source: MutableList<T>, destination: MutableList<in T>) {
    for (item in source) {
        destination.add(item)
    }
}
```

**Kotlin 的`点变型`直接对应 Java 的限界通配符**：Kotlin 中的 `MutableList<out T>` 和 Java 中的 `MutableList<? extends T>` 是一个意思，而 in 投影的 `MutableList<in T>` 对应 Java 中的 `MutableList<? super T>`

---
### 星号投影：使用 `*`代替类型参数

有时你对类型参数一无所知，但仍然希望以安全的方式使用它。 这里的安全方式是定义泛型类型的这种投影，该泛型类型的每个具体实例化将是该投影的子类型。

使用星号投影可以用来表明不知道关于泛型的任何信息，但需要注意的是 `MutableList<*>` 和 `MutableList<Any?>` 不一样，`MutableList<Any?>`可以包含任何类型的元素，而`MutableList<*>`是包含某种特定类型元素的列表。其次星号投影与 Java 中的通配符对应，Kotlin 的 `MyType<*>` 对应于 Java 的 `MyType<?>`。

```kotlin
private fun printFirst(list: List<*>) {
    if (list.isNotEmpty()) {
        println(list.first())
    }
}
```

星投影语法：

1. 对于 `Foo <out T>`，其中 T 是一个具有上界 TUpper 的协变类型参数，`Foo <*>` 等价于 `Foo <out TUpper>`。这意味着当 T 未知时，你可以安全地从 `Foo <*>` 读取 TUpper 的值。
2. 对于 `Foo <in T>`，其中 T 是一个逆变类型参数，`Foo <*>` 等价于 `Foo <in Nothing>`。 这意味着当 T 未知时，没有什么可以以安全的方式写入 `Foo <*>`。
3. 对于 `Foo <T>`，其中 T 是一个具有上界 TUpper 的不型变类型参数，`Foo<*>` 对于读取值时等价于 `Foo<out TUpper>` 而对于写值时等价于 `Foo<in Nothing>`。

如果泛型类型具有多个类型参数，则每个类型参数都可以单独投影。 例如，如果类型被声明为 `interface Function <in T, out U>`，可以想象以下星投影：

```kotlin
Function<*, String> 表示 Function<in Nothing, String>；
Function<Int, *> 表示 Function<Int, out Any?>；
Function<*, *> 表示 Function<in Nothing, out Any?>。
```

参考下面代码：

```kotlin
private interface Function<in T, out U> {
    fun invoke(t: T): U
}

private fun map1(function: Function<*, String>) {

}

private fun map2(function: Function<Int, *>) {

}

private fun map3(function: Function<*, *>) {

}

     map1(object : Function<Nothing, String> {
        override fun invoke(t: Nothing): String {
            return ""
        }
    })

    map2(object : Function<Int, Date> {
        override fun invoke(t: Int): Date {
            return Date()
        }
    })

    map3(object : Function<Nothing, Number> {
        override fun invoke(t: Nothing): Number {
            return 3
        }
    })
```


---
## 4 其他

### `@UnsafeVariance`注解

考虑下面代码，add方法的不允许定义T类型参数

```kotlin
private class MyCollection<out T> {
    // ERROR!，为什么？因为 T 是协变的

    //fun add(t: T) {
    //}
}
```
如果上面add编译通过，考虑下面代码，原本只能存入Int类型的集合现在可以存入浮点类型了
```
private fun test() {
    var myList: MyCollection<Number> = MyCollection<Int>()
    //myList.add(3.0)
}
```

对于协变的类型，通常我们是不允许将泛型类型作为传入参数的类型的，或者说，对于协变类型，我们通常是不允许其涉及泛型参数的部分被改变的。逆变的情形正好相反，即不可以将泛型参数作为方法的返回值。


但实际上有些情况下，我们不得已需要在协变的情况下使用泛型参数类型作为方法参数的类型：

```kotlin
public interface Collection<out E> : Iterable<E> {
    ...
    //为了让编译器放过一马，我们就可以用 @UnsafeVariance 来告诉编译器：“我知道我在干啥，保证不会出错，你不用担心”
    public operator fun contains(element: @UnsafeVariance E): Boolean
    ...
}
```

---
## 5 解决重复依赖

下面： IMvpView 要求有一个 IPresenter 泛型参数，IPresenter 又要求有一个 IMvpView 泛型参数，导致出现泛型循环引用，解决方案是后面的 IPresenter 的泛型参数可以引用前面定义的泛型参数

```kotlin
interface ILifecycle{}

interface IPresenter<out View: IMvpView<IPresenter<View>>>: ILifecycle{
    val view: View
}

interface IMvpView<out Presenter: IPresenter<IMvpView<Presenter>>>: ILifecycle{
    val presenter: Presenter
}
```