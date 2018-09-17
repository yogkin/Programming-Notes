# 运算符重载

Kotlin 允许我们为自己的类型提供预定义的一组操作符的实现。这些操作符具有固定的符号表示 （如 `+ 或 *`）和固定的优先级。为实现这样的操作符，我们为相应的类型（即二元操作符左侧的类型和一元操作符的参数类型）提供了一个固定名字的成员函数或扩展函数。重载操作符的函数需要用 `operator` 修饰符标记。


可以重载的操作符分类：

-  一元操作符
-  递增和递减
-  二元操作
-  “In”操作符
-  索引访问操作符
-  调用操作符
-  广义赋值
-  相等与不等操作符
-  比较操作符
-  命名函数的中缀调用


Kotlin 中的运算符重载基于**约定**

---
## 1 重载运算符

### 重载二元运算符

可重载的二元操作符：

| 表达式 | 翻译为 |
| --- | --- |
| `a + b` | `a.plus(b)` |
| `a - b` | `a.minus(b)` |
| `a * b` | `a.times(b)` |
| `a / b` | `a.div(b)` |
| `a % b` | `a.rem(b)`、 `a.mod(b)` （已弃用） |
| `a..b` | `a.rangeTo(b)` |


- 操作符重载可以以成员函数的形式定义，也可以以扩展函数的形式定义
- 从 Java 调用 Kotlin 运算符非常容易，因为每个运算符重载都会被定义为一个普通函数
- Kotlin 运算符不会自动支持交换性，即交换运算符的两边操作对象的位置
- 运算符可以重载，即定义不同参数类型的操作符
- Kotlin 没有为数字类型定义任何的位运算，因此我们也无法为类型提供类似操作符，取代为运算符的是一系列中缀表达式：`shl、shr、ushr、and、or、xor、inv`

```
data class Point(val x: Int, val y: Int) {
    operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
}

fun main(args: Array<String>) {
    val p1 = Point(10, 20)
    val p2 = Point(30, 40)
    println(p1 + p2)
}
```

### 重组赋值运算符：`plus` 和 `plusAssign` 对比

像`+=、-=`这样的运算符称作**复合运算符**

```
fun main(args: Array<String>) {
    var point = Point(1, 2)
    point += Point(3, 4)
    println(point)
}
```
- 如果定义一个返回值为 Unit ，名为 `plusAssign` 的函数，Kotlin 将会在使用 `+=` 运算符的地方调用该函数
- 当你在代码中调用 `+=` 的时候，理论上 `plus` 和 `plusAssign` 都会被调用，如果这种情况下两个函数都有定义且适用，编译器会报错，解决方式是：1 使用普通函数的形式；2 用 val 替换 var，这样 plusAssing 就不再适用了
- 一般来说，尽量不要给一个类同时添加 plus 和 plusAssign 运算
- 如果一个对象是不可变的，应该为其定义 plus 运算符，如果一个类是可表变的，那么可以为其定义 plusAssign 运算符

集合相关：

- kotlin 集合支持两种方法，`+` 和 `-` 运算符总是返回要给新的集合，`+=` 和 `-=`用于可变集合时，始终就地的修改它们，而它们用于不可变集合时，会返回一个修改过的副本
- 只有当集合引用声明为可变集合且使用 val 修饰时采用使用 `+=` 和 `-=`

```
fun main(args: Array<String>) {
    val numbers = ArrayList<Int>()
    numbers += 42
    println(numbers[0])
}

fun main(args: Array<String>) {
    val list = arrayListOf(1, 2)
    list += 3
    val newList = list + listOf(4, 5)
    println(list)//1,2,3
    println(newList)// 1,2,3,4,5
}
```

### 重载一元操作符

用于重载一元操作符的函数，没有任何参数

| 表达式 | 翻译为 |
| --- | --- |
| `+a` | `a.unaryPlus()` |
| `-a` | `a.unaryMinus()` |
| `!a` | `a.not()` |
| `++a、a++` | `a.inc()` |
| `--a、a--` | `a.dec()` |

```
operator fun Point.unaryMinus(): Point {
    return Point(-x, -y)
}

fun main(args: Array<String>) {
    val p = Point(10, 20)
    println(-p)
}
```

---
## 2 重载比较运算符

### 等号运算符：equals

- kotlin 中的 `==、!=` 会被替换为 equals
- `==、!=` 可用于 null 值，kotlin内部已经做了判断
- 恒等运算符 `===` 不能被重载

