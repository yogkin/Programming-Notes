# 第十章：使用脚本和类

---
## 1  Java和Groovy的混合

在应用中，可以在一个Java类、一个Groovy类、一个Groovy脚本中实现某个特定的功能，之后可以在Java类中、Groovy类或Groovy脚本中调用该功能。

-  在Groovy脚本中使用Groovy类无需做什么，直接使用即可，只需要保证所依赖的类在classpath下即可，要么时源代码要么是字节码
-  如果在java中需要使用Groovy脚本，则可以使用JSR2223提供的ScriptEngine API
-  如果需要在Java中需要Groovy类，或者在Groovy中使用Java类，可以利用Groovy的联合编译(joint-compilation)工具

---
## 2 运行Groovy代码

使用Groovy可以直接使用groovy命名运行groovy代码:

使用java运行groovy代码可以先用groovyc把groovy编译成字节码，然后设置classpath后运行该字节码

 比如有一个Greet.class类：
 
```
println "hello from groovy"
```

使用Java运行Groovy类
```
        groovyc Greet.groovy
        java -jar G:/Developers/DeveloperSoftware/Groovy-2.4.6/embeddable/groovy-all-2.4.6.jar Greet
```
---
## 3 在Groovy中使用Groovy类

在Groovy中使用Groovy,要在Groovy代码中使用另一个Groovy类，只需要确保该类在我们的classpath下即可，
可以使用Groovy源代码也可以使用编译好的.class文件，groovy会优先查找.groovy的类文件，找不到还回去查找.class的类文件

---
## 4 利用联合编译混合使用Groovy和Java代码

比如一个 JavaClass.java和一个GroovyClass.groovy联合编译，可以使用下面命令进行联合编译：

```
        groovyc -j JavaClass.java GroovyClass.groovy//-j 用于把标志传给java编译器
```
---
## 5 在Java中创建与传递Groovy闭包

闭包的运行其实就是调用它的call方法，所以在java中向groovy传递闭包，需要实现一个带call方法的内部类：

```groovy
//groovy
class GroovyClosure {

        def useClosure(closure){
            closure()
        }

         def passToClosure(int value , closure){
            closure(value)
        }
}

//java
public class CallGroovyClosure {
    public static void main(String... args) {
        GroovyClosure aGroovyClass = new GroovyClosure();
        //调用无参闭包
        Object result = aGroovyClass.useClosure(new Object() {
            public String call() {
                return "You called from Groovy!";
            }
        });
        System.out.println("received : " + result);
        //调用有参数的闭包
        Object result2 =  aGroovyClass.passToClosure(4, new Object(){
            public String call(int value) {
                return "You called from Groovy! pass value is:"+value;
            }
        });
        System.out.println(result2);
        /*
        received : You called from Groovy!
        You called from Groovy! pass value is:4
         */
    }
}
```

---
## 6 在Java中调用Groovy的动态方法

Groovy可以在运行时创建方法，这些方法不能从java中直接调用，因为编译时，这写字节码还不存在，而java调用方法必须在
编译时知道方法名或者使用反射。
每一个Groovy对象都实现了GroovyObject接口，其中有一个叫invokeMethod的方法，该方法接受需要调用的方法名称和参数，在java这一端
可以使用invokeMethod的方法调用Groovy中使用元编程动态定义的方法。

Groovy类可以包含一个特殊的方法，methodMissing(String name, args),当某个调用的方法不存在时，该方法会被调用（前提是定义了该方法）

```groovy
//groovy代码
class DynamicGroovyClass {
    def methodMissing(String name, args) {
        println "you call $name with ${args}"
        args.size()
    }
}
//java代码
public class CallDynamicMethod {
    public static void main(String... args) {
        GroovyObject groovyObject = new DynamicGroovyClass();
        Object result1 = groovyObject.invokeMethod("squeak", new Object[]{});
        System.out.println(result1);
        Object result2 = groovyObject.invokeMethod("squeak", new Object[]{"like","a","duck"});
        System.out.println(result2);
        /*
        结果：
         you call squeak with []
         0
         you call squeak with [like, a, duck]
         3
         *
         */
    }
}

```

---
## 7 在Groovy中使用Java类

在Groovy中使用Java类非常简单，如果是JDK中的类，直接使用即可，如果是我们编写的类，则配好对应的classpath即可，
如果需要依赖的类与groovy脚本处于同一目录下，则不需要指定classpath

---
## 8 在Groovy中使用Groovy脚本

通过GroovyShell类，我们可以在一个脚本中调用另一个脚本，只需要调用evaluate方法，传入表示另一个脚本的File类。比如`Script1.groovy和Script2.groovy`:

Script1.groovy:
```groovy
println "in script1"
shell = new GroovyShell()
shell.evaluate(new File('script2.groovy'))
//或者直接
evaluate(new File('script2.groovy'))
```
Script2.groovy:
```groovy
println "hello from script2"
```

如果需要向脚本传递传递参数，可以使用Groovy脚本中的binding对象(每一个脚本都有一个binding对象),比如Script1a.groovy和Script2a.groovy：

Script1a.groovy:
```groovy
println "in script1"
name = "ztiany"//name不能用def修饰
shell = new GroovyShell(binding)
def evaluate = shell.evaluate(new File('script2a.groovy'))//evaluate 求..的值
println evaluate
//如果不希望把影响当前的binding，则创建一个binding，在该实例上调用setProperty
binding1 = new Binding()
binding1.setProperty('name','jordan')
shell1 = new GroovyShell(binding1)
shell1.evaluate(new File('script2a.groovy'))
binding2 = new Binding()
binding2.setProperty('name','kobe')
```

Script2a.groovy:

```groovy
println "hello from script2a , value is ${name}"
name = "dan"
//这个脚本需要一个遍历name，可以使用一个binding实例来绑定该变量
```


---
## 9 从Java中使用Groovy脚本

java中需要使用Groovy脚本，则可以使用JSR2223提供的ScriptEngine API，java6已经默认包含了JSR2223

```groovy
public class CallScript {
    public static void main(String... args) {
        ScriptEngineManager scriptEngineManager = new ScriptEngineManager();
        ScriptEngine groovy = scriptEngineManager.getEngineByName("groovy");
        try {
            groovy.eval("println 'Hello from Groovy' ");
            //eval方法还有其他的重载版本可以使用，比如接受一个Reader
            //使用put可以向脚本中传递参数
            groovy.put("name", "Ztiany");
            groovy.eval("println \"Hello $name from Groovy \"");
        } catch (ScriptException e) {
            e.printStackTrace();
        }
    }

}
```

