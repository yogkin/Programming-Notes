[TOC]

# Coroutines-Guide

- [Guide to kotlinx.coroutines by example](https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md)

## 协程基础

### 1 launch

通过Builders中的launch函数，传入一个suspending函数，可以启动一个协程。

- **launch作用**：启动一个新的协程并返回一个代表协程引用的Job对象，当前线程不会被阻塞，调用launch时可以指定协程上下文。

- **CommonPool作用**：它是一个协程上下文，同时也是一个协程调度器，用于协程调度程序的计算密集型任务。
如果在launch中传入CommonPool，此时协程将会在CommonPoll提供的线程池中运行。

- **delay作用**：在给定的时间内延迟协程，此时该协程将让出执行权(delay是非阻塞的)，仅用于协程中使用。
当一个suspending函数在等待时，如果取消这个它(或者它已经完成了)将会抛出CancellationException异常


### 2 runBlocking

如果我们希望直接在主线程上面创建协程，那么可以使用runBlocking，runBlocking作用:

1. 作为一个适配器，用于启动顶级主协程
2. 运行一个新的协程并且阻塞当前线程直到协程执行完毕， 当前线程可中断。
3. runBlocking适用于主函数或者测试中，
4. 如果这个阻塞的线程被中断, 内部的协程将会被取消，runBlocking 调用抛出InterruptedException.
5. 使用runBlocking启动协程，会阻塞当前的协程，所以不要在协程中再使用runBlocking启动协程，因为这样没有什么意义

### 3 协程等待机制——join

launch返回一个**代表协程调度的后台任务**Job，Job的join方法可以让协程等待另一个协程。

join作用：当调用jon的jon方法时，此时当前协程会等到之前启动的协程运行完毕(Suspends coroutine until this job is complete)

### 4 协程是轻量级的


能创建线程的数量公式：`Number of Threads = (MaxProcessMemory - JVMMemory - ReservedOsMemory) /(ThreadStackSize)`

- MaxProcessMemory 指的是一个进程的最大内存
- JVMMemory JVM内存
- ReservedOsMemory 保留的操作系统内存
- ThreadStackSize 线程栈的大小

一般我们如果创建100_000个线程，铁定会抛出OOM异常，而协程却不会。

协程是轻量级的，它拥有自己的运行状态，但它对资源的消耗却非常的小。
协程不是线程，携程由程序自己管理，所以即使创建1000个携程也不会什么很大的消耗。

```kotlin
fun main(args: Array<String>) = runBlocking {
    //这里创建了100_000协程
    val jobs = List(100_000) {
        launch(CommonPool) {
            delay(2000L)//每一个协程都停一秒，但是相互之间并不阻塞
            print(".")
        }
    }
    println(jobs.size)
    jobs.forEach { it.join() }
    println(" end ....")
}
```

### 5 协程就像守护线程一样

当主线程退出时，协程也会退出。

```kotlin
fun main(args: Array<String>) = runBlocking {
    //withTimeout作用：Runs a given suspending block of code inside a coroutine with a specified timeout and throws
    // CancellationException if timeout was exceeded.
    //给定时间被协程没有执行完毕，将会抛出异常
    try {
        withTimeout(1300L) {
            repeat(1000) { i ->
                println("I'm sleeping $i ...")
                delay(500L)
            }
        }
        println("ending=---")
    } catch (e: CancellationException) {
        e.printStackTrace(System.err)
    }
}
```

## 撤销与超时

---
### 1 取消协程

有时候需要细粒度的控制协程，launch返回的Job提供了cancel方法，可用于取消已经启动的协程。

1. kotlinx.coroutines中的所有suspend函数都可以取消。
2. 但是，如果协程正在运行中，并且不检查取消状态，则它不能被取消(即使调用了cancel)，

---
### 2 提高协程间的协作能力

在使用协程时，代码应该检查自己是否被已取消了，以及时的响应其他协程。提高协程间的协作能力

有两种方法可以让计算代码变得可以被取消：

1. 定期调用一个挂定函数，比如yield
2. 在协程内明确地检查自己的取消状态，使用isActive属性

---
### 3 在协程关闭时释放资源

可取消的suspending函数在取消时抛出CancellationException，可以用通常的异常处理方式来处理异常，
并且在必要的时候释放协程中使用的公共资源。

---
### 4 实现不可取消的协程

任何在finally块中使用暂停函数的操作都会导致CancellationException，因为运行此代码的协程被取消了，通常情况下，这不是问题，
因为所有关闭资源的操作（关闭文件，取消作业或关闭任何类型的通讯通道）通常都是非阻塞的，不涉及任何挂起功能，但是，在罕见的情况下，
如果需要在已取消的协程中使用suspend ，则可以使用`run(NonCancellable) {...}`来包装代码块：

