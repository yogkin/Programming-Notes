# 第十四章：MOP方法合成

添加方法的行为可以分为两类：注入和合成


- **方法注入**(method inject)表示：编写代码时知道要添加到一个类或者多个类中的方法的名字，利用方法注入可以动态的向类中添加行为。
- **方法合成**(method synthesis)表示：想在调用的时候动态的确认方法的行为。Groovy的invokeMethod、methodMissing、GroovyInterceptable对于方法的合成很有帮助。

合成的方法直到调用时才会作为独立的方法存在。

---
## 1 使用methodMissing合成方法
```groovy
class Person {

    def work() {
        "working ... ..."
    }

    def plays = ['Tennis', 'Volleyball', 'Basketball'];

    def methodMissing(String name, args) {
        System.out.println("methodMissing called for $name")
        def methodInList = plays.find { it == name.split('play')[1] }//分割方法名，取数组中的字符串
        if (methodInList) {
            "playing ${name.split('play')[1]}"
        } else {
            throw new MissingMethodException()
        }
    }
}

def jack = new Person()
println jack.work()
println jack.playTennis()
println jack.playVolleyball()
println jack.playBasketball()
```
打印结果：
```
working ... ...
methodMissing called for playTennis
playing Tennis
methodMissing called for playVolleyball
playing Volleyball
methodMissing called for playBasketball
playing Basketball
```

上面用通过methodMissing的特性拦截了对不存在方法的调用,同样通过`propertyMissing`，可以拦截对不存在的属性的访问,从调用者的角度来看，调用普通的方法和合成的方法病没有有什么两样,但是存在一个陷阱，重复的调用一个不存在的方法每次都需要处理会带来性能问题，所以需要把已经调用的方法缓存起来，这就是(拦截、缓存、调用)

```groovy
class Person1 {

    def work() {
        "working ... ..."
    }

    def plays = ['Tennis', 'Volleyball', 'Basketball'];

    def methodMissing(String name, args) {
        System.out.println("methodMissing called for $name")
        def methodInList = plays.find { it == name.split('play')[1] }//分割方法名，取数组中的字符串
        if (methodInList) {
            def impl = {
                Object[] vars->
                    "playing ${name.split('play')[1]}"
            }
            Person1 instance = this;//这里为什么不能直接使用this呢？
            instance.metaClass."$name"=impl//这样就把方法给缓存起来了
            impl(args)

        } else {
            throw new MissingMethodException()
        }
    }
}
```

---
## 2 使用GroovyInterceptable合成方法

```groovy
class Person2 implements GroovyInterceptable{

    def work() {
        "working ... ..."
    }

    def plays = ['Tennis', 'Volleyball', 'Basketball'];

    @Override
    def invokeMethod(String name, Object args) {
        System.out.println("intercept method $name")
        def method = metaClass.getMetaMethod(name, args)
        if (method) {
            method.invoke(this,args)
        }else {
            metaClass.invokeMethod(this,name,args)//注入三个参数的方法，两个参数的方法就是GroovyObject的方法了
        }
    }


    def methodMissing(String name, args) {
        System.out.println("methodMissing called for $name")
        def methodInList = plays.find { it == name.split('play')[1] }//分割方法名，取数组中的字符串
        if (methodInList) {
            def impl = {
                Object[] vars->
                    "playing ${name.split('play')[1]}"
            }
            Person2 instance = this;//这里为什么不能直接使用this呢？
            instance.metaClass."$name"=impl//这样就把方法给缓存起来了
            impl(args)

        } else {
            throw new MissingMethodException()
        }
    }
}

def jack = new Person2()
println jack.work()
println jack.playTennis()
println jack.playVolleyball()
println jack.playBasketball()
```

打印结果：
```
intercept method work
working ... ...
intercept method playTennis
methodMissing called for playTennis
playing Tennis
intercept method playVolleyball
methodMissing called for playVolleyball
playing Volleyball
intercept method playBasketball
methodMissing called for playBasketball
playing Basketball
```

---
## 3 使用ExpandoMetaClass合成方法
```groovy
class Person3{

    def work() {
        "working ... ..."
    }
}

def plays = ['Tennis', 'Volleyball', 'Basketball'];

Person3.metaClass.methodMissing = {
    String name, args->
        System.out.println("methodMissing called for $name")
        def methodInList = plays.find { it == name.split('play')[1] }//分割方法名，取数组中的字符串
        if (methodInList) {
            def impl = {
                Object[] vars->
                    "playing ${name.split('play')[1]}"
            }
            Person3.metaClass."$name"=impl//这样就把方法给缓存起来了
            impl(args)
        } else {
            throw new MissingMethodException()
        }
}

def jack = new Person3()
println jack.work()
println jack.playTennis()
println jack.playVolleyball()
println jack.playBasketball()
```

---
## 4 给具体的实例合成方法

前面学习了如果为具体的示例注入方法，通过向具体的实例提供的metaClass也可以将方法合成到这些实例中去。

能够基于实例的当前状态或者实例所接受到的输入创建动态的方法或者行为，为创建和实现高度动态的DSL铺平了道路。


```groovy
class Person4{
}

def emc = new ExpandoMetaClass(Person4)
emc.methodMissing={
    String name,args->
        "I am jack of all trades ... I can $name"
}

emc.initialize()
def jack = new Person4()
jack.metaClass = emc

println jack.sing()
println jack.paly()
println jack.fuck()
println jack.codeing()
```

