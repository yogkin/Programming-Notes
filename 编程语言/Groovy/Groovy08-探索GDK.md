# 第七章：探索GDK

---
## 1 使用Object类的扩展

Groovy在Collection上添加`了each，find，collect`等方法，其实不仅Collection可以使用这些方法，其任何对象都是可以使用的，这为我们以类似的方式使用不同的对象和集合体提供了一致的api。

而Object上也添加了一些与集合无关的方法，查看GKD文档可以更加清晰的了解：

### dump 和 inspect方法

```groovy
//如果想知道一个实例包含哪些内容，可以在运行时使用dump方法
str = 'hello'
println str
println str.dump()//<java.lang.String@5e918d2 value=hello hash=99162322>

//inspect方法旨在说明创建一个对象需要提供什么,如果类没有实现该方法，则简单的返回toString方法
println BigDecimal.inspect()
```


### 使用上下文with方法

```groovy
//with作用域范围内调用任何方法都会被定向到上下文你对象
list = [1,3]
list.add(3)
println list.size()
println list.contains(3)
//使用with省略调用者
list.with {
    add(5)
    println size()
}
//with方法使用的是闭包的delegate特性
list.with {
    println "this is $this"
    println "owner is $owner"
    println "delegate is $delegate"
}
//打印结果
this is ztiany.chapter7._001ExploreGDK@399f45b1
owner is ztiany.chapter7._001ExploreGDK@399f45b1
delegate is [1, 3, 3, 5]
```


### 使用sleep方法

```groovy
thread = Thread.start {
    println "thread start"
    startTime = System.nanoTime()
    new Object().sleep(2000)
    endTime = System.nanoTime()
    println "thread done in ${(endTime -startTime)/10**9} seconds"
}
new Object().sleep(300)
println "let interrupt the thread"
thread.interrupt()
thread.join()
```

在Object方法上调用sleep与使用Java提供的sleep方法不同的是,如果发生interruptedException，前者会压制，如果我们确实想被中断，也不必受try/catch块之苦

```groovy
def playWithSleep(boolean flag) {
    thread = Thread.start {
        println "thread start"
        startTime = System.nanoTime()
        new Object().sleep(2000){
            flag//返回true表示希望被interruptedException中断
        }
        endTime = System.nanoTime()
        println "playWithSleep flag=$flag thread done in ${(endTime -startTime)/10**9} seconds"
    }
    thread.interrupt()
    thread.join()
}
playWithSleep(true)
playWithSleep(false)
```


### 间接访问属性

当我们需要访问对象的位置属性时，可以使用[]操作符动态的访问属性，该操作符映射到的方法是Groovy添加的getAt方法,如果该操作符用于对象的左侧，它映射到的是putAt方法

```groovy
class Car{
    int miles,fuelLevel
}

car = new Car(fuelLevel: 3,miles: 32)
properties =['miles', 'fuelLevel']//假设这是不确定的

properties.each {name->
    println "$name = ${car[name]}"
}

car[properties[1]] = 2333
println car.fuelLevel
/*
结果：
miles = 32
fuelLevel = 3
2333
 */
```


### 间接的调用方法

Groovy为每类都添加了invokeMethod方法，当需要调用一个未知的方法时，只需要简单的调用invokeMethod方法，而不是
像Java那样使用反射，还要处理各种异常。

```groovy
class Person{
    def walk() {
        println "walking..."
    }

    def walk(int miles) {
        println "wrlking $miles"
    }
}

peter = new Person()
peter.invokeMethod('walk',null)
peter.invokeMethod('walk',21)
/*
walking...
wrlking 21
 */
```

### java.lang的扩展

对于基本量行的的包装类型，有一个值得注意的地方，那就是重载 `plus : +, next:++`，Number上加上了`upto、downto、step`等方法

---
## 2 其他扩展

更详细的GDK所做的扩展，参考GDK文档，这里介绍的只是一个子集

### 数组的扩展

所有的数据都可以使用range类型进行索引

```groovy
arr = [3, 4, 6, 7, 5, 2, 2, 2, 51, 99921, 65, 21, 12] as int[]
println arr[1..4]
```
### Process类

Process类提供了访问`stdin，stdout，stderr`的命令，分别对应out,in,err属性
还有一个text属性，可以为我们提供完整的标准输出或者来进程的响应，如果想一次性读取读取完整的错误，可以在
进程实例上使用err.text，使用<<操作符可以以管道的方式连接到一个进程中，(管道：在Unix系统中用于将一个进程的输出链接到另一个进程的输入)

```groovy
def execute = 'cmd'.execute()//执行cmd获取一次进程
execute.out.withWriter {
//使用out的withWriter方法，withWriter方法会将OutputStreamWriter附到OutputStream上，并传给闭包，就像前面学习的一样，闭包或自动关闭流
    it << "dir\r\n"     //输入命令
}
println execute.in.text
```

如果想将命令行参数发送给进程，有两个选择：把参数格式化为一个字符串，扩展创建一个参数的String数组，String[]也支持execute方法，其第一个元素
会被当作命令，其他元素作为参数，作为替代list也可以使用execute方法

```groovy
String[] command = ['groovy', '-e', ' "println \'Groovy\' "   ']
println "calling ${command.join('  ')}"
println command.join(' ').execute().text
```

结果为：
calling groovy  -e   "println 'Groovy' "
Groovy

### 线程相关

Thread的start和startDaemon方法可以快速的启动线程/守护线程

```groovy
Thread.start {
    new FileWriter("test.md").withWriter {
        it << '# 我是一个线程'
        it << System.getProperty("line.separator", "\n")
        it << '- 我是一个线程'
        it << '- 我是一个线程'
    }
}
```

---
## 3 io的扩展

Java中file也加入了很多方面的方法，其中eachFile和eachDir这样的方法可以接受闭包，Java中读取File的文本显得过于繁复,Groovy通过向`BufferedReader、InputStream、File`添加了一个text属性用于一次新读取file或者流中的文本，使之简单的了许多:


```groovy
println new File("test.md").text//读取文档中的文本就是这么简单
//如果不想一次性读取整个文本，可以使用eachLine进行逐行读取
new File("test.md").eachLine {
    println it
}
//还可以对文本进行过滤
println new File("test.md").filterLine {
    String s->
        s.startsWith("#")
}
//如果想使用完毕时自动刷新并关闭流，可以使用withStream方法，该方法会创建一个InputStream的实例作为参数发送给闭包,就像之前学习的闭包的特性一样，流会自动关闭

//InputStream的withReader会创建一个BufferedReader，并将其作为参数传给闭包，也可以调用newReader方法创建一个BufferedReader

new File('test.md').withWriter {
    println it//it is a BufferedWriter
    it << "ni hao a"
}
```

---
## 4 java.util的扩展

- List Set  SortedMap 都加入了asImmutable方法，用于获取对应的一个不可变对象，
- asSynchronized 方法用于创建线程安全的实例
- Iterator支持的inject方法前面学习过
- Timer.runAfter方法接受一个闭包，改闭包将在给定的(毫秒)时间后运行

---
## 5 使用扩展模块定制方法

使用extension-modules特性，我们还可以在编译时向现有类添加方法或者静态方法
步骤：

1. 想要添加的方法必须在一个扩展模块的类中
    - 定义一个类，必须都是静态的方法，第一个参数是该方法需要加入到的类型：比如
2. 需要在清单文件中放置一些辅助信息