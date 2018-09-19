# 委托

---
## 1 类的委托

类的委托：委托模式已经证明是实现继承的一个很好的替代方式， 而 Kotlin 可以零样板代码地原生支持它。

```kotlin
private interface MachineGun {
    fun shooting()
}

/** 飞机 */
private interface Plane {
    fun fly()
}


/** 战斗机 */
private class Fighter(plane: Plane, gun: MachineGun) : Plane by plane, MachineGun by gun

private fun testClassDelegation() {
    
    val gun = object : MachineGun {
        override fun shooting() {
            println("飞机扫射")
        }
    }

    val plane = object : Plane {
        override fun fly() {
            println("飞机起飞")
        }
    }

    //传入代理实现
    val fighter = Fighter(plane, gun)
    fighter.fly()
    fighter.shooting()
}
```

---
## 2 属性委托

有一些常见的属性类型，虽然我们可以在每次需要的时候手动实现它们， 但是如果能够为大家把他们只实现一次并放入一个库会更好。

>Kotln 中最强大和最独特的功能之一：委托属性。这个功能的基础是委托，这是一种设计模式，操作的对象不用自己去执行，而是把工作委托给另一个辅助对象，我们把赋辅助对象称之为委托

### 2.1 适用于委托的属性

- 延迟属性（lazy properties）: 其值只在首次访问时计算。
- 可观察属性（observable properties）: 监听器会收到有关此属性变更的通。
- 把多个属性储存在一个映射（map）中，而不是每个存在单独的字段中。

为了涵盖这些属性，Kotlin支持委托属性，语法是： `val/var <属性名>: <类型> by <表达式>`

- 在 `by` 后面的表达式是该委托， 因为属性对应的 `get()`（和 `set()`）会被委托给它的 `getValue()` 和 `setValue()` 方法。
- 属性的委托不必实现任何的接口，但是需要提供一个 `getValue()` 函数（和 `setValue()`——对于 var 属性）


### 2.2 标准委托

Kotlin 标准库为几种有用的委托提供了工厂方法

- 延迟属性 Lazy
- 可观察属性 Observable
- 把属性储存在映射中
- 局部委托属性

#### 延迟属性 Lazy

`lazy()` 是接受一个 lambda 并返回一个 Lazy <T> 实例的函数，返回的实例可以作为实现延迟属性的委托：

- 第一次调用 get() 会执行已传递给 lazy() 的 lambda 表达式并记录结果， 后续调用 get() 只是返回记录的结果。
- 默认情况下，对于 lazy 属性的求值是同步锁的（synchronized）：该值只在一个线程中计算，并且所有线程 会看到相同的值。
- 如果初始化委托的同步锁不是必需的，这样多个线程 可以同时执行，那么将LazyThreadSafetyMode.PUBLICATION 作为参数传递给 lazy() 函数。
- 而如果你确定初始化将总是发生在单个线程，那么你可以使用 LazyThreadSafetyMode.NONE 模式， 它不会有任何线程安全的保证和相关的开销。

```kotlin
//默认是同步初始化
private val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}

//无锁延迟初始化
private val lazyValueUnSafe: String by lazy(LazyThreadSafetyMode.PUBLICATION) {
    println("computed!")
    "Hello"
}
```

#### 可观察属性 Observable

- `Delegates.observable()` 接受两个参数：初始值和修改时处理程序（handler）。 每当我们给属性赋值时会调用该处理程序（在赋值后执行）。它有三个参数：被赋值的属性、旧值和新值
- 如果希望能够截获一个赋值并否决它，就使用 `vetoable()` 取代 `observable()`。 在属性被赋新值生效之前会调用传递给 vetoable 的处理程序。

```kotlin
private class User {

    var name: String by Delegates.observable("<no name>") { property, old, new ->
        println("$property.name now：$old -> $new")
    }

    var age: Int by Delegates.vetoable(0) { property, old, new ->
        println("$property.name now：$old -> $new")
        true
    }
}
```

#### 把属性储存在映射中

一个常见的用例是在一个映射（map）里存储属性的值，这时属性会以属性名从映射中获取属性的值。这经常出现在像解析 JSON 或者做其他动态事情的应用中。在这种情况下，你可以使用映射实例自身作为委托来实现委托属性。

```kotlin
private class MapDelegate(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int     by map
}
```

把属性储存在映射中由`kotlin.collections.MapAccessor`提供支持，如果我们需要扩展，可以参考其源码。


#### 局部委托属性

可以将局部变量声明为委托属性。 例如可以使一个局部变量惰性初始化

