# 第十七章：应用编译时元编程

Groovy同时提供了运行时和编译时元编程。编译时元编程是一种高级特性，可以用于某些特殊情况，比如：框架或者工具的编写使用借助Groovy可以在编译时**分析和修改**程序的结构，比如无需修改源代码，既可以为线程安全、日志消息和代码的不同部分执行前置和后置操作等、对类进行检查。Groovy2中的类型检查器就实现了为一个抽象语法树(AST)变换，优雅的单元测试工具Spock使用这种范式来支持流畅的测试用例，还有代码检查工具CodeNarc。

---
## 1 在编译时分析代码

为方法和遍历取一个好名字并不是易事，但是只用一个字母来命名变量和方法显然时不对的，使用Groovy可以实现自动检查。要检查代码异味、需要编译代码结构，分析类名、方法名、字段名、参数名等。Groovy允许我们进入其编译阶段、一窥其处理的(AST)。AST树结构描述了程序中的表达式和语句，它是使用节点表示的，随着程序的进行其AST会被变换，包括节点插入、删除、重新排序。在编译过程中我们可以随着AST的演进对其进行检查。

要使用编译时元编程，必须理解和使用AST，使用groovyConsole的inspect AST工具可以显示代码的AST。


 Groovy的编译阶段分为：**初始化、解析、转换、语义分析、规范化、指令选择、class生成、输出、结束**。 首个合理的时机是在语义分析阶段之后，AST会此时被创建出来。

### 步骤1：编写我们的编译检查类，用于检查过短的方法名。

**package ztiany.chapter16.CodeCheck**
```
@GroovyASTTransformation(phase = CompilePhase.SEMANTIC_ANALYSIS)//声明在语义分析后执行
class CodeCheck implements ASTTransformation {

    @Override
    void visit(ASTNode[] nodes, SourceUnit source) {
        source.getAST().classes.each {
            classNode ->
                classNode.visitContents(new OurClassVisitor(source));
        }
    }

    class OurClassVisitor implements GroovyClassVisitor{
        SourceUnit mSourceUnit;
        OurClassVisitor(SourceUnit sourceUnitClass) {
            mSourceUnit = sourceUnitClass
        }

        @Override
        void visitClass(ClassNode node) {

        }

        @Override
        void visitConstructor(ConstructorNode node) {

        }

        def reportError(message, lineNumber, columnNumber) {
            mSourceUnit.addError(new SyntaxException(message,lineNumber,columnNumber))
        }

        @Override
        void visitMethod(MethodNode node) {
            if (node.name.size() == 1) {
                reportError "Make method name descriptive,avoid single letter names",node.lineNumber, node.columnNumber
            }
            node.parameters.each {
                parameter ->
                    if (parameter.name.size() == 1) {
                        reportError "Single letter parameters are morally wrong!",parameter.lineNumber,parameter.columnNumber
                    }
            }
        }

        @Override
        void visitField(FieldNode node) {

        }

        @Override
        void visitProperty(PropertyNode node) {

        }
    }
}
```

准备好`org.codehaus.groovy.transform.ASTTransformation`文件，将其放在`services`中，其中内容为:`ztiany.chapter16.CodeCheck`


### 步骤2：使用我们的参与编译过程

**1：把CodeCheck编译在classes文件夹中:**

`groovyc -d classes ztiany\chapter16\CodeCheck.groovy`

**2:假设当前准备文件目录如下：**

```
├─classes
│  └─ztiany
│      └─chapter16
│              CodeCheck$OurClassVisitor$_visitMethod_closure1.class
│              CodeCheck$OurClassVisitor.class
│              CodeCheck$_visit_closure1.class
│              CodeCheck.class
│
└─META-INF
    └─services
            org.codehaus.groovy.transform.ASTTransformation
```

**3:使用jar目录打包文件**

`jar -cvf CodeCheck.jar -C classes/ . META-INF`

**4:验证结果**


```
class Smelly {
    def canVote(a) {
        a>17?"you can vote":"you can not vote"
    }

    def p(instance) {
        println instance
    }
}
```

`groovyc -classpath CodeCheck.jar ztiany\chapter16\Smelly.groovy`

编译信息显式：
```
C:\Users\Administrator\Desktop\Files>groovyc -cp CodeCheck.jar Smelly.groovy
org.codehaus.groovy.control.MultipleCompilationErrorsException: startup failed:
Smelly.groovy: 6: Single letter parameters are morally wrong! @ line 6, column 17.
       def canVote(a) {
                   ^

Smelly.groovy: 10: Make method name descriptive,avoid single letter names @ line 10, column 5.
       def p(instance) {
       ^

2 errors
```


---
## 2 使用AST变换拦截方法调用

- [ ] todo

## 3 使用AST变换注入方法

- [ ] todo
