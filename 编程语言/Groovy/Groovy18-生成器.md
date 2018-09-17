# 第十七章：Groovy生成器

**生成器是内部的DSL**，为处理某些特定类型的问题提供了方便，如果需要使用嵌入的、层次式结构的(树、xml、json、html)等表示形式，生成器将会非常有用

---
##  1 处理xml

使用`groovy.xml.MarkupBuilder`生成器来创建xml文档，在生成器的上調用任意的方法或属性時,它会根据的调用的上下文，体贴的假设我们引用的是所生成文档中的元素名：

调用languages方法表示创建一个languages接口，附上的闭包提供了一个内部的上下文，闭包内的任何方法调用都会认为是上一个节点的子节点。调用方法中传入单个String表示输出一个节点，传入Map参数表示为节点的属性：

```groovy
bldr1 = new MarkupBuilder()

bldr1.languages {
    language(name: "c++") {
        author('Stroustrup')
    }
    language(name: "java") {
        author('Gosling')
    }
    language(name: "Lisp") {
        author('McCarthy')
    }
}
println ""

//也可以在MarkupBuilder创建时指定要给Writer
langs = ['c++':'Stroustrup' ,'java':'Gosling', 'Lisp':'McCarthy']
StringWriter stringWriter = new StringWriter()

bldr2 = new MarkupBuilder(stringWriter)
bldr2.languages {
    language(name: "c++") {
        author('Stroustrup')
    }
    language(name: "java") {
        author('Gosling')
    }
    language(name: "Lisp") {
        author('McCarthy')
    }
}
println stringWriter.toString()
```
也可以在MarkupBuilder创建时指定要给Writer
```
langs = ['c++':'Stroustrup' ,'java':'Gosling', 'Lisp':'McCarthy']
StringWriter stringWriter = new StringWriter()

bldr2 = new MarkupBuilder(stringWriter)
bldr2.languages {
    language(name: "c++") {
        author('Stroustrup')
    }
    language(name: "java") {
        author('Gosling')
    }
    language(name: "Lisp") {
        author('McCarthy')
    }
}
println stringWriter.toString()
```
MarkupBuilder适用于中小型的xml，如果文件够大(若干兆字节)，可以string用StreamingMarkupBuilder，它的内存占用可能会更好一些:
```
xmlDocument = new StreamingMarkupBuilder().bind {
    println mkp//.BaseMarkupBuilder$Document@4690b489 mkp是一个特殊的命名，指向Document
    mkp.xmlDeclaration()//声明xml 文件 <?xml version='1.0'?>
    mkp.declareNamespace(computer:"Computer")//声明命名空间
    languages{//声明languages节点
        comment  << "Created using StreamingMarkupBuilder"//注释
        langs.each{
            key,value->
                computer.language(name:key){//使用computer命名空间 创建language节点，并设置属性name为key
                    author(value)//在language节点在创建author节点，把value作为其内容
                }
        }
    }
}

println xmlDocument
```

---
## 2 处理Json

使用JsonBuilder可以很方便的构建json，而使用JsonSlurpe可以快速的解析json。

```groovy
class Person{
    String first
    String last
    def sins
    def tools
}

john = new Person(first: "john", last: "Smith", sins: ['java', 'groovy'], tools: ['script': 'Groovy', 'test': 'Spock'])

bldr = new JsonBuilder(john)
writer = new StringWriter()
bldr.writeTo(writer)
println writer//{"sins":["java","groovy"],"first":"john","last":"Smith","tools":{"script":"Groovy","test":"Spock"}}

//当然可以对输出加以定制
bldr1 = new JsonBuilder(john)

bldr1{
    firstName john.first
    lastName john.last
    "special interest groups" john.sins
    "preferred tools"{
        numberOfTools john.tools.size()
        tools john.tools
    }
}
writer1 = new StringWriter()
bldr1.writeTo(writer1)
println writer1
//JsonBuilder可以从JavaBean、HashMap、列表生成JSON格式的输出。
//Json格式的输出内容保留在内存中，然后调用writeTo方法写入流中，如果希望在创建时就在流中可以使用StreamingJsonBuilder

bldr2 = new JsonBuilder(john)
def writer = new FileWriter('person.json')
bldr2.writeTo(writer)
writer.flush()
writer.close()


//--------------------
//使用JsonSlurper解析Json
//--------------------
sluper = new JsonSlurper()
def parse = sluper.parse(new FileReader('person.json'))
println "$parse.first , $parse.last , ${parse.sins.join(' , ')}"
```


---
## 3 构建Swing应用

略


---
## 4 使用元编程定制生成器

对于使用嵌入的、层次式结构的(树、xml、json、html)的专门化任务，生成器提供了一个创建内部DSL的方式，当在应用中处理专门化任务时，可以检查一下是否有生成器
可以解决这个问题，如果没有则可以执行创建。