```kotlin
private fun localDelegate(computeFoo: () -> String) {

    val memoizedFoo by lazy(computeFoo)

    //memoizedFoo只会在第一次访问时求值
    if (!memoizedFoo.isEmpty()) {
        println("memoizedFoo is $memoizedFoo")
    }
}
```

### 2.3 属性委托要求

对于一个只读属性（即 val 声明的），委托必须提供一个名为 getValue 的函数，该函数接受以下参数：

    thisRef —— 必须与属性所有者类型（对于扩展属性——指被扩展的类型）相同或者是它的超类型
    property —— 必须是类型 KProperty<*> 或其超类型

对于一个可变属性（即 var 声明的），委托必须额外提供一个名为 setValue 的函数，该函数接受以下参数：

     thisRef —— 同 getValue()，
     property —— 同 getValue()，
     new value —— 必须和属性同类型或者是它的超类型。

`getValue() 或/和 setValue() `函数可以通过委托类的成员函数提供或者由扩展函数提供。当需要委托属性到原本未提供的这些函数的对象时后者会更便利。两函数都需要用 operator 关键字来进行标记。Kotlin提供的接口ReadOnlyProperty和ReadWriteProperty中定义了这些操作符，分别对应只读属性和可读写属性。

接口

```kotlin
    public interface ReadOnlyProperty<in R, out T> {
        public operator fun getValue(thisRef: R, property: KProperty<*>): T
    }

    public interface ReadWriteProperty<in R, T> {
        public operator fun getValue(thisRef: R, property: KProperty<*>): T
        public operator fun setValue(thisRef: R, property: KProperty<*>, value: T)
    }
```

示例

```kotlin
private class ADelegate(property: ReadWriteProperty<ADelegationExample, String>) : ReadWriteProperty<ADelegationExample, String> by property

private val propertyA = object : ReadWriteProperty<ADelegationExample, String> {

    private var value = "abc"

    override fun setValue(thisRef: ADelegationExample, property: KProperty<*>, value: String) {
        this.value = value
    }

    override fun getValue(thisRef: ADelegationExample, property: KProperty<*>): String {
        return value
    }
}

private class ADelegationExample {
    var name: String by ADelegate(propertyA)
}
```

### 2.4 翻译规则

在每个委托属性的实现的背后，Kotlin 编译器都会生成辅助属性并委托给它。 例如，对于属性 prop，生成隐藏属性 prop$delegate，而访问器的代码只是简单地委托给这个附加属性：

```java
                 //代理属性
                private class C {
                     var prop: Type by MyDelegate()
                }

                //编译后
                class C {
                     private val prop$delegate = MyDelegate()
                     var prop: Type
                     get() = prop$delegate.getValue(this, this::prop)
                     set(value: Type) = prop$delegate.setValue(this, this::prop, value)
                }
```

### 2.5 提供委托

通过定义 `provideDelegate` 操作符，可以扩展创建属性实现所委托对象的逻辑。如果 by 右侧所使用的对象将 provideDelegate 定义为成员或扩展函数，那么会调用该函数来创建属性委托实例。

- `provideDelegate` 的一个可能的使用场景是在创建属性时（而不仅在其 getter 或 setter 中）检查属性一致性。
- `provideDelegate` 方法只影响辅助属性的创建，并不会影响为 getter 或 setter 生成的代码。


```kotlin
//属性代理1
private class ResourceDelegate1 : ReadOnlyProperty<UI, String> {

    override fun getValue(thisRef: UI, property: KProperty<*>): String {
        return "ABC"
    }
}

//属性代理2
private class ResourceDelegate2 : ReadOnlyProperty<UI, String> {

    override fun getValue(thisRef: UI, property: KProperty<*>): String {
        return "DEF"
    }
}

//用于提供属性代理，属性代理由其内部的provideDelegate操作符提供。
private class ResourceLoader(val resourceId: Int) {

    operator fun provideDelegate(thisRef: UI, prop: KProperty<*>): ReadOnlyProperty<UI, String> {
        //检测属性
        checkProperty(thisRef, prop.name)
        // 创建委托
        return if (resourceId == 1) {
            ResourceDelegate1()
        } else {
            ResourceDelegate2()
        }
    }

    private fun checkProperty(thisRef: UI, name: String) {

    }
}

private class UI {

    fun bindResource(resourceId: Int): ResourceLoader {
        return ResourceLoader(resourceId)
    }

    //image和text由含有成员操作符函数provideDelegate的返回值代理
    val image by bindResource(1)
    val text by bindResource(2)
}
```