# 第二章：面向Java开发者的Groovy

因为groovy支持java语法，并保留了java语义，所以我们可以随心所欲的混用这两种语言风格，但是当我们把java代码重构为groovy代码后，会发现groovy会更加简洁简单。

---
## 1 Groovy基础语法

### 定义变量

groovy中的类型与java一致，但是groovy使用def来定义变量，可以指定变量的类型，也可以不指定类型

```
    def int a = 1
    def str1 = '你好啊'
    str2 = '你好啊'//定义变量时，def也是可以省略的，不过建议加上
    String str3 = "你还啊"//也可以按照Java的方式定义变量
```

###  字符串

```

    println 'Ztiany'            //单引号 字符串严格按对应ava里面的字符串
    println "打印 $str1"        //双引号字符串，可以使用$引用其他变量，即对$进行转义
    println '''我的天空
               有你不同
                哈哈
    '''                        //三个引号，表示打印段落,。
```

字符串后面会讲到。

###  定义函数

```
    String function01(arg1 , arg2){
        println arg1
        println arg2//最后一行是执行语句，所以没有返回值，则返回null
    }
```

函数可以不指定的返回值类型,这时需要用def来定义函数
```
    def noReturnType(){
        "你好啊"//最后一行的执行结果就是本函数的返回值,如果这是最后一行，返回值就是String
        // 10000 如果这是最后一行，返回值就是Integer
    }
```
如果指定了类型，则可以不必要用def来定义函数
```
    String getString(){
        return "返回String"
    }

     void doSomething(){//如果没有返回值，应该使用void来定义函数
            print('没有返回值')
    }
```
- 其实无返回类型的函数，内部都返回Object，Groovy是基于java的，最终groovy会被转换成java字节码在jvm上运行
- 函数可以不指定的返回值类型,那就需要用def来定义函数，这时函数的最后一行的执行结果就是本函数的返回值，如果函数指定了返回类型，就必须返回正确的类型



###  函数的调用

 println 即为调用打印函数，groovy对于函数的调用可以不写(),但是groovy经常把属性和函数掉用搞混淆，比如：
```
            定义函数
            def getString(){
               "hello"
            }
            //不带括号调用函数
            getString
            //这时会抛出异常groovy.lang.MissingPropertyException: No such property: getString for class: ConsoleScript3
```
这一点还是需要注意的。


###  assert语句
```
java中，assert只有在设置了运行时标志(-ea或者-enableassertion)来进行断言检查时才有用，而groovy的assert语句一直有用
```

### Groovy实现循环

```groovy
    0.upto(2) {//integer的upto方法 实现循环
        print "$it "//当闭包值由一个参数时，可以使用it引用这个参数
    }
    3.upto 5, {//参数可以不带括号哦
        print "$it "
    }
    //如果遍历从0开始，可以使用times
    4.times {
        print "$it "
    }
    0.step 10 ,2 , {//使用step可以指定循环的目标数，但每次会跳过指定的数据
        print("$it")
    }

//打印`ha ha ha merry groovy`
    for (i in 0..2) {//0..2 表示一直range数据结构，后面会讲
        print 'ha '
    }
    println 'merry groovy'
```

### GDK

