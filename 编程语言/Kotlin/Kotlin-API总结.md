# Kotlin API 总结

---
## 1 Standard

```
    let：调用某对象的let函数，则该对象为函数的参数。在函数块内可以通过 it 指代该对象。返回值为函数块的最后一行或指定return表达式。
    
    apply：调用某对象的apply函数，在函数块内可以通过 this 指代该对象。返回值为该对象自己。
    
    run：调用run函数块。返回值为函数块最后一行，或者指定return表达式。
    
    also：调用某对象的also函数，则该对象为函数的参数。在函数块内可以通过 it 指代该对象。返回值为该对象自己。
    
    with：with函数和前面的几个函数使用方式略有不同，因为它不是以扩展的形式存在的。它是将某对象作为函数的参数，在函数块内可以通过 this 指代该对象。返回值为函数块的最后一行或指定return表达式。
```

---
## 2 Unit 和 Nothing

参考 [文章翻译：Kotlin 中的 Nothing 和 Unit](https://zhuanlan.zhihu.com/p/26890263)

---
### Unit

Kotlin 里没有 void，所有函数都有返回类型：

```kotlin
fun doSomething() {
  Log.d("Hello", "World")
}
```

这个函数的返回类型是 Unit，而且所有不显式声明返回类型的函数都会返回 Unit 类型，所以写不写这个 Unit 并没有什么区别。

```kotlin
/**
 * The type with only one value: the Unit object. This type corresponds to the `void` type in Java.
 * 只有一个实例，这个实例相当于Java中的void
 */
public object Unit {
    override fun toString() = "kotlin.Unit"
}
```

从Unit的定义可以看出，Unit 是一个真正的类，继承自 Any 类，只有一个值，因为 Unit 是一个真正的返回类型，所以我们可以把 Unit 类型的值赋给一个变量，但是这样并没有什么作用。

```kotlin
val doSomethingResult = doSomething()
```

### Nothing

Nothing的定义为：

```kotlin
/**
 * Nothing has no instances. You can use Nothing to represent "a value that never exists": for example, if a function has the return type of Nothing, it means that it never returns (always throws an exception).
 *
 * Nothing没有实例对象，可以用 Nothing 表示一个用于不存在的值，比如：如果一个函数返回类型的Nothing，意思就是这个函数永远，不会返回(这个函数总是抛出一个异常)
 */
public class Nothing private constructor()
```

下面两个函数等价，函数的返回类型就是 Nothing。

```kotlin
fun fail() {
    throw RuntimeException("Something went wrong")
}

fun fail(): Nothing {
    throw RuntimeException("Something went wrong")
}
```

我们不需要在代码用 Nothing 类型。Nothing 只是一个编译期的抽象概念（所以没有实例）。只有需要将一个函数显式标记为无法完成时，才有用 Nothing 类型的必要。

---
## 3 使用集合

- 有很多关于集合的方法可以直接使用，比如`setOf()、listOf()、mutableListOf<>()、listOf<>()`等
- 使用集合的`StreamAPI`可以很方便的操作集合数据
- List 的`romoveAt()和remove()` 方法