```
public open class Any {
...省略其他方法

public open operator fun equals(other: Any?): Boolean

}
```

由于在 Any 中已经将 equals 定义为 operator，所以重载是不需要添加 operator


### 排序运算符：compareTo

- Kotlin 支持相同的 Comparable 接口，比较运算符（`>，<，>=，<=`）将被替换为 compareTo
- Kotlin 提供了 compareValuesBy 方法，用于快速实现字段比较
- `p1 < p2` 等价于 `p1.compareTo(p2) < 0`

```kotlin
class Person(
        val firstName: String, val lastName: String
) : Comparable<Person> {

    override fun compareTo(other: Person): Int {
        return compareValuesBy(this, other,
            Person::lastName, Person::firstName)
    }
}

fun main(args: Array<String>) {
    val p1 = Person("Alice", "Smith")
    val p2 = Person("Bob", "Johnson")
    println(p1 < p2)
    println("abc" < "bac")
}
```

---
## 3 集合与区间的约定

语法 `a[b]` 称为下表运算符

### 通过下标方法元素：`set、get`

- Kotlin 中，下标运算符是一个约定
- `get(row: Int, col: Int)` 形式同样可以使用下标运算符 `obj[row, col]`

```kotlin
operator fun Point.get(index: Int): Int {
    return when(index) {
        0 -> x
        1 -> y
        else ->
            throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}

fun main(args: Array<String>) {
    val p = Point(10, 20)
    println(p[1])
}

//可变的点
data class MutablePoint(var x: Int, var y: Int)

operator fun MutablePoint.set(index: Int, value: Int) {
    when(index) {
        0 -> x = value
        1 -> y = value
        else ->
            throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}

fun main(args: Array<String>) {
    val p = MutablePoint(10, 20)
    p[1] = 42
    println(p)
}
```

### `in` 的约定

- `in` 操作符与 contains 函数对应：

### rangeTo 的约定

- `..` 运算符是 rangeTo 函数的一个简洁方式
- rangeTo 用于创建一个闭区间，而 until 用于创建一个开空间

rangeTo 通用可以定义在类型上，比如标准库中的：

```kotlin
operator fun <T:Comparable<T>> T.rangeTo(that:T):Comparable<T>
```

代码示例:

```kotlin
val now = LocalDate.now()
val vacation = now..now.plusDays(10)
println(now.plusWeeks(1) in vacation)//true
```

### 在 `for` 循环中使用 iterator 的约定

- Kotlin 中，for 循环中可以使用 in 运算符，这和区间检查的含义不同，它被用来执行迭代，这意味着`for(x in list){}`将被翻译为`list.iterator()`的调用
- 在 Kotlin 中，iterator 也是一种约定，这意味着 iterator 可以被定义为扩展函数，标准库中的 Stirng 迭代就是使用这个约定

```kotlin
operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> =
        object : Iterator<LocalDate> {
            var current = start

            override fun hasNext() =
                current <= endInclusive

            override fun next() = current.apply {
                current = plusDays(1)
            }
        }

fun main(args: Array<String>) {
    val newYear = LocalDate.ofYearDay(2017, 1)
    val daysOff = newYear.minusDays(1)..newYear
    for (dayOff in daysOff) { println(dayOff) }
}
```

### 解构声明和组件函数

有些时候, 能够将一个对象解构(destructure) 为多个变量, 将会很方便，这种语法称为解构声明(destructuring declaration). 一个解构声明会一次性创建多个变量解构声明在编译时将被分解为以下代码：

```kotlin
        val name = person.component1()
        val valage = person.component2()
```

结构声明通用使用的是约定的原理。

- 要做解构声明中初始化每个变量，将会调用名为 componentN 的函数，而 data class 会自动为字段声明这些函数
- 不能定义无限数量的 componentN 函数，最多 5 个
- 返回多个值的方法也可以使用 Pair 和 Triple 类
- 解构声明通用可以用于循环中

```kotlin
    //使用 Pair和Triple
    val p = Pair(1,"ABC")
    val t = Triple(1, "C", 'G')
    //循环中使用解构声明，遍历map，之所致可以这样遍历，是应为Map提供了componentN函数
    fun printEntries(map: Map<String, String>) {
        for ((key, value) in map) {
            println("$key -> $value")
        }
    }
```