```kotlin
val job = launch {
        try {
            repeat(1000) { i ->
                println("I'm sleeping $i ...")
                delay(500L)
            }
        } finally {
        //使用run和NonCancellable  
            run(NonCancellable) {
                println("I'm running finally")
                delay(1000L)
                println("And I've just delayed for 1 sec because I'm non-cancellable")
            }
        }
    }
```
run在Builders中提供，其作用是：启动一个suspending函数，并暂停当前函数直到该suspending函数执行完毕，并返回结果。

---
### 5 给协程设计超时

可以给一个协程设置超时时间，给定时间被协程没有执行完毕，将会抛出异常：TimeoutCancellationException


---
## 编写异步风格的代码

示例代码：

```kotlin
    val result1 = async(CommonPool) {
        delay(3000)
        1
    }
    val result2 = async(CommonPool) {
        delay(3000)
        2
    }

    //await表示，当Deferred代表的协程调度完毕，会在指定的协程剩下文中调度await所在的代码块。
    val launch = launch(UI) {
        println("${result1.await() + result2.await()}")
    }
```

上面是一个标准的异步风格写成的代码，以 CommonPool 开启两个异步协程，并获取结果，launch一个新的协程，优雅无阻塞的等待异步异步任务返回结果。

---
##  协程上下文与调度器

协程总是在由CoroutineContext表示的上下文中执行。
协程的上下文是一组不同的elements。

---
### 1 Dispatchers(调度器)

CoroutineContext 包含一个Dispatchers，它确定对应协程用于执行的线程或线程池。Dispatchers可以将协程执行限制在一个特定的线程上，将其调度到线程池，或者让其无限制地运行。

Kotlinx协程中提供了默认的协程调度器，还允许我们实现自己的协程调度器：

- Unconfined：表示不约束协程调度线程，它初始化时使用当前线程作为调度器，然后如果它被挂起了，它将使用恢复它的线程中执行，
所以每次它被挂起恢复后，调度它的线程就会发生变化
- CommonPool：如果ForkJoinPool可用，CommonPool将使用ForkJoinPool线程池调度协程
- ThreadPoolDispatcher：ThreadPoolDispatcher中提供了一些函数用于创建可用的协程调度器，比如newSingleThreadContext函数，需要
注意但是，如果不在需要创建的协程调度器，应该及时的释放它们。

---
### 2 Unconfined

Unconfined调度程序适用于不消耗CPU时间，也不更新任何局限于特定线程的共享数据（如UI）。

---
### 3 coroutineContext属性

通过CoroutineScope接口在任何协程块中可以访问coroutineContext属性，coroutineContext是对这个特定协程的上下文的引用。

---
### 4 协程调试

启动VM时，指定参数：`-Dkotlinx.coroutines.debug`

---
### 5 Job

1.  Job是其context的一部分。协程可以使用`coroutineContext[Job]`表达式获取job
2. 当一个协程的context用于去启动其他协程，新的协程的Job将会成为其父协程Job的子Job，当
父协程被取消，所有的字写成也会被递归的取消。

---
### 6 协程上下文组合

1. 使用`+`运算符可以组合多个上下文，右侧的上下文替换左侧上下文的相关条目，包括协程调度器。
2. 组合CoroutineName可以给协程命名

---
### 7 管理多个协程

```kotlin
    val job = Job() // create a job object to manage our lifecycle

    // now launch ten core.base for a demo, each working for a different time
    val coroutines = List(10) { i ->
        // they are all children of our job object
        launch(coroutineContext + job) {
            // we use the context of main runBlocking thread, but with our own job object
            delay(i * 200L) // variable delay 0ms, 200ms, 400ms, ... etc
            println("Coroutine $i is done")
        }
    }

    println("Launched ${coroutines.size} core.base")
    delay(500L) // delay for half a second
    println("Cancelling job!")
    job.cancel() // cancel our job.. !!!
```

---
## 通道

---
### 1 Channels 是概念

Deferred(延期的值)提供了在协程之间传输单个值的便利方式。而Channels 提供了一种传输流的方式。Channels的概念非常类似BlockingQueue，一个关键的不同是：挂起的send操作代替了阻塞的put 操作；挂起的 receive操作代替了阻塞的take 操作

Channels就像多线程中的BlockingQueue，只是把阻塞换成了挂起。

- Channels可以指定容量
- 如果容量已满，继续send数据会挂起协程
- 如果Channels中没有数据，调用receive也会挂起协程

### 2 生产者与消费者模式

BlockingQueue可用于实现多线程环境中的生产者与消费者模型，同样Channel用于也可以用于在协程中实现这样的模型。
使用Product中的produce方便的实现(生产者与消费者模型)。
