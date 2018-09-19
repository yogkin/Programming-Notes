# 第十三章：MOP方法注入


java程序员常常抱怨：要是String支持`encrypt`方法就方便了。面向对象本就是针对程序的可扩展性，但具体能扩展到什么程度往往要受到语言本身的限制。

而Groovy可以做到打开任意类，根据需要加入想要的方法。根据所接收到的特定输入添加一个方法，能够修改类的行为，是元编程和Groovy元对象的核心。

Groovy的MOP支持以下三种技术行为中的任一种：

-  分类(category)
-  ExpandoMetaClass
-  Minxin(混入)

---
## 1 使用分类注入方法

Groovy的分类提供了一种可控的注入方式--**方法的注入的作用可以限定在一个代码块内**，分类是一种能够修改类的metaClass对象，而且这种修改仅在代码块的作用域和执行线程内有效。

分类也有一些限制，起作用限定在`use`代码块内，所以也限定于执行线程，因此注入的方法也是有限制的，多次的进入和退出use也是有代价的，因此如果调用不太频繁，而且需要分类提供的可控方法注入，则可以选择分类进行方法注入。

```groovy
class StringUtils {
    def static toSSN(self) {//如果只希望为String提供toSSN方法则声明此方法的参数类型为String
        if (self.size() == 9) {
            "${self[0..2]}--${self[3..4]}--${self[5..8]}"
        }
    }
}
//使用use注入方法,use是DefaultGroovyMethods的方法,Groovy脚本把use方法路由到了GroovyCategorySupport类的use方法。
use(StringUtils) {
    println "123456789".toSSN()
    println new StringBuffer("987654321").toSSN()
}
try {
    "123456789".toSSN()
} catch (e) {
    println e
}
```
**使用Category注入方法**：
```groovy
@Category(String )//指定为String提供方法注入，如果想要为所有的类提共此方法注入，则声明为Object
class AnnotatedStringUtils {

    def toSSN() {//如果只希望为String提供toSSN方法则声明此方法的参数类型为String
        if (this.size() == 9) {
            "${this[0..2]}--${this[3..4]}--${this[5..8]}"
        }
    }
}
use(AnnotatedStringUtils){
    println "963258741".toSSN()
}
```
**传入闭包**:
```groovy
class FindUtils {
    def static extractOnly(String self, closure) {
        def result = ''
        self.each {
            if (closure.call(it)) {
                result += it
            }
        }
        result
    }
}

use(FindUtils) {
    println "123698547".extractOnly {
        it == '4' || it == '5'
    }
}
```
**传入多个类**：
```groovy
use(StringUtils, FindUtils) {
    println "123698547".extractOnly {
        it == '5' || it == '0'
    }
    println "123698547".toSSN()
}
```
如果多个类中存在重复的方法，则参数列表中最后一个类提供的方法优先级最高,use还可以嵌套使用，即use中使用use，此时内部的分类优先级较高。

**拦截方法**：

```groovy
class Helper {
    def static toString(String self) {
        //这里的toString用于拦截self的toString，所以这里调用metaClass的toString方法
        def method = self.metaClass.metaMethods.find { it.name == 'toString' }
        '!!' + method.invoke(self, null) + '!!'
    }
}

use(Helper) {
    println "123".toString()
}
//使用分类拦截方法并不优雅，因此分类应该用于注入方法
```


### 内置分类内置分类：
Groovy提供了很多方法我们使用的分类，比如DOMCategory用于处理XML，支持了GPath
- groovy.time.TimeCategory
- groovy.servlet.ServletCategory
- groovy.xml.dom.DOMCategory


---
## 2 使用ExpandoMetaClass注入方法

要创建领域特定语言(DSL)，需要能够向不同的类、甚至类的层次结构中加入任意方法，为了保持一定的流畅性，我们需要注入实例方法和静态方法，操纵构造器，以及将实例方法转换为属性。

**使用ExpandoMetaClass注入方法也有一些限制，注入的方法只能从Groovy内调用，不能从编译的java代码中调用**

**注入类方法**:

```groovy
//给Integer注入daysFromNow方法
Integer.metaClass.daysFromNow = {
    Calendar today = Calendar.getInstance()
    today.add(Calendar.DAY_OF_MONTH, delegate)
    today.time
}
println 4.daysFromNow()

//小窍门 在注入的方法前面加入get，在以后的方法中可以以属性的方法访问
Integer.metaClass.getDaysFromNow = {
    ->
    Calendar today = Calendar.getInstance()
    today.add(Calendar.DAY_OF_MONTH, (int) delegate)
    today.time
}
println 6.daysFromNow
```
**给多个类注入方法的方式**:
```groovy
getPrettyName = {
    ->
    "prettyName---" + delegate
}
Integer.metaClass.getPrettyName = getPrettyName
Long.metaClass.getPrettyName = getPrettyName

println 4.prettyName
println 6L.prettyName

//或者在接口上注入方法，则所有实现者都拥有此方法
Number.metaClass.getPrettyName = getPrettyName
println 1000000000000L.prettyName
```
**注入静态方法**:
```groovy
Integer.metaClass.'static'.isEven = {
    val ->
        val % 2 == 0

}

println Integer.isEven(5)
println Integer.isEven(6)
```

