# 第三章：动态类型

---
## 1 动态类型

动态类型语言中的类型是运行时推断的，方法和参数也是运行时检查的，通过这种能力，可以在运行时向类中注入行为。动态类型放宽了对类型的要求，使语言能够根据上下文判定类型，主要有以下两大优点：

1. 可以在不知道方法具体细节的情况下编写对象上的调用语句，在运行期间，对象会动态的响应方法或消息，在静态类型语言中，可以使用多态在某种程度上实现这种动态行为，然而大部分静态语言把继承和多态捆绑在一起，而真正的多态并不关注类型:把消息发送给一个对象，在运行期间，它会确定所要使用的相应实现
2. 不必用大量的强制类型转换操作来取悦编译器

java中类型代码
```java

class Engine implements Cloneable {

}

class Car implements Cloneable {

    private int mYear
    private Engine mEngine

    @Override
    protected Object clone() {
            try {
            Car cloneCar = (Car) super.clone()
            cloneCar.mEngine = (Engine) this.mEngine.clone()
            cloneCar
        } catch (CloneNotSupportedException e) {//不会发生这种情况，这只是为了取悦编译器
                return null
        }
    }
}
```

---
## 2 动态类型不等于弱类型

在运行时，Groovy会把错误的类型转换毫不含糊的提示出来，把实际的验证推迟到运行时，我们得以在编码和编译时以及代码执行时修改程序的结构，JVM上的动态类型语言说明，动态类型并不等于弱类型

-  强类型：Ruby/Groovy /Java/C#
-  弱类型：JavaScript/Perl/C/C++
-  动态：Ruby/Groovy/JavaScript/Perl
-  静态：Java/C#/C/C++


---
## 3 能力式设计

Java中的面向接口编程：基于接口编程尽管非常强大，但是往往有很多限制(至少需要实现接口，有的程序员甚至刚开始没有设计接口)

**能力式设计**：利用了对象的能力——依赖一个隐式的接口(只要对象能响应这个方法，我就可以调用这个方法，而实现并不需要确定对象有这个方法)，这种称为**鸭子模型**：如果它走起来像鸭子，叫起来也像鸭子，那么它就是鸭子
想要这种能力只需要实现该方法，而不需要实现或扩展任何东西，这样的接口就是少了许多繁文缛节，增加了生产效率

> 因为来自java的背景，可能需要付出更多的努力才能习惯groovy的动态特性，但是一旦感觉对了，就可以好好利用了。

实例：搬动一些重物，需要找一个愿意帮忙且有能力帮忙的人

**在Java中有这样的演变**：
```java
//第一次：男人搬东西
class Man {
    void moveThings() {
        println 'man move things'
    }
}

public void takeHelp(Man man) {
    man.moveThings()
}

//第二次：但是如果有一个女人也可以帮忙了呢。需要把moveThings设计为抽象的能力
abstract class Human {
    abstract void moveThings()
}
class Woman extends Human{
    @Override
    void moveThings() {

    }
}
//第三次：但是如果大象也可以帮忙呢？最后还是选择了把moveThing抽象到接口中(事实上确实应该如此，这本来就只是一种能力，也许这并不是一个恰当的例子)
interface Helper{
    void moveThings()
}
//至此扩展需要付出很多的努力。要使用多种多样的对象，需要创建接口，并修改依赖接口的代码
```
**看看Groovy的动态类型能力如果应对上面的例子**：
```groovy
def takeHelpGroovy(helper) {//helper可以是任何对象，只要它有moveThings方法
    helper.moveThings()
}
```
takeHelpGroovy方法接受一个helper，并且没有指定其类型，这样它的默认类型就是Object，然后调用了其moveThings方法，这就是能力是设计，我们利用了对象的能力——依赖一个隐式的接口，这种称为鸭子模型：如果它走起来像鸭子，叫起来也像鸭子，那么它就是鸭子,想要这种能力只需要实现该方法，而不需要实现或扩展任何东西，这样的接口就是少了许多繁文缛节，增加了生产效率


---
## 4 使用动态类型需要自律

当我们看到了动态类型的诸多优化与灵活性时，也应该看到其风险

-  在没有创建依赖的类时，可能会敲错名字：**依赖单元测试，使用动态类型语言而没有做单元测试就像是在玩火**
-  没有类型信息，怎么知道给方法什么呢？：**所以需要良好的命名约定**
-  如果把方法发给一个不能提供搬动重物的事物(没有对应方法)，又会怎样：**当对象不支持预期的方法时，Groovy会抛出异常，同时可以使用Groovy提供的respondTo方法来判断对象是否有对应的能力(方法)**

```groovy

class RespondToTest{
    String doSomething(String str) {
        str.toUpperCase()
    }
}

def rt = new RespondToTest()
println rt.metaClass.respondsTo(rt, 'doSomething')

/*打印结果为：[public java.lang.String ztiany.chapter3.RespondToTest.doSomthing(java.lang.String)] 分别是返回值，方法，参数*/

if (rt.metaClass.respondsTo(rt, 'doSomething')) {
    println rt.doSomething("dd")
}
```