GDK扩展了JDK，并在各个类中添加了很多方法，例如String的`execute`方法，更多的方法可以从[GroovyJDK](http://groovy-lang.org/gdk.html)获取

process 非常有用，可以与系统级进程进行交互：
```groovy
    println "git help".execute().text;//String 的execute方法用于执行String表达的命令
    println "git help".execute().getClass().name;//返回结果为：java.lang.ProcessImpl
    println "cmd /C dir".execute().text//dir并不是一个程序，而只是一个shell命令，所以需要通过调用cmd来执行dir命令
```
如果使用Java，则需要些如此多的代码：
```java
            private static void getProcessText() {
                BufferedReader br = null;
                try {
                     Process proc = Runtime.getRuntime().exec("git help");
                     br = new BufferedReader(new InputStreamReader(proc.getInputStream()));
                    String line;
                    while ((line = br.readLine() )!= null){
                        System.out.println(line);
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }finally {
                    if (br != null) {
                        try {
                            br.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
```
### 安全操作符
java中需要进行各种判断来避免空指针，而groovy使用操作符"?."
```groovy
    //定义一个函数
    def revereStringSafe(String str) {
        str?.reverse()
    }
    println revereStringSafe("ddffaa")//调用函数
    println revereStringSafe(null)
```
### 异常处理
Groovy并不强制处理任何异常，我们不处理的异常都会别自动的传递到上一层
```groovy
    def openFile(fileName) {
        new FileInputStream(fileName)//这里并没有强制我们检测一次
    }
    try {
        openFile(openFile("noExist"))//这里会抛出异常,如果不try catch程序就会退出
    } catch (Exception ex) {
        println ex
    }
//如果需要捕获的是Exception，也可使用如下方式(省略掉Exception)：
    try {
        openFile(openFile("noExist"))//这里会抛出异常
    } catch (ex) {//可以省略掉Exception，但是对于Error，Throwable就不能省略了
        println ex
    }
```
###   静态方法中可以使用this
```groovy
    class Wizard {
        def static learn(int a) {
            println(a)
            this
        }
    }
    Wizard.learn(1).learn(3).learn(4)
```
### Groovy 其他特性

- return 几乎是可选的，最后一个语句的执行结果将自动作为返回值
- 分号几乎总是可选的
- 方法和类默认都是公开的
- `?.`操作符只有对象引用不完空是才会被调用
- 可以使用具名参数初始化JavaBean
- Groovy不强迫开发者捕获自己不关心的异常，没有被捕获的将自动向上传递
- 静态方法中可以使用this来引用Class对象

---
## 2 JavaBean

在Java中，JavaBean中的字段总是伴随者setter和getter方法，这样看起来非常的繁琐，而在Groovy中，javaBean变得很简单：

```groovy
class Car{
    def miles = 0;
    final  year;

    Car(year) {
        this.year = year
    }
}
```
Groovy会自动给JavaBean生成**访问器和更改器**，当使用`Car.miles`其实是调用的访问器，对于被final修饰的year，Groovy不会提供**更改器**，当视图修改year时**会抛出异常**，可以根据需要给字段加上类型信息，比如可以把miles设置为private的，但是groovy并不遵守这一点(还是可以被访问)，最好的做法是手动添加一个拒绝修改的**更改器**

```groovy
    private def miles = 0;
    void setMiles(miles) {
        throw new IllegalAccessException("no allow")
    }

//执行代码
Car car = new Car(2008)
println(car)
//car.year = 2004;//会报错 Cannot set readonly property: year for class: ztiany.chapter2.Car
//car.miles = 43;//会报错 no allow
println(car.miles)
```

---
## 3 灵活初始化与具名参数


```groovy
class Robot {

    def type, height, width

    def access(location, weight, fragile) {
        //注意打印的顺序是倒着打印的 fragile-weight-location
        println "received fragile? $fragile, weight:$weight, loc:$location"
    }
}
```
初始化Robot

```groovy
//robot实例把type，height，width等实参当作了名值对,要求类必须有一个无参的构造器
robot = new Robot(type: 'arm', width: 10, height: 40)
println "$robot.type , $robot.height, $robot.width"//打印结果：arm , 40, 10
```

access方法有三个形参，如果第一个是map，则可以将这个映射中的键值对展开放在实参列表中
```groovy
robot.access(x: 20, y: 20, z: 10, 50, true)//这里我们依次放入了映射、weight、fragile，打印结果：received fragile? true, weight:50, loc:[x:20, y:20, z:10]
robot.access(50, true, x: 20, y: 20, z: 10)//映射的传递可以往后移，这个一定要注意，不然很容易被迷惑，打印结果：received fragile? true, weight:50, loc:[x:20, y:20, z:10]
```

如果方法的实参的个数多余形参的个数，并且多出来的实参是名值对，那么groovy就会假设方法的第一个形参是一个Map，然后将所有的名值对组织到一起作为方法的第一个实参
尽管这种灵活性非常强大，但是可能给人带来困惑，所以谨慎的使用，如果想使用具名参数，那最好只接受一个Map形参，而不要混用不同的形参,如：

```groovy
def access(Map location, weight, fragile) {
    println "received fragile? $fragile, weight:$weight, loc:$location"
}
access(x: 20, y: 20, z: 10, 50, true)//这时如果传入的不是两个对象和一个名值对就会报错。//打印结果：received fragile? true, weight:50, loc:[x:20, y:20, z:10]
```

---
## 4 可选形参

Groovy可以把方法和构造器中的形参设置为可选的，想设置多少就可以设置多少，但是要求这些可选的形参必须放在参数列表的最后面，**定义形参只需 为其设置一个默认值**

```groovy
def log(x, base = 10) {
    Math.log(x) / Math.log(base)
}

println log(1024)
println log(1024, 10)
println log(1024, 2)
```
不仅如此，Groovy会把末尾的数据形参视作可选的，可以为最后一个参数提供0个或者多个值

```groovy
def task(name, String[] details) {
    println "$name ,  $details"
}

task 'call', '123-456-7890'
task 'call', '123-456-7890', 'ee3-456-rwe2'
task 'call'
```


---
## 5 使用多赋值

```groovy
    def splitName(fullName) {
        fullName?.split(' ')
    }
    def (firstName, secondName) = splitName("Zhan tianyou")//从splitName方法接受多个参数
    println firstName + "  " + secondName
```

还可以使用该特性来交换变量，无需创建中间变量来保存被交换值

```groovy
    def name1 = "Ztiany"
    def name2 = "MeiNv"
    println "$name1 and $name2"
    (name1, name2) = [name2, name1]
    println "$name1 and $name2"
```
当变量与值的数量不相等时，如果有多余的变量，Groovy会将它们设置为null，多余的值则会被抛弃，**如果对于的变量是不能设置为null的基本类型，Groovy将会抛出一个异常**,在Groovy2中，只要可能int会被看作基本类型，而非Integer


---
## 6 实现接口

假设UIAPI中有一个Button,我们需要为其设置各种监听事件。

```groovy
class Button {

    final name;
    OnClickListener mClickListener
    private OnTouchListener mOnTouchListener
    private OnFocusListener mFocusListener

    Button(name) {
        this.name = name
    }

    def setOnClickListener(OnClickListener onClickListener) {
        mClickListener = onClickListener
    }

    def setOnTouchListener(OnTouchListener onTouchListener) {
        mOnTouchListener = onTouchListener
    }

    def seFocusListener(OnFocusListener onFocusListener) {
        mFocusListener = onFocusListener
    }

    interface OnClickListener {
        void onClick(Button clickButton)
    }

    interface OnTouchListener {
        void onTouch(Button touchButton)
    }

    interface OnFocusListener {
        void focusGained(Button thisButton)

        void focusLost(Button thisButton)
    }

    def performClick() {
        mClickListener?.onClick(this)
    }

    def performTouch() {
        mOnTouchListener?.onTouch(this)
    }

    def performFocusGained() {
        mFocusListener?.focusGained(this)
    }

    def performFocusLost() {
        mFocusListener?.focusLost(this)
    }
}
```


在Groovy中，可以把一个映射或者一个代码块转换为接口，因此可以快速实现带有多个方法的接口

**java方式的注册事件**：
```java
button.setOnClickListener(new Button.OnClickListener() {
    @Override
    void onClick(Button clickButton) {
        println clickButton
    }
})

button.performClick()
```
**groovy提供了其他方式**：不需要actionPerformed方法声明，也不需要显式的`new`

```groovy
button.setOnClickListener(
        {
            println it
        } as Button.OnClickListener //借助as操作符，相当于实现了OnClickListener接口，it表示方法的参数，当方法的参数不止一个时，可以分别定义方法参数
)

button.performClick()
```

当然也可以先定义实现再作为参数传入
```groovy
listener = {
    println it.name + ' listener '
}
//listener被as转化了两次
button.setOnClickListener(listener as Button.OnClickListener)
button.setOnTouchListener(listener as Button.OnTouchListener)
button.performClick()
button.performTouch()
```

对于有多个方法的接口，如果打算所有的方法提供共同的实现，和下面一样不需要多做任何操作(可以把focusListener成了FocusGained和FocusLost的实现):

```groovy
focusListener = {
    println it.name + "focusListener"
}
button.seFocusListener(focusListener as Button.OnFocusListener)
button.performFocusGained()
button.performFocusLost()
```
Groovy没有强制实现接口中所有的方法，可以定义自己关心的方法，而不考虑其他方法,但是大多数情况下，大多数接口的方法都需要被实现，这时可以创建一个映射，以每个方法名作为key，以方法对一个的代码作为值传入，并且并不要实现所有的方法，但是如果没有提供的方法被调用了，将会抛出一个NullPointerException

```groovy
handFocus = {

    focusGained:
    {
        println it.name + ' focusGained'
    }

    focusLost:
    {
        println it.name + ' focusLost'
    }
}

button.seFocusListener(handFocus as Button.OnFocusListener)
button.performFocusGained()
button.performFocusLost()
```

---
##  7 Boolean求值

Groovy中的布尔求值与Java不同，根据上下文，Groovy会把表达式计算为布尔值，java代码中的if语句必须要求表达式的结果是个布尔值，而Groovy不会，Groovy会尝试推断，比如在需要boolean的地方放一个对象引用，Groovy会尝试检查引用是否为null，它将null视作false，将非null视作true，如：

```groovy
str1 = "hello"
str2 = null
if (str1) {
    println str1
}
if (str2) {//不会被打印
    println str2
}
```

注意，当面所说的判断true/false的条件并不全面，对于集合来说，Groovy判断false的条件是 集合是否为空，只有当集合不为null，并且含有至少一个元素时才会计算为true

下面是一个被特殊对待的类型列表：

| 类型           | 为真条件|
| ------------ | ----------------------- |
| Boolean      | 值为true                |
| Collection   | 集合不为空              |
| Character    | 值不为0                 |
| CharSequence | 长度大于0                |
| Enumeration  | Has More Elements 为true |
| Iterator     | hasNext 返回true        |
| Number       | Double值不为0           |
| Map          | 该映射不为空             |
| Matcher      | 至少又要给匹配           |
| Object[]     | 长度大于0                |
| 其他类型      | 引用不为null          |


---
## 8 操作符重载

Groovy支持操作符重载，可以巧妙的应用这一点来创建DSL领域特定语言，java不支持，Groovy的做法的每一个操作符都会映射到一个标准的方法，在java可以使用的那些方法，在groovy中既可以使用操作符，也可以使用与之对应的方法

例如 ,这里的ch++ 映射到的是String的next方法

```groovy
for (ch = 'a'; ch < 'd'; ch++) {
    println(ch)
}
for (ch in 'a'..'d') {
    println(ch)
}
//String和ArrayList重载了很多的操作符，如ArrayList的leftShift
list = []
list << "d"
println list
```


下面为一个类添加一个+的操作符

```groovy
class ComplexNumber {
    def real, imaginary

    def plus(other) {
        new ComplexNumber(real: real + other.real, imaginary: imaginary + other.imaginary)
    }

    @Override
    public String toString() {
        return "ComplexNumber{" +
                "real=" + real +
                ", imaginary=" + imaginary +
                '}';
    }
}
println new ComplexNumber(real: 2, imaginary: 4) + new ComplexNumber(real: 5, imaginary: 3)
```

当应用于某个上下文时，操作符重载可以使代码更富于表现力，应该只重载那些能够使事物变得显而易见的操作符，下面的表格描    述了groovy中的操作符所映射到的方法：

| Operator                | Method            |
| ----------------------- | ----------------- |
| a + b                   | a.plus(b)         |
| a – b                   | a.minus(b)        |
| a * b                   | a.multiply(b)     |
| a ** b                  | a.power(b)        |
| a / b                   | a.div(b)          |
| a % b                   | a.mod(b)          |
| a或b                    | a.or(b)           |
| a & b                   | a.and(b)          |
| a ^ b                   | a.xor(b)          |
| a++ or ++a              | a.next()          |
| a– or –a                | a.previous()      |
| a[b]                    | a.getAt(b)        |
| a[b] = c                | a.putAt(b, c)     |
| a << b                  | a.leftShift(b)    |
| a >> b                  | a.rightShift(b)   |
| switch(a) { case(b) : } | b.isCase(a)       |
| ~a                      | a.bitwiseNegate() |
| -a                      | a.negative()      |
| +a                      | a.positive()      |
>注意： a或b即`a|b`


---
## 9 Groovy对java1.5的支持

下面是JDK1.5的新特性：

- 自动装箱
- for-each循环
- enum
- 变长参数
- 注解
- 静态导入
- 泛型


### 自动装箱

Groovy从一开始就支持自动装箱，必要时Groovy会将基本类型视作对象

    int val = 4;
    println val.class.getName()//java.lang.Integer

虽然定义的是int，但是创建的Integer实例，而不是基本类型，Groovy会根据该实例的使用方式来决定将其存储为int类型还是Integer类型，Groovy2.0之前，所有的基本类型都会视作对象，之后为了改善性能，只有在需要的时候才会被视作对象

### fro each groovy

对循环的支持强于java，传统的增强for循环，groovy需要指明类型，而用in 则不需要

```groovy
list = [1, 2, 3, 4]
for (int i : list) {
    println i
}
for (i in list) {
    println i
}
```

### Groovy同样支持枚举，它是类型安全的

```
enum CoffeeSize{
    Short, Small, Medium, Large,Mug
}

def orderCoffee(size) {
    switch (size) {
        case [CoffeeSize.Short, CoffeeSize.Small]:
            println('111111111')
            break
        case [CoffeeSize.Medium,CoffeeSize.Large]:
            println('22222222')

            break
        case CoffeeSize.Mug:
            println('333333')
            break
    }
}

orderCoffee CoffeeSize.Short
orderCoffee CoffeeSize.Small
orderCoffee CoffeeSize.Medium
orderCoffee CoffeeSize.Large
orderCoffee CoffeeSize.Mug

for (coffee in CoffeeSize.values()) {
    println coffee.name()
}
```


### 变长参数

Groovy以两种形式支持Java的编程参数：

- 一种是用`...`方式
- 一种是以数组为末尾的形参方法

```groovy
def receiveVarArgs(int a, int ... b) {
    println "You passed $a and $b"
}
def receiveArray(int a, int [] b) {
    println "You passed $a and $b"
}
receiveVarArgs 1,3,5,6,8
receiveArray 1,2,32,34,54
//在向带数组形参的方法传递数组时，需要注意.[1,2,3]默认类型为List，所以传递数据是需要使用下面方法
int[] arr = [1, 3, 7, 77]
arr2 = [3, 44, 34, 65] as int[]
receiveArray 1, arr
receiveArray 4, arr2
```


### 注解

Java使用注解来表示元数据，Java5中也定义了一些注解，`@Override，@Deprecated，@SuppressWarnings`,Groovy也可以定义和使用注解，使用方式与java相同，但对于Java编译相关的注解，Groovy的理解会有所不同，比如Groovy会忽略Override注解

### 静态导入

Groovy不仅支持静态导入，还支持静态导入别名

### 泛型

Groovy 是支持可选类型的动态类型语言，作为java的超级它也支持泛型，但Groovy编译器不会像Java编译器那样检查类型，Groovy的动态类型和泛型类型互相作用使我们的代码运行起来

```groovy
ArrayList<Integer> list = new ArrayList<>()
list.add(33)
list.add("Dd")
for (int i   : list) {//Cannot cast object 'Dd' with class 'java.lang.String' to class 'int'
    println i
}
//在add方法中，groovy只会把类型看做一种建议，对集合进行循环的时候，Groovy会尝试将元素转换成int类型，这是就导致错误
```

---
##  10 使用Groovy生成代码

Groovy在groovy.transform包和其他一些包中提供了很多代码生成的注解

- **@Canonical** 用于生成toString
- **@Delegate** 实现代理
- **@Lazy** 延迟加载对象
- **@Newify**  声明的方式创建对象
- **@Singleton**
- **@InheritConstructors**  创建构造函数

### Canonical 用于生成toString

```groovy
@Canonical(excludes = 'lastName')
class Person {
    def firstName
    def lastName
    def age
}
println new Person(firstName: "Zhan", lastName: "Tianyou", age: 26)
```

### Delegate

Delegate 只有当派生的类是真的可替换的，而且可代替基类，继承才显示出其优势，从存粹的代码复用角度来讲，委托要由于继承，而Java中使用委托会是代码变得冗余，而且需要更多工作，Groovy是委托变得容易

```groovy
class Worker {
    def work() {
        println "get work done"
    }
    def analyze() {
        println "analyze......"
    }
    def writeReport() {
        println 'get report written...'
    }
}

class Expert {
    def analyze() {
        println "Expert analyze......"
    }
}

class Manager {
    @Delegate
    Expert mExpert = new Expert()
    @Delegate
    Worker mWorker = new Worker()
}
bernie = new Manager()
bernie.analyze()
bernie.work()
bernie.writeReport()
```
在编译时，Groovy会检查Manager类，如果该类中没有被委托的方法，就把这些方法从北委托的类中引进来，因此，首先它会引入Expert类的analyze方法，而从Worker类中只会把work和writeReport方法引进来，因analyze方法已经从Expert引入

### Lazy

我们想把耗时对象的创建推迟到真正需要时，完全可以懒惰与高效并得，编写更好的代码，同时又能获得懒惰初始化的所有好处

```groovy
class Heavy {
    def size = 10;

    Heavy() {
        println "Creating heavy with $size"
    }
}

class AsNeeded {
    def value
    @Lazy
    Heavy mHeavy1 = new Heavy()
    @Lazy
    Heavy mHeavy2 = {
        new Heavy(size: value)
    }()//这里是调用闭包的方法
    AsNeeded() {
        println "Creating AsNeeded"
    }
}

def asNeed = new AsNeeded(value: 10000)
println asNeed.mHeavy1.size
println asNeed.mHeavy1.size
println asNeed.mHeavy2.size
//Lazy不仅推迟了对象的创建，将将字段标志位volatile的
//Lazy的第二个好处是提供给了一种轻松实现线程安全的虚拟代理模式
```
### Newify

Groovy经常按照传统的Java语法创建对象:new,    然而在创建DSL时，去掉这个关键字表达会更流畅，@Newify可以帮助创建类似Ruby的构造器，于是new变成了该类的一个方法。这个注解还可以用来创建类似Python的构造器，可以完全去掉new，此时@Newify必须指定类型，只有将auto=false这个元素据值设置给@Newify，才能创建类似
Ruby的构造器:

```groovy
class CreditCard {
    def number
    def areaCode
}

@Newify([Person, CreditCard])//现在，fluentCreate可以使用新的方式创建对象了。
def fluentCreate() {
    println Person.new(firstName: "zhan", lastName: "tianyou", age: 26)
    println Person(firstName: "liu", lastName: "fefie", age: 25)
    println CreditCard(number: '535-553-0555', areaCode: 2000)
}
fluentCreate()
```
可以在不同的作用于使用@Newify注解，类或者方法，在创建DSL是，@Newify注解非常有用

### Singleton 用于实现单利 ，使用lazy可以设置为懒汉式

```groovy
@Singleton(
        strict = false
)
class TheUnique {
    private TheUnique() {
        println "TheUnique created $this"
    }

    def hello() {
        println "hellow $this"
    }
}
TheUnique.instance.hello();
TheUnique.instance.hello();
TheUnique theUnique = new TheUnique()
theUnique.hello()
```
使用Singleton注解，会使目标类的构造方法变为私有的，但是Groovy并不区分私有的还是共有的，所以在Groovy内部还是可以使用new来创建类的对象，但是必须谨慎的使用该类，

### InheritConstructors

使用InheritConstructors可以帮助我们生成构造器
```groovy
class Father {
    Father(int a) {

    }

    Father(int a, int b) {

    }

    Father(int a, int b, int c) {

    }
}

@InheritConstructors//现在Son不需要声明构造器了。
class Son extends Father {

}
```


---
## 11 Groovy的陷阱

- Groovy的 == 等于Java中的equals
- 编译时类型检查默认关闭
- def in 都是groovy中的关键字
- 在Groovy中，方法内不能有局部代码块
- 闭包与匿名内部类冲突
- 分号总是可选的，但有些地方却必不可少
- 创建基本类型的数组语法不同
- 一般情况下，应该使用getClass方法，向Map、生成器等一些类对class属性有特殊的处理，为了避免出现意外，应使用getClass方法

### Groovy的 == 等于Java中的equals

如果希望比较引用是否相等，需要用Groovy中的is()方法,但是事情情况并不总是如此，真实的情况是，当且仅当该类没有实现Comparable接口时，==操作才会映射到equals方法，否则映射到Comparable的compareTo方法

```groovy
str1 = 'hello'
str2 = str1
str3 = new String('hello')
str4 = 'Hello'

println(' str1 == str2  ' + (str1 == str2))
println(' str1 == str3  ' + (str1 == str3))
println(' str1 == str4  ' + (str1 == str4))
println(' str1.is(str2) ' + (str1.is(str2)))
println(' str1.is(str3) ' + (str1.is(str3)))
println(' str1.is(str4) ' + (str1.is(str4)))

打印结果：
 str1 == str2  true
 str1 == str3  true
 str1 == str4  false
 str1.is(str2) true
 str1.is(str3) false
 str1.is(str4) false
```

### 编译时类型检查默认关闭
Groovy是类型可选的，Groovy编译器大多数情况下不会执行完整的类型检查，而是在遇到类型定义是执行强制类型转。

如果把一个类型赋值给一个Integer，变量：

```groovy
Integer a = 31;
a = "ds"
```


可以正常编译通过，但是会在运行时进行类型转换 ，在Groovy中` x = y `在语义上等价于` x = (ClassOfX)y`，即强转，同样的，调用一个类不存在的方法，也不会在编译时期发生错误，但是会抛出MissingMethodException异常:`a.noMethod()`。
虽然这种编译器看上去并不严格，但事实上，对于动态编程和元编程来说，这是必要的。

### def、in 都是groovy中的关键字

def用于定义方法和属性等，而in用于指定循环的区间，**也不要在java代码中使用it变量**，如果在闭包中使用了这个参数他引用的是闭包的参数

### 在Groovy中，方法内不能有局部代码块

```groovy
def functionA() {
//    {} 不能有这样的代码块，否则groovy以为是闭包
}
```

### 闭包与匿名内部类冲突

闭包使用花括号，而匿名内部类也使用花括号，所以会造成冲突，如下一个类接受一个闭包参数

```groovy

class Calibrator {
    Calibrator(calculationBlock) {
        println 'Calibrator created'
        calculationBlock()
    }
}

new Calibrator({
    println 'ddddd'
})
closesure = {
    println 'ccccc'
}
new Calibrator(closesure)
```
### 分号总是可选的 ，至少有一个地方分号是必不可少的

```groovy
class Semi {
    def val = 3;
//这行代码如果不加分号，.将导致运行时MissingMethodException: No signature of method: java.lang.Integer.call()，不懂为什么是这个异常？其把下面构造代码块看成了一个闭包(闭包后面再学)就可以理解。
    {
        println "Semi created"
    }
}
println new Semi()
```
如果使用静态代码块则不存在这样的问题，如果同时使用静态代码块和构造代码块，可以把构造代码块放在静态代码块的前面，从而避免分号。

### 创建基本类型的数组语法不同

在groovy中，创建数据使用以下方式:
```groovy

int[] arr1 = [1, 2, 4]
def arr2 = [1, 2, 5] as int[]
```

---
## 12 Groovy操作符

[参考](http://blog.csdn.net/dabaoonline/article/list/2)

### 展开操作符(*)

通常被集合对象调用其所有的对象使用。等价于遍历调用,并将结果收集在一个集合中:

```groovy
        def items = [4,5]
        def list = [1,2,3,*items,6]//展开集合
        assert list == [1,2,3,4,5,6]

        def m1 = [c:3, d:4]
        def map = [a:1, b:2, *:m1]//展开map
        assert map == [a:1, b:2, c:3, d:4]
```

### in操作符

in操作符等价于isCase方法。在一个集合中,它等价于调用contains,例如:
```groovy
    def list = ['Grace','Rob','Emmy']
    assert ('Emmy' in list)
```
等价于调用`:list.contains(‘Emmy’)或者list.isCase(‘Emmy’)`

### 强制操作符

强制操作符(as)用于变量的强制转换。强制转换操作符转换对象从一个类型向另外一个类型而不考虑其兼容性。举个例子:
```groovy
   Integer x = 123
   String s = (String) x
```
由于Integer不可以自动转化成String,所以在运行时会抛出一个ClassCastException 。

不过可以使用强制操作符替代:

```groovy
        //可以避免异常
        Integer x = 123
        String s = x as String
        //Cannot cast object 'Sun Feb 26 00:48:41 CST 2017' with class 'java.util.Date' to class 'java.lang.Integer
       def d = new Date()
       i = d as Integer
```