**注入构造器**:
```groovy
//可以使用 << 来为一个类添加构造器，使用 << 来覆盖现有的构造器或者方法会导致异常
Integer.metaClass.constructor << {
    Calendar calendar ->
        new Integer(calendar.get(Calendar.DAY_OF_YEAR))
}
//如果想要替换(覆盖)一个原来的构造器可以使用=操作符


Integer.metaClass.constructor = {
    int val ->
        println '拦截了构造方法'
        //在覆盖的构造方法内仍可以使用反射来调用原来的实现
        constructor = Integer.class.getConstructor(Integer.TYPE)
        constructor.newInstance(val)
}

println new Integer(4)
println new Integer(Calendar.getInstance())
```
**注入一组方法**:
```groovy
String.metaClass {

    parseInteger = { ->
        Integer.parseInt(delegate)
    }
    //静态方法
    'static' {
        isShorString = {
            String str ->
                str.length() < 4
        }
    }

    constructor = {
        String str ->
            println "拦截了String的构造方法"
            constructor = String.class.getConstructor(String.class)
            constructor.newInstance(str)
    }
}

println '213'.parseInteger()
println String.isShorString("3442323")
println new String("aaaBBB")
```
**向具体的示例中注入方法**:

每一个实例都有一个MetaClass，向实例的MetaClass注入方法，可以防止注入的方法应用到类上.
```groovy
class Person {
    def play() {
        println "playing"
    }
}
def emc = new ExpandoMetaClass(Person)
emc.sing = {
    "sing... ob baby"
}
emc.initialize()
def jack = new Person()
def paul = new Person()

jack.metaClass = emc

println jack.sing()

try {
    println paul.sing()
} catch (e) {
    println  e
}

//还有更简单的方式

def jams = new Person()
jams.metaClass.sing = {
    "jams sing ob baby"
}
println jams.sing()

//也可以给对象注入一组方法
def jordan = new Person()
jordan.metaClass {
    playBasketball = {
        ->
        "飞人乔丹灌篮"
    }

    playBasetball = {
        ->
        "飞人乔丹打棒球"
    }
}

println jordan.playBasketball()
println jordan.playBasetball()

```

---
## 3 使用Mixin注入方法

Groovy的Mixin(混入、混合)是一种运行时能力，可以将多个类的实现引入进来或者混入，如果将一个类混入另一个类，Groovy会在内存中把这些类的实例连接起来，当调用一个实例的方法时，将会先路由到混入的类中，如果该方法存在混入的类中，则在混入的类中处理，否则由主类处理，可以将多个类混入到一个类中，最后加入的Mixin优先级最高。
```groovy
class Friend {
    def listen() {
        "$name listening as a friend"
    }
}

/*
使用@Mixin注解语法注入方法:
将Mixin注入到APerson中，Mixin注解会将作为参数的类中的方法添加到被注解的类中，
Mixin的方式非常优雅，但是如果也有一定的限制，这种方式只能由类的作者使用
 */

@Mixin(Friend)
class APerson {
    String firstName
    String lastName

    String getName() {
        "$firstName $lastName"
    }
}

def john = new APerson(firstName: "Zhan",lastName: 'Tianyou')
println john.listen()//Zhan Tianyou listening as a friend

//另外一种方式是使用静态代码块注入mixin
class BPerson{
    static {
        mixin Friend
    }

    String getName() {
        "BPerson"
    }
}

println new BPerson().listen()//BPerson listening as a friend

```
**动态的混入方法**:
```groovy
//向Dog中混入方法
class Dog{
    String name;
}
Dog.mixin Friend//调用Dog类的mixin方法，混入Friend
//现在Dog以及拥有了Listen方法了
println  new Dog(name: "Buddy").listen()//Buddy listening as a friend

//也可向特定类的实例中混入方法，这样混入的方法只在该实例上：
class Cat {
    String name
}

try {
        new Cat(name: "Rude").listen()// No signature of method: ztiany.chapter13.Cat.listen()
} catch (e) {
    println e
}
tom = new Cat(name: "tom")
tom.metaClass.mixin Friend
println tom.listen()//tom listening as a friend
```

