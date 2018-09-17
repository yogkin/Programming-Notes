# Groovy介绍

---
## 1 Java平台的三个重要的组成部分

-  java虚拟机，(Java virtual machine),JVM已经变得越来越强大了
-  Java开发包(JDK),包括丰富的第三方类库和框架
-   基于JVM的语言集合，Java语言当然是第一位，这些语言集合可以帮我们在Java平台编写程序


Java语言已经变得非常强大，但我们还是要超越java，寻找更为轻量级的而且高效的语言，如果使用得当，**动态语言**，**函数式编程**，**元编程功能**可以帮助我们更快的航行

---
## 2 Groovy介绍

Groovy基于Java，它是轻量级、限制少、而且还是动态的、面向对象的、并且允许在Java虚拟机上。其保留了Java语法，Groovy编译为java字节码，还扩充了Java API类库，要部署Groovy，只需要引入Groovy的一个Jar包。


---
## 3 为什么要使用动态语言

动态语言能够在运行时扩展程序，包括修改类型，行为、对象结构，利用动态语言，静态语言在编译时做的一些事情可以放在运行时做，甚至可以执行在运行时即时创建的程序语句。

例如下面使用Groovy的例子。


```groovy
    Integer.metaClass.percentRaise = {
        amount ->
            amount * (1 + delegate / 100.0)//delegate 引用的是integer的一个实例
    }

    println 5.percentRaise(8000)
```


这样我们就为Integer动态的添加了一个percentRaise函数，用于计算八千元工资提升了5%后是多少。

**动态语言的灵活性给我们带来了在应用执行时演进程序的优势**，代码合成是在运行时在内存中创建代码，动态语言代码合成更容易让人接受。

动态语言已经存在了很久一段时间，但是下面四个因素让其变得更加流行

-  机器的速度：1 更好的即时编译技术，2 JVM对动态语言的支持
-  可用性 互联网和活跃的基于社区的开发开发方式，使较新的动态语言更易于获取和使用
-  对单元测试的意识 动态语言一般都是动态类型的
-  杀手级应用 Grails

>动态语言中的动态是指**编译时做的一些事情可以放在运行时做**，这与动态类型并不是同一回事

---
## 4 为什么选择Groovy

-  易于掌握
-  遵循java语义
-  满足了我们对动态语言的热爱
-  扩展了JDK

几乎所有的java代码都可以拿来当作groovy代码来执行。

### Groovy更加便捷

Groovy的for循环

```groovy
    3.times {
        println(it)
    }
```

### 在使用groovy编程时，java有的groovy几乎都有，groovy类同样的也扩展了Object类，Groovy类就是java类，面向对象规范和java语义都保留了下来

```groovy
    println(XmlParser.class)
    println(XmlParser.class.superclass)
    打印结果：
    class groovy.util.XmlParser
    class java.lang.Object
```

### Groovy是动态的，类型也是可选的


下面给String添加了一个方法用于检测器是否为回文结构，这样不侵入一个类的源代码，即可使用便捷的方法来轻松的扩展类，即使是不可变的String

```groovy
    String.metaClass.isPalindrome = {
            delegate == delegate.reverse()
    }

    println "abc".isPalindrome()
    println "aba".isPalindrome()
```

### Groovy使用便捷的方法和闭包扩展了JDK

例如下面list的join方法，用于把元素连接起来

```groovy
    list = ["I", "am", "Ztiany"]
    println list.join("   ")
```

---
## 5 学习Groovy的哪些内容

-  Groovy常规编程基础
-  Groovy应用于日常编码，包括使用XML、访问数据库以及使用多个Java/Groovy类和脚本
-  Groovy的元编程能力
-  应用Groovy元编程来创建和使用生成器以及领域特定语言(DSL)

---
## 6 Groovy特性

-  Groovy的松散的Java 语法允许省略**分号**和**修改符**。
-  能够在运行时轻松地为对象指定新方法和属性。这一编程领域称为**元编程**
-  除非另行指定，Groovy 的所有内容都为 `public`。
-  Groovy允许定义简单脚本，同时无需定义正规的 `class` 对象。
-  Groovy 语法还允许省略变量类型。
-  Groovy编译器会自动加上getter/setter方法，所以不需要写
-  Groovy类的属性可以通过"."来获取，看起来他们好像是在java中公用的一样，但是底层Groovy调用的是getter/setter方法
-  如果使用==比较两个类的实例，groovy底层会自动调用对象的equals()方法，这个操作也避免了空指针。