---
## 5 可选类型

Groovy是可选类型的，这意味着可以不指定任何类型让Groovy来确定，也可以将精确的指定所要使用的变量或引用的类型
需要注意的是:

**Groovy默认并不做完整的类型检查，当我们写下`X obj = 2`时，Groovy所做的只是简单的做一个强转，如：`X obj = (X)2;`**

---
## 6 多方法

动态类型和动态语言改变了对象响应方法调用的方式:
Groovy会聪明的选择正确的实现，不仅基于目标参数，还基于所提供的参数的实际类型，因为方法分派基于多个实体：目标加参数，所以这被称做**多分派或者多方法**

```groovy
class Employee {
    public void raise(Number number) {
        System.out.println("Employee raise")
    }
}

class Executive extends Employee {

    void raise(Number number) {
        System.out.println("Executive raise Number")
    }

    void raise(BigDecimal bigDecimal) {
        System.out.println("Executive raise BigDecimal")
    }
}


def runRaise(Employee employee) {
    //如果是在Java中：运行时方法必须接受一个Number作为参数，这是由基类Employee规定的，
    //这里编译器会把BigDecimal看作Number
    employee.raise(new BigDecimal(32))
}


/*此段代码如果是在Java中运行的话，则打印结果应该是  System.out.println("Executive raise Number"), 而此在Groovy中则是System.out.println("Executive raise BigDecimal")*/

runRaise(new Employee())
runRaise(new Executive())
```

**集合的例子**
```groovy

public class UsingCollection {
    public static void main(String... args) {
        ArrayList<String> list = new ArrayList<>();
        Collection<String> col = list;
        list.add("one")
        list.add("two")
        list.add("three")
        list.remove(0);
        //Java的情况：因为这里的引用是Collection，没有remove(index)的方法，所以这里调用的是remove(Object)方法
        //Groovy的情况：这里调用的是List方法，Groovy的动态与多方法能力很好的处理这种情况
        col.remove(0);
        println(list.size());
        println(col.size());
    }
}
```

---
## 7 关闭动态类型

Groovy的元编程能力，都依赖于其动态类型，但动态类型也是有代价的，原本编译时期可以发现的问题被推迟到运行时，然后动态方法分配机制也是有开销的，

可以让Groovy编译器将其类型检查从动态的不严格模式收紧到我们对静态类型编译器所期待的水平，还可以权衡动态类型和元编程能力的收益，让Groovy编译器静态编译代码，以便获取更高效的字节码。

### 静态类型检查

`@TypeChecked`注解在编译时期会验证方法或者属性是或属于某一个类，这会阻止我们使用元编程能力。

静态类型会**限制使用动态方法**，然而他并没有阻止使用Groovy行JDK中的类添加方法，静态类型检查器会检查这些类中的方法和属性，并且还会检查Groovy添加的方法，这包括：

-  一个特殊的DefaultGroovyMethodsSupport类及其扩展:
    -  NioGroovyMethods
    -  IOGroovyMethods
    -  SocketGroovyMethods
    -  PluginDefaultGroovyMethods
    -  **DefaultGroovyMethods**
    -  DateGroovyMethods
    -  ProcessGroovyMethods
    -  ResourceGroovyMethods
    -  StringGroovyMethods
-  此外它还会检查开发者能够添加的定制扩展(后面将会学习)

比如：
- DefaultGroovyMethods中的方法`println`
- 开发者能够添加的定制扩展 ,例如`str.reverse()`,来自StringGroovyMethods

```groovy

@TypeChecked
def shout(String str) {
    println 'printing in uppercase'
    println str.toUpperCase()
    println 'printing again uppercase'
    println str.toUpperCase()//这个方法没有使用任何元编程，因此可以利用编译时验证
    str.reverse()
}


//instanceof不需要强转
@TypeChecked
def use(Object o) {
    if (o instanceof String) {
        o.length()
    } else {
        0
    }
}

//TypeChecked忽略一些方法检查
@TypeChecked
class Sample {

    def method() {
    }

    //DateGroovyMethods中包含toCalendar方法，所以可以编译通过
    def dateTest(Date date) {
        date.toCalendar()
    }

    @TypeChecked(TypeCheckingMode.SKIP)//使用此注解，则会跳过静态类型检查
    def method2(String str) {
        str.shot()
    }
}
```

### 静态编译

Groovy的动态类型和元编程的优点显而易见，但是这些都要以性能作为代价，性能的下降与代码、所调试的方法的个数等因素有关，当不需要元编程和动态能力时，与等价的Java代码相比，性能损失了10%，我们可以关闭动态类型，阻止元编程，放弃多方法，并让Groovy生成性能足以和
Java媲美的高效字

```groovy
@CompileStatic
def shot1(String str) {
    println str.toUpperCase()
}
```

