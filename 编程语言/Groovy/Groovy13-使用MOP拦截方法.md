# 第十二章：使用MOP拦截方法

在Groovy中可以轻松的实现AOP编程(aspect-oriented-programing)，比如有三种类型的建议：

- 前置建议：某个操作之前调用
- 后置建议：某个操作之后调用
- 环绕建议：代替某个操作

使用MOP可以轻松的实现这些建议的拦截器

---
## 1 使用GroovyInterceptable拦截方法

GroovyInterceptable拦截方法的规则上一章已经学习过了。**使用GroovyInterceptable拦截方法适用于拦截自己编写的类**

```groovy
class Car implements GroovyInterceptable {
    def check() {
        System.out.println("check called")//这里使用的时System.out.println，因为println时Groovy在Object上注入的方法，使用它会被拦截掉
    }

    def start() {
        System.out.println("start called")
    }

    def end() {
        System.out.println("end called")
    }

    @Override
    def invokeMethod(String name, Object args) {
        System.out.println("call $name....")
        if (name != "check") {
            System.out.println("running filter....")
            this.metaClass.getMetaMethod("check").invoke(this, null)
        }

        def validMethod = Car.metaClass.getMetaMethod(name, args)
        if (validMethod != null) {
            validMethod.invoke(this, args)
        } else {
            Car.metaClass.invokeMethod(this, name, args)
        }

    }
}

def car = new Car()
car.start()
car.end()
car.check()
try {
    car.speed()
} catch (e) {
    println e
}
```
打印结果:
```
call start....
running filter....
check called
start called
call end....
running filter....
check called
end called
call check....
check called
call speed....
running filter....
check called
groovy.lang.MissingMethodException: No signature of method: ztiany.chapter12.Car.speed() is applicable for argument types: () values: []
Possible solutions: sleep(long), sleep(long, groovy.lang.Closure), end(), split(groovy.lang.Closure), check(), start()
```

---
## 2 使用MetaClass拦截方法

**注意：对于没有实现GroovyInterceptable的类，如果它的MetaClass上实现了invokeMethod();不管方法是否存在，都会调用到该方法**

```groovy
//使用MetaClass拦截方法:

class Car1 {
    def check() {
        System.out.println("check called")
    }
    def start() {
        System.out.println("start called")
    }
    def end() {
        System.out.println("end called")
    }
}

Car1.metaClass.invokeMethod = {
    String name, args ->
        System.out.println("call $name....")
        if (name != "check") {
            System.out.println("running filter....")
            delegate.metaClass.getMetaMethod("check").invoke(delegate, null)
        }
        def validMethod = Car1.metaClass.getMetaMethod(name, args)
        if (validMethod != null) {
            validMethod.invoke(delegate, args)
        } else {
            //这里因为已经在invokeMethod中了，所以使用invokeMissingMethod，而不是递归的调用invokeMethod
            Car1.metaClass.invokeMissingMethod(delegate, name, args)
        }
}

def car1 = new Car1()
car1.start()
car1.end()
car1.check()
try {
    car1.speed()
} catch (e) {
    println e
}
```
打印结果：
```
call start....
running filter....
check called
start called
call end....
running filter....
check called
end called
call check....
check called
call speed....
running filter....
check called
groovy.lang.MissingMethodException: No signature of method: ztiany.chapter12.Car1.speed() is applicable for argument types: () values: []
Possible solutions: sleep(long), sleep(long, groovy.lang.Closure), end(), split(groovy.lang.Closure), check(), start()
```

如果感兴趣的只是拦截对不存在的方法的调用，那应该使用methodMissing来代替invokeMethod，可以在metaClass上同时提供invokeMethod和methodMissing，invokeMethod有优先于methodMissing处理，而调用metaClass的invokeMissingMethod其实就是调用methodMissing方法.

```groovy
Integer.metaClass.invokeMethod = {
    String name, args ->
        System.out.println("call $name on $delegate")

        def validMethod = Integer.metaClass.getMetaMethod(name, args)
        if (validMethod == null) {
            Integer.metaClass.invokeMissingMethod(delegate, name, args)//调用methodMissing
        } else {
            System.out.println(" running pre filter")
            resut = validMethod.invoke(delegate, args)
            System.out.println(" running post filter")
            resut
        }
}

Integer.metaClass.methodMissing = {
    String name, args ->
        System.out.println("methodMissing $delegate")
}

println 4.floatValue()
println 5.intValue()
try {
    21.method()
} catch (e1) {
    println e1
}
```
打印结果：
```
call floatValue on 4
 running pre filter
 running post filter
4.0
call intValue on 5
 running pre filter
 running post filter
5
call method on 21
methodMissing 21
```

### ExpandoMetaClass

ExpandoMetaClass是MetaClass的一个实现，也是Groovy中实现动态行为的关键类之一，默认情况下Groovy并没有使用ExpandoMetaClass

```groovy
println Integer.metaClass.getClass().name//结果：groovy.lang.ExpandoMetaClass

println String.metaClass.getClass().name//结果：org.codehaus.groovy.runtime.HandleMetaClass
String.metaClass.toString = {
    "non to string"
}
println String.metaClass.getClass().name//结果：groovy.lang.ExpandoMetaClass
```