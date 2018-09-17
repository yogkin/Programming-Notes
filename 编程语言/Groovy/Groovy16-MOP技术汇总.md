# 第十五章：MOP技术汇总

---
## 1 使用Expando创建动态的类

Groovy完全可以在运行时创建一个类，假设需要构建一个用于配置设备的应用，而对于设备一无所知，只知道它们的某些属性和配置脚本，
那就无法在编写代码时奢侈的为每个设备创建一个类，因此需要在运行时合成与这些设备打交道并完成配置的类。

Groovy的Expando类提供了动态合成类的能力。

```groovy
//就这么随意的创建了carA和carB
carA = new Expando()
carB = new Expando(year:2012, miles:0)
carA.year= 2012
carA.miles = 10
println carA.dump()
println carB.dump()
//不仅可以定义属性，还可以定义行为：

carC = new Expando(year:2012, miles:0, trun:{ println 'turning...'})
carC.drive = {
    miles += 10
    println "$miles miles driven"
}

carC.drive()
carC.trun()
```
假设有一个文件，其中保存了汽车用的数据，如下所示：
```
miles,year,make
42535,2003,Acura
43652,2006,Chevy
13655,2004,Honda
```
无需显示创建一个Car类，解析其文件即可:
```
data = new File("CarInfo").readLines()
props = data[0].split(",")
data -= data[0]

def averageMilesDrivenPerYear ={
    mile.toLong() / (2008 - year.toLong() )
}

cars = data.collect{
    car = new Expando()
    it.split(",").eachWithIndex { String entry, int i ->
        car[props[i]] = entry
    }
    car.ampy = averageMilesDrivenPerYear
    car
}

props.each {
    name->
        print " $name "
}
println " Avg.MPY "
ampyMethod = 'ampy'
cars.each { car ->
    for (String property : props) {
        print " ${car[property]} "
    }
    println car."$ampyMethod"()
}
```

---
## 2 方法委托：汇总练习

继承用来扩展一个类的行为，而委托依赖包含或者聚合的对象可以提供一个类的行为，**如果希望一个类替换另一个类，应该是用继承**，如果只是想简单的使用
一个对象，则应该选择委托，请将基础保留给is-a或者kind-of关系，大多数情况下应该使用委托。

java中的继承非常简单，使用extends关键字即可继承一个类，而实现委托则需要很多的路由方法。而Groovy可以简单的实现委托。

**繁复的实习委托**：

```groovy
class Worker {
    def simpleWork1(spec) {
        println "Worker does work1 with space $spec"
    }

    def simpleWork2(spec) {
        println "Worker does work2 with space $spec"
    }
}

class Expert {
    def advanceWork1(spec) {
        println "Expert does work1 with space $spec"
    }

    def advanceWork2(scope , spec) {
        println "Expert does work2 with scope $scope space $spec"
    }
}

class Manager {
    def worker = new Worker()
    def expert = new Expert()

    def schedule() {
        println "scheduling ......"
    }

    def methodMissing(String name, args) {
        println "intercept call $name"
        def delegateTo = null
        if (name.startsWith('simple')) {
            delegateTo = worker
        }
        if (name.startsWith('advance')) {
            delegateTo = expert
        }
        print "delegateTO $delegateTo   "
        def to = delegateTo?.metaClass?.respondsTo(delegateTo, name, args)
        println "to $to"
        if (to) {
            Manager instance = this
            instance.metaClass."$name" = {
                Object[] vars ->
                    delegateTo.invokeMethod(name, args)
            }
            this."$name"(args)
        } else {
            throw new MissingMethodException(name, Manager.class, args)
        }
    }
}

def peter = new Manager()
peter.schedule()
peter.simpleWork1("fast")
peter.simpleWork2("quality")
peter.advanceWork1("prototype ")
peter.advanceWork2('product',"quality")
```

**Groovy的方式**:
```groovy
class  Manager2{
    {//这个一个代码块，调用了delegateClassTo方法
        delegateClassTo Worker,Expert,GregorianCalendar
    }

    def schedule() {
        println "scheduling ..."
    }
}

这段代码轻松的实现了方法委托，实现了上面delegateClassTo方法。当调用到为实现的方法时就会被路由到methodMissing上，然后会根据delegateClassTo传入的Class对象，进行实例化，查看Class的实例中是否有能处理该方法的实例。

//给所有对象创建一个delegateClassTo方法，所以上面的Manager2也就有了delegateClassTo方法。
Object.metaClass.delegateClassTo={
    Class...classes->
        def objectOfDelegates = classes.collect{
            it.newInstance()
        }
        //注意这里的闭包指向的是实际调用delegateClassTo方法的对象。下面对说明
        delegate.metaClass.methodMissing={
            String name,args->
                println "intercept called $name"
                def delegateTo = objectOfDelegates.find{
                    it.metaClass.respondsTo(it,name,args)
                }
                if (delegateTo) {
                    delegate.metaClass."$name"={
                        java.lang.Object[] varArgs->
                            delegateTo.invokeMethod(name,args)
                    }
                    delegate."$name"(args)
                }else {
                    throw new MissingMethodException(name, delegate.getClass(), args)
                }
        }
}


def manager2 = new Manager2()
manager2.schedule()
manager2.simpleWork1("fast")
manager2.simpleWork2("quality")
manager2.advanceWork1("prototype ")
manager2.advanceWork2('product','quality')
```

上面说到"**闭包指向的是实际调用delegateClassTo方法的对象**。",这里举实例验证一下：
```
def closure = {
    int a->println delegate
}
closure(2)

Object.class.metaClass.testA={
    int a->
        println(delegate)
}

"abc".testA(2)
1231.testA(1)

/*
打印结果为：
ztiany.chapter4.TestClosure@6646153
abc//说明闭包的delegate被设置程abc
1231//说明闭包的delegate被设置程1231
*/
```

---
## 3 MOP技术回顾

### 3.1 用于方法拦截的选项

- 如果有权修改原代码，则可以使用GroovyInterceptable，只需实现invokeMethod方法
- 如果无法修改代码，或者是Java类，则可以使用ExpandoMetaClass或者分类，使用分类必须使用use代码块

### 3.2 用于方法注入的选项

使用分类或者ExpandoMetaClass来实现

- 分类必须使用use代码块，有所限制
- 想注入静态方法和构造器，ExpandoMetaClass是更好的选择，注意ExpandoMetaClass不是默认的metaClass的实现

### 3.3 用于方法合成的选项

- 如果有权修改代码，在原代码上使用methodMissing方法通过第一次调用时注入方法来改进性能。
- 如果无法修改代码，可以将methodMissing添加到ExpandoMetaClass中

