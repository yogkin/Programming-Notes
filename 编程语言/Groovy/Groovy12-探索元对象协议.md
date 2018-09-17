#  第十一章：探索元对象协议

Groovy的元编程提供了在运行时修改对象的类型或者让其动态的获得行为的能力，不再局限于java的静态结构，我们可以基于应用所接受的输入，**动态的添加方法和行为**。

**所谓元编程(metaPrograming)就是**：能够编程操作程序的程序，包括操作程序自身。

Groovy这样的动态语言通过**元对象协议(MetaObject Protocol)**提供这种能力，Groovy的元编程核心是MetaClass。

---
## 1 Groovy对象

Groovy对象是带有附加功能的Java对象。

Groovy应用中对象分为三类：

- POJO 使用Java或其他JVM语言编写的创建的
- POGO 用Groovy编写的对象
- Groovy拦截器(实现了GroovyInterceptable)的Groovy对象


所有的Groovy类都扩展了GroovyObject接口
```groovy
        public interface GroovyObject {
                Object invokeMethod(String name, Object args);
                Object getProperty(String propertyName);
                void setProperty(String propertyName, Object newValue);
                MetaClass getMetaClass();
                void setMetaClass(MetaClass metaClass);
        }
```

`invokeMethod/getProperty/setProperty` 使Java对象有了高度的动态性
`getMetaClass/setMetaClass `使创建代理拦截POGO上的方法调用，在POGO上注入方法变得容易

GroovyInterceptable继承了GroovyObject，是一个**标记类型的接口**，对于实现了该接口的方法，其上所有的方法调用(不管方法是否存在)都会被invokMethod方法拦截.。


Groovy支持对POJO和POGO进行元编程：

- 对应POJO，Groovy维护了MetaClass的一个MetaClassRegistry。
- 对于POGO，都有一个到其MetaClass的直接引用，当调用一个方法时，Groovy会检查目标对象是POJO还是POGO

>Class对象上的metaClass是该对象及其子类公共的，但是如果某个Class的实例拥有方法的实现时，优先调用实例上MetaClass的方法。

对于不同对象的类型，Groovy处理方法并不一样：

1. POJO：Groovy会去应用类的MetaClassRegistry（注册表）取它的metaClass，并将方法委托给它，因此在他的metaClass上定义的任何拦截器或者方法，**都要优先于POJO原来的方法**。
2. POGO：如果对象没有实现GroovyIntercepable：
    -  1    先查找他的metaClass中的方法，有则调用
    -  2    没有查找他的metaClass中的方法则查找自身的方法，有则调用
    -  3    没有查找自身的方法到则以方法名查找属性字段，如果属性字段是Closure，Groovy会调用它
    -  4    没有找到则查看自身是有一个methodMissing的方法，有则调用，没有就会调用POGO的invokeMethod(),如果POGO实现invokeMethod()方法则调用，默认invokeMethod(),抛出MissingMethodException
3. 如果对象实现了GroovyIntercepable：调用POGO的invokeMethod()方法



上面2.4点注意：`methodMissing`必须定义为如下形式：

```
   class A {
    def methodMissing(String name, Object args) {
        println "$name + $args"
    }
}
```

**由此可见，方法调用时，metaClass的优先级高于对象本身。**：

示例：

```groovy
//--------------------
//测试Groovy对方法调用的处理
//--------------------

testInterceptedMethodCallOnPOJO()
testInerrceptable()
testInterceptedExistingMethodCalled()
testUnInerceptedExistingMethodCalled()
testPropertyThatIsClosure()
testMethodMissingCalledOnlyForNoExisting()
tetInvokedMethodCalledForOnlyNonExistent()

void testInterceptedMethodCallOnPOJO() {
    println '------testInterceptedMethodCallOnPOJO-------'
    def val = new Integer(3)
    Integer.metaClass.toString = {
        -> "intercepted"
    }
    println val.toString()
}

void testInerrceptable() {
    println '------testInerrceptable-------'
    def obj = new AnInterceptable()
    println obj.existingMethod()
    println obj.nonExistMethod()

}


void testInterceptedExistingMethodCalled() {
    println '------testInterceptedExistingMethodCalled-------'
    AGroovyObject.metaClass.existingMethod2 = {
        ->
        "intercepted"
    }
    AGroovyObject aClass = new AGroovyObject()
    println aClass.existingMethod2()
}


void testUnInerceptedExistingMethodCalled() {
    println '------testUnInerceptedExistingMethodCalled-------'
    def obj = new AGroovyObject()
    println obj.existingMethod()
}

void testPropertyThatIsClosure() {
    println '------testPropertyThatIsClosure-------'
    def obj = new AGroovyObject()
    println obj.closureProp()
}


void testMethodMissingCalledOnlyForNoExisting() {
    println '------testMethodMissingCalledOnlyForNoExisting-------'
    def obj = new ClassWithInvokeAndMissingMethod()
    println obj.existingMethod()
    println obj.nonExistMehtod()
}

void tetInvokedMethodCalledForOnlyNonExistent() {
    println '------tetInvokedMethodCalledForOnlyNonExistent-------'
    def obj = new ClassWithInvokeOnly()
    println obj.existingMethod()
    println obj.nonExistMehtod()
}

//--------------------
//Class
//--------------------

class AnInterceptable implements GroovyInterceptable {
    def existingMethod() {
        "existingMethod"
    }
    def invokeMethod(String name, args) {
        "Interceptable "
    }

}

class AGroovyObject {
    def existingMethod() {
        "existingMethod"
    }
    def existingMethod2() {
        'existingMethod2'
    }
    def closureProp = {
        'closureProp called '
    }

}

class ClassWithInvokeAndMissingMethod {
    def existingMethod() {
        "existingMethod"
    }
    def invokeMethod(String name, args) {
        "invokeMethod called"
    }
    def methodMissing(String name, args) {
        'methodMissing called'
    }
}

class ClassWithInvokeOnly {
    def existingMethod() {
        "existingMethod"
    }
    def invokeMethod(String name, args) {
        'invokeMethod called'
    }
}
```
打印结果为：

```
------testInterceptedMethodCallOnPOJO-------
intercepted
------testInerrceptable-------
Interceptable
Interceptable
------testInterceptedExistingMethodCalled-------
intercepted
------testUnInerceptedExistingMethodCalled-------
existingMethod
------testPropertyThatIsClosure-------
closureProp called
------testMethodMissingCalledOnlyForNoExisting-------
existingMethod
methodMissing called
------tetInvokedMethodCalledForOnlyNonExistent-------
existingMethod
invokeMethod called
```

---
## 2 查询方法和属性

```groovy
str = "hello"
methodName = "toUpperCase"
mehtodOfInterest = str.metaClass.getMetaMethod(methodName)
println mehtodOfInterest.invoke(str)
```
打印结果：`HELLO`


---
## 3 动态的访问对象

- 动态的访问属性
- 动态的访问方法


```groovy
def printInfo(obj) {

    usrRequestedProperty = 'bytes'
    usrRequestedMethod = 'toUpperCase'
    //动态访问属性的方式
    println obj[usrRequestedProperty]
    println obj."$usrRequestedProperty"
    //动态调用方法的方式
    println obj."$usrRequestedMethod"()
    println obj.invokeMethod(usrRequestedMethod, null)
}

//访问对象的所有属性
"hello".properties.each {
    println it
}
```