**在类中使用多个Mixin**:

当混入多个类时，所有这些类的方法在目标类中都是可用的，默认情况下多个方法会因为冲突而被隐藏，
也就是说，当作为Mixin的两个或者多个存在相同名字、参数签名也相同时，最后加入到Mixin的方法优先级较高。

通过编程实现，以方法调用链的方式将这些方法连接起来，反而可以让他们合作,注意：这里的实现调用链是核心！
```groovy
//一般应用需要不同类型的输出目标，可能是文件、套接字、web服务等，将之抽象为Writer类
abstract class Writer {
    abstract void write(String message)
}
//StringWriter是这个类的一个具体实现，在write方法中将给定点的消息写入到一个StringBuilder
@ToString
class StringWriter extends Writer {
    def target = new StringBuilder()

    @Override
    void write(String message) {
        target.append(message)
    }
}

def writeStuff(writer) {
    writer.write("this is stupid")
    println writer
}

/*
 *  给theWriter混入filters
 */

def create(theWriter, Object[] filters = []) {
    def instance = theWriter.newInstance()
    filters.each { filter ->
        instance.metaClass.mixin filter
    }
    instance
}

writeStuff(create(StringWriter))

//一个过滤器
class UppercaseFilter {
    void write(String message) {
        def allUpper = message.toUpperCase()
        invokeOnPreviousMixin(metaClass, 'write', allUpper)
    }
}

Object.metaClass.invokeOnPreviousMixin = {


    MetaClass currentMixinMetaClass, String method, Object[] args ->

        println "currentMixinMetaClass $currentMixinMetaClass.delegate.theClass"

        def previousMixin = delegate.getClass()//这里的delegate是目标实例，即Filter的某个实例


        //mixedIn属性：用于为实例保存有序的mixin实例
        for (mixin in mixedIn.mixinClasses) {//mixedIn.mixinClasses 遍历所有的mixin

            //mixin的类型为：MixinInMetaClass
            //MixinInMetaClass.mixinClass：CachedClass
            //CachedClass.theClass：返回原来的Class：原来的class即Filter对象的Class

            if (mixin.mixinClass.theClass == currentMixinMetaClass.delegate.theClass) {
//当找到传入参数的mixin时跳出
                break
            }
            previousMixin = mixin.mixinClass.theClass
        }

        println "mixedIn[previousMixin] ${mixedIn[previousMixin]}"//ExpandoMetaClass$MixedInAccessor

        mixedIn[previousMixin]."$method"(*args)//这里的*args 为展开操作符
}

writeStuff(create(StringWriter, UppercaseFilter))

//过滤 stupid的Filter

class ProfanityFilter {
    void write(String message) {
        def filtered = message.replaceAll('stupid', 's******')
        invokeOnPreviousMixin(metaClass, 'write', filtered)
    }
}

writeStuff(create(StringWriter, ProfanityFilter))
writeStuff(create(StringWriter, UppercaseFilter, ProfanityFilter))//这里混入的顺序不一样，所以结果也不一样
writeStuff(create(StringWriter, ProfanityFilter, UppercaseFilter))
```
打印结果：
```
ztiany.chapter13.StringWriter(this is stupid)
currentMixinMetaClass class ztiany.chapter13.UppercaseFilter
mixedIn[previousMixin] groovy.lang.ExpandoMetaClass$MixedInAccessor$1@b3ca52e
ztiany.chapter13.StringWriter(THIS IS STUPID)
currentMixinMetaClass class ztiany.chapter13.ProfanityFilter
mixedIn[previousMixin] groovy.lang.ExpandoMetaClass$MixedInAccessor$1@7770f470
ztiany.chapter13.StringWriter(this is s******)
currentMixinMetaClass class ztiany.chapter13.ProfanityFilter
mixedIn[previousMixin] groovy.lang.ExpandoMetaClass$MixedInAccessor$2@2df3b89c
currentMixinMetaClass class ztiany.chapter13.UppercaseFilter
mixedIn[previousMixin] groovy.lang.ExpandoMetaClass$MixedInAccessor$1@23348b5d
ztiany.chapter13.StringWriter(THIS IS S******)
currentMixinMetaClass class ztiany.chapter13.UppercaseFilter
mixedIn[previousMixin] groovy.lang.ExpandoMetaClass$MixedInAccessor$2@6200f9cb
currentMixinMetaClass class ztiany.chapter13.ProfanityFilter
mixedIn[previousMixin] groovy.lang.ExpandoMetaClass$MixedInAccessor$1@2002fc1d
ztiany.chapter13.StringWriter(THIS IS STUPID)
```