定制生成器的方式有两种：
- 利用Groovy的元编程能力，一切自己来，
- 利用Groovy提供的BuilderSupport或FactoryBuilderSupport

```groovy
class ToDoBuilder {
        def level = 0 //代表任务的层级嵌套深度
        def result = new StringWriter()//存储生成的DSL

    def build(Closure closure) {
        result << "todo:\n"
        closure.delegate = this
        closure()
        println result
    }

    def methodMissing(String name,args) {
        handle(name,args)
    }

    def propertyMissing(String name) {
        Object[] empty = []
        handle(name,empty)
    }

    def handle(String name, args) {
        level++
        level.times {
           result << "  "
        }

        result << placeXifStatusIsDone(args)
        result << name.replaceAll("_"," ")
        result << printParameters(args)
        result << "\n"

        if(args.length > 0 && args[-1] instanceof Closure){
            def theClosure = args[-1]
            theClosure.delegate = this
            theClosure()
        }
        level--
    }

    def placeXifStatusIsDone(args) {
        args.length > 0 && args[0] instanceof Map && args[0]['status'] == 'done'? "x  ":'-  '
    }

    def printParameters(args) {
        def values = ""
        if(args.length > 0 &&  (args[0] instanceof Map) ){
            values += '['
            def count = 0
            args[0].each {
                key,value->
                    count++
                    values += (count > 1? " ":"")
                    values+= "$key : $value"
            }
            values += ']'
        }
        values
    }
}


//使用ToDoBuilder处理下面层次结构：
bldr = new ToDoBuilder()
bldr.build{
       Prepare_Vacation(start: '02/15',end : '02/22'){
           Reserve_Flight(on:'01/01',status:'done')
           Reserve_Hotel(on:'01/02')
           Reserve_Car(on:'01/02')
       }

        Buy_New_Mac{
            Install_QuickSilver
            Install_TextMate
            Install_Groovy{
                Run_all_tests
            }
        }
}
```

结果为：
```
todo:
  -  Prepare Vacation[start : 02/15 end : 02/22]
    x  Reserve Flight[on : 01/01 status : done]
    -  Reserve Hotel[on : 01/02]
    -  Reserve Car[on : 01/02]
  -  Buy New Mac
    -  Install QuickSilver
    -  Install TextMate
    -  Install Groovy
      -  Run all tests
```


在上述用于待办事项列表的DSL中，使用ToDoBuilder来处理，使用methodMissing和propertyMissing方法来拦截不存在的方法和属性名，位置知道的方法时build。几乎全部都是标准字节的Groovy代码，而且很好的应用了元编程，当调用不存在的方法和属性名时，就假单它时一个条目。当面对嵌套层次较深，而且大量使用Map和普通参数的复杂情况，使用BuilderSupport较好。


---
## 5 使用BuilderSupport

如果要创建的生成器不止一个，可能要将某些方法识别代码重构到一个公共基类中，而Groovy已经提供了BuilderSupport类用于识别节点节点结构的便捷方法，就像解析xml的sax一样，我们只需要监听节点和属性事件。


TodoBuilderWithSupport扩展了BuilderSupport。BuilderSupport提供了setParent方法和重载版本的createNode方法，还有一个可选的nodeCompleted方法，setParent 用于让生成器的作者直到当前所处理的节点的父节点，重载版本的createNode方法用于创建节点，不管其返回什么参数，都将视为节点，而生成器支持将其一个参数发送给nodeCompleted，关于重载的createNode方法，在DSL中如果传入Map+闭包，者调用的是createNode(Object name, Map attributes) 方法，此时闭包被当作一个方法而不是参数。BuilderSupport不会像处理确实的方法那样处理确实的属性。然而仍然可以使用propertyMissing方法来处理这些情况。

```groovy
class TodoBuilderWithSupport extends BuilderSupport {

    int level = 0;
    def result = new StringWriter()

    @Override
    void setParent(Object parent, Object child) {
        println "${parent.getClass()}    ---    ${child.getClass()}"
    }

    @Override
    def createNode(Object name) {//没有参数的方法
        if (name == ("build")) {
            result << "todo:\n"
            'buildnode'
        } else {
            handleName(name, [:])
        }
    }

    @Override
    def createNode(Object name, Object value) {
        throw new Exception("invalid format")
    }

    @Override
    def createNode(Object name, Map attributes) {//map 参数的方法
        handleName(name, attributes)
    }

    @Override
    def createNode(Object name, Map attributes, Object value) {
        throw new Exception("invalid format")
    }

    @Override
    void nodeCompleted(Object parent, Object node) {
        level--
        if (node == "buildnode") {
            println result
        }
    }

    def propertyMissing(String name) {
        handleName(name, [:])
        level--//因为这里不会调用nodeCompleted方法，所以需要手动 减去层级
    }


    def handleName(String name, args) {
        level++
        level.times {
            result << "  "
        }
        result << placeXifStatusIsDone(args)
        result << name.replaceAll("_", " ")
        result << printParameters(args)
        result << "\n"
        name
    }

    def placeXifStatusIsDone(args) {
        args['status'] == 'done' ? "x  " : '-  '
    }

    def printParameters(args) {
        def values = ""
        if (args.size() > 0 ) {
            values += '['
            def count = 0
            args.each {
                key, value ->
                    count++
                    values += (count > 1 ? " " : "")
                    values += "$key : $value"
            }
            values += ']'
        }
        values
    }

}


builder = new TodoBuilderWithSupport()

builder.build {
    Prepare_Vacation(start: '02/15', end: '02/22') {//map

        Reserve_Flight(on: '01/01', status: 'done')//map
        Reserve_Hotel(on: '01/02')//map
        Reserve_Car(on: '01/02')//map

    }

    Buy_New_Mac {
```
---
## 6 使用FactoryBuilderSupport


