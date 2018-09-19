# 第十九章：在Groovy中创建DSL

---
## 1  DSL

### 什么是DSL

领域定义语言(Domain Specific Language )相关概念

 - 针对的是某一特定类型的问题，
 - 语法聚焦于指定的领域问题，
 - DSL的应用领域和能力都是非常有限的。
 - 其基本思想是“求专不求全”

### 与DSL相对的GPL

与领域定义语言相对的就是通用编程语(GPL)了，GPL即General Purpose Language 的简称,通用编程语言也就是我们非常熟悉的Java、Groovy、Python 以及 C 语言等。通用的计算机编程语言是可以用来编写任意计算机程序的，并且能表达任何的可被计算的逻辑，即图灵完备的

>**图灵机**（英语：Turing machine），又称确定型图灵机，是英国数学家艾伦·图灵于1936年提出的一种抽象计算模型，其更抽象的意义为一种数学逻辑机，可以看作等价于任何有限逻辑数学过程的终极强大逻辑机器。
>>图灵的基本思想是用机器来模拟人们用纸笔进行数学运算的过程，他把这样的过程看作下列两种简单的动作：
 - 在纸上写上或擦除某个符号；
 - 把注意力从纸的一个位置移动到另一个位置；

>**图灵完备**是对计算能力的描述。一门语言为什么要图灵完备呢？可以这么理解：一台计算机也是一个图灵机，一个图灵完备的语言意味着这个语言可以使用计算机完成任何计算机可以完成的任务，也就能够发挥计算机的所有能力。（这句话有点绕口）反之，一个图灵不完备的语言，就意味着不能发挥计算机的所有能力。[参考](https://www.zhihu.com/question/20115374)

而DSL并不是图灵完备的，它们的表达能力有限，只是在特定领域解决特定任务的，DSL 通过在表达能力上做的妥协换取在某一领域内的高效。

### DSL特点

一般书写DSL有如下特点：

- 没有计算和执行的概念
- 其本身并不需要直接表示计算
- 使用时只需要声明规则、事实以及某些元素之间的层级和关系

一门DSL语言会很小，很简单，很有表现力，聚焦于一个问题区域或领域，DSL本身有两大特点：**上下文驱动；非常流畅**。

#### 上下文驱动

上下文(Context)是DSL的特点之一，上下文也是人类赖以生存的基础，在人类的交流中，上下文提供的连续性功不可没。

一个订购Pizza的例子：

java中没有上下文的例子，需要每次都使用pizzaShop引用：

```java
                PizzaShop pizzaShop = new PizzaShop();
                pizzaShop.setSize(Size.LARGE);
                pizzaShop.setCrust(Crust.THIN);
                pizzaShop.setTopping("olives", "Onions", "Bell Pepper");
                pizzaShop.setAddress("101 ... Main St");
                int time = pizzaShop.setCard(CardType.VISA, "1013-5222-5366-4136");
                System.out.println(String.format("pizza will arrive in %d minutes\n", time));
```

在Groovy中因为有了with方法，而且括号几乎总是可选的。使用Groovy实现就没有那么乱了：

```groovy
                def pizzaShop = new PizzaShop()
                pizzaShop.with {
                        setSize              Size.LARGE
                        setCrust             Crust.THIN
                        setTopping         "olives", "Onions", "Bell Pepper"
                        setAddress         "101 ... Main St"
                        int time =            setCard CardType.VISA, "1013-5222-5366-4136"
                        printf                 "pizza will arrive in %d minutes\n", time

                }
```

上下文是一切变得紧凑，也减少了混乱，提升了实际效果。


#### 流畅性

流畅性是DSL的另一个特点，它改进了代码的可读性，同时使代码自然的流动，比如在Groovy中编写循环的方式

```groovy
                //传统的java方式
               for (int i = 0; i < 10; i++) {
                       System.out.println(i);
                }
                //groovy的方式
                for(i in 9){println i}
                0.upto(9){println it)
                10.times{println it}
```

Groovy生成器是DSL很好的例子。


### DSL的分类

DSL可以分为 外部DSL和内部DSL

- 外部DSL：要构建一种DSL，先定义它的语法，然后通过代码生成技术把DSL代码转成一种通用语言代码，或者写一个这种DSL的解释器。XML配置文件是外部DSL的另一种常见形式。
- 内部DSL：利用编程语言自带的语法结构定义出来的DSL，我称之为“内部DSL”，也叫做“内嵌DSL”。其语法受到现有语言的约束，不适用任何解析器，但是巧妙地将语法映射到底层语言中的方法和属性等构造


### 创建内部的DSL

动态语言很适合设计和实现内部的DSL，这个语言提供了很多的元编程能力和语法，便于很轻松的加载和执行代码片段，单并不是所有的动态语言都是这样的。

- ruby 创建DSL非常容易
- groovy 的动态类型和元编程能力对创建内部的DSL帮助很大，其对括号的处理没有ruby显得优雅
- python 创建DSL面临着一些挑战。

---
## 2 Groovy与DSL

Groovy的很多核心功能对创建内部的DSL很有帮助：

- 动态类型和可选类型
- 动态加载、操纵、执行脚本的灵活性
- 分类和ExpandoMetaClass
- 闭包为执行提供了很好的上下文
- 操纵符重载有助于自由的定义操作符
- 生成器支持
- 灵活的括号(使用一定技巧可以避免括号)


Groovy与DSL的内容主要包括：

1. 使用命令链式改进流畅性
2. 闭包与DSL(即闭包的delegate)
3. 方法拦截与DSL
4. 括号的限制与变通的方案
5. 分类与DSL
6. ExpandoMetaClass与DSL


### 使用命令链式改进流畅性

```groovy
class CommandChain {

    def move(dir) {
        println "moving $dir"
        this
    }
    def and(then){this}

    def turn(dir) {
        println "turning $dir"
        this
    }

    def jump(speed, dir) {
        println "jumping $speed and $dir"
        this
    }

}

def (forward, left, then, fast, right) = [
        'forward',
        'left',
        'then',
        'fast',
        'right'
]

cc = new CommandChain()
cc.with {
    move forward and then turn left
    jump fast,forward and then turn right
}

/*
moving forward
turning left
jumping fast and forward
turning right
 */
```

### 方法拦截与DSL

不用Pizza类也可以实现订购pizza,在orderPizza.dsl中定义订购pizza的逻辑:

**orderPizza.dsl**:
```groovy
size large
crust thin
topping Olives,Onions,Bell_Pepper
address "101 Main St ..."
card visa,'1233-2143-4331-323'
```

**GroovyPizzaDSL.groovy**
```groovy
def large = 'large'
def thin = 'thin'
def visa = 'visa'
def Olives = 'Olives'
def Onions = 'Onions'
def Bell_Pepper = "Bell_Pepper"

orderInfo = [:]//要在闭包中使用，不能使用def定义

def methodMissing(String name, args) {
    orderInfo[name] = args
}

def acceptOrder(closure) {
    println "accept order"
    closure.delegate = this
    closure.call()
    println "Validation and processing performed here for order received:"
    orderInfo.each {
        key, value ->
            println "$key -> ${value.join(' , ')}"
    }
}
```

```
//执行DSL
def dslDef = new File("GroovyPizzaDSL.groovy").text
def dsl = new File('orderPizza.dsl').text
def script = """
    ${dslDef}
       acceptOrder{
            ${dsl}
        }
"""

println script
new GroovyShell().evaluate(script)
```

### 括号的限制与变通的方案

```groovy
class Total1 {
    def value = 0

    def clear() {
        value = 0
    }

    def add(number) {
        value += number
    }

    def total() {
        println "total is $value"
    }
}

def calc1 = new Total1()
calc1.with {
    clear()
    add 3
    add 5
    add 5
    total()
}
```
上面clear()和 total()都需要括号，因为调用没有参数的方法时不加括号，Groovy会认为其是属性。

下面使用变通的方案，定义名称修改为getClear和getTotal，这样就可以通过clear和total来调用对应的方法了，这是之前学习过的Groovy的特性。

```groovy
class Total2 {
    def value = 0

    def getClear() {
        value = 0
    }

    def add(number) {
        value += number
    }

    def getTotal() {
        println "total is $value"
    }
}

def calc2 = new Total2()
calc2.with {
    clear
    add 3
    add 5
    add 5
    total
}
```


### 分类与DSL

使用分类可以以可控的方式拦截方法的调用，创建DSL也可以使用分类，比如实现如下流畅的调用：` 4.days.ago.at(4:43)`

```groovy
class DateUtils{
    static int getDays(Integer self) {
        self
    }
    static Calendar getAgo(Integer self) {
        def date = Calendar.instance
        date.add(Calendar.DAY_OF_MONTH, -self)
        date
    }
    static Date at(Calendar self, Map time){
        def hour =0
        def minute = 0
        time.each {
            key,value->
                hour = key.toInteger()
                minute = value.toInteger()
        }
        self.set(Calendar.HOUR_OF_DAY,hour)
        self.set(Calendar.MINUTE,minute)
        self.set(Calendar.SECOND,0)
        self.time
    }
}

use(DateUtils){
    println 4.days.ago.at(5:30)
}
```


### ExpandoMetaClass与DSL

```groovy
Integer.metaClass{
    getDays={
        delegate
    }
    getAgo = {
        def date = Calendar.instance
        date.add(Calendar.DAY_OF_MONTH, -delegate)
        date
    }
}

Calendar.metaClass.at={

    Map time ->

        def hour =0
        def minute = 0

        time.each {
            key,value->
                hour = key.toInteger()
                minute = value.toInteger()
        }
       delegate.set(Calendar.HOUR_OF_DAY,hour)
       delegate.set(Calendar.MINUTE,minute)
       delegate.set(Calendar.SECOND,0)
       delegate.time
}

println 4.days.ago.at(5:32)
```