BuilderSupport适合处理层次式结构，然而对于处理不同类型的节点并不方便，假设需要处理20种节点类型，createNode方法将会变得很复杂，
基于节点名称创建不同类型的节点会导致大量的麻烦，所以可以使用抽象工程的方法来解决问题。

```groovy
//1 首先继承FactoryBuilderSupport
class RobotBuilder extends FactoryBuilderSupport {
    //2 注册个节点的工厂方法
    {
        registerFactory('robot', new RobotFactory())
        registerFactory('forward', new ForwardMoveFactory())
        registerFactory('left', new LeftTurnFactory())
    }
}

/*
1，工厂扩展自AbstractFactory
2，newInstance方法用于创建对应的节点
3，isLeaf方法默认返回false，表示该节点可以有一个处理子节点的闭包，如果返回了true，还进入比表则会报错
4，onHandleNodeAttributes方法适合对属性执行特殊的处理操作，比如在ForwardMoveFactory把使用了的speed和duration去除掉，如果onHandleNodeAttributes
    返回true，会把留下来的属性填给节点
5，setChild会在父节点的工厂上调用，比如ForwardMove的父节点Robot的RobotFactory种，其setChild方法的child就是ForwardMove的实例
6，setParent会在子节点处理完成时调用
7，onNodeCompleted方法会在节点处理完成是调用
 */

class RobotFactory extends AbstractFactory {

    @Override
    Object newInstance(FactoryBuilderSupport builder, Object name, Object value, Map attributes) throws InstantiationException, IllegalAccessException {
        return new Robot(name: value)
    }

    @Override
    void onNodeCompleted(FactoryBuilderSupport builder, Object parent, Object node) {
        super.onNodeCompleted(builder, parent, node)
    }

    @Override
    void setParent(FactoryBuilderSupport builder, Object parent, Object child) {
        super.setParent(builder, parent, child)
    }

    @Override
    void setChild(FactoryBuilderSupport builder, Object parent, Object child) {
        parent.movements << child
    }
}


class ForwardMoveFactory extends AbstractFactory {

    @Override
    Object newInstance(FactoryBuilderSupport builder, Object name, Object value, Map attributes) throws InstantiationException, IllegalAccessException {
        new ForwardMove()//会自动将rotation: 20填充过ForwardMove
    }

    @Override
    boolean onHandleNodeAttributes(FactoryBuilderSupport builder, Object node, Map attributes) {
        if (attributes.speed && attributes.duration) {
            node.dist = attributes.speed * attributes.duration
            //使用调用了就去掉，不然会填充给实例
            attributes.remove('speed')
            attributes.remove('duration')
        }
        true
    }

    @Override
    boolean isLeaf() {
        true
    }
}

class LeftTurnFactory extends AbstractFactory {
    @Override
    boolean isLeaf() {
        true
    }

    @Override
    Object newInstance(FactoryBuilderSupport builder, Object name, Object value, Map attributes) throws InstantiationException, IllegalAccessException {
        new LeftTurn()
    }
}

class Robot {
    String name
    def movements = []

    void go() {
        println "Robot $name operation..."
        movements.each {
            println it
        }
    }
}

class ForwardMove {
    def dist

    @Override
    String toString() {
        "move distance .... $dist"
    }
}

class LeftTurn {
    def rotation

    String toString() {
        "turn left ...  $rotation degress"
    }
}

//--------------------
//运行代码
//--------------------

rb = new RobotBuilder()

def robot = rb.robot("iRobot") {
    forward(dist: 20)
    left(rotation: 20)
    forward(speed: 10, duration: 2)
}
robot.go()

/*
结果：

Robot iRobot operation...
move distance .... 20
turn left ...  20 degress
move distance .... 20

 */
```


FactoryBuilderSupport还有很多方便的方法，具体可以查看API文档

---
## 7 总结

本章学习了如果使用Groovy生成器，对于枯燥的处理生成XML，JSON生成器提供了执行它们的DSL语法，可以使用Groovy提供的生成器， 也可以亲自创建生成器。

