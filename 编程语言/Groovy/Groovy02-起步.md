# 第一章：Groovy起步

**Groovy特点**：

- Groovy的松散的Java 语法允许省略**分号**和**修改符**。
- 能够在运行时轻松地为对象指定新方法和属性。这一编程领域称为**元编程**
- 除非另行指定，Groovy 的所有内容都为 `public`。
- Groovy允许定义简单脚本，同时无需定义正规的 `class` 对象。
- Groovy 语法还允许省略变量类型。
- Groovy编译器会自动加上getter/setter方法，所以不需要写
- Groovy类的属性可以通过`.`来获取，看起来他们好像是在java中公用的一样，但是底层Groovy调用的是getter/seting方法
- 如果使用==比较两个类的实例，groovy底层会自动调用对象的wquals()方法，这个操作也避免了空指针。

**配置环境变量**：

Groovy是基于jvm的语言，所以需要依赖jdk。在官网下载安装包或者zip包，需要自行配置环境变量GROOVY_HOME(与JAVA_HOME一样的概念)。配置完毕后即可在终端进行检查；

```
    C:\Users\Administrator>groovy -v
    Groovy Version: 2.4.6 JVM: 1.8.0_73 Vendor: Oracle Corporation OS: Windows 10
```

终端打印出版本号，表示Groovy已经成功的运行了。


---
## 1 使用Groovysh

在终端输入`groovysh`用于打开Groovyshell，groovysh是以交互式方式尝试一些小型的groovy代码的好工具，需要注意的是，**当按下回车键groovysh就会编译并执行输入完的语句，打印代码执行过程中的所有语句，并打印这条语句的执行结果**，对于某些语句，Groovy会等待你输入完毕才执行，比如下面的为String添加isPalindrome函数。

groovysh示例：
```
    Math.sqrt(16)//执行Math.sqrt(16)，并打印其结果
    ===> 4.0
    groovy:000> println 'test'//执行println 'test'，
    test
    ===> null//打印println函数的结果，由于println函数返回null
    groovy:000> String.metaClass.isPalindrome = {
    groovy:001> delegate == delegate.reverse()
    groovy:002> }
    ===> groovysh_evaluate$_run_closure1@6e4566f1
    groovy:000> "MMMdMMM".isPalindrome()
    ===> true
```

在命令行中，如果不太确定命令，可以双击tab键，groovy会做出提示
```
    "dd".  //比如在终端输入"dd".后，双击tab键，groovy给出了下面提示

    capitalize()           center(                charAt(                chars()                codePointAt(
    codePointBefore(       codePointCount(        codePoints()           collectReplacements(   compareTo(
    compareToIgnoreCase(   concat(                contains(   //......
```

使用？可以查看groovysh的帮助命令：


```
                  :help      (:h ) Display this help message
                  ?          (:? ) Alias to: :help
                  :exit      (:x ) Exit the shell
                  :quit      (:q ) Alias to: :exit
                  import     (:i ) Import a class into the namespace
                  :display   (:d ) Display the current buffer
                  :clear     (:c ) Clear the buffer and reset the prompt counter.
                  :show      (:S ) Show variables, classes or imports
                  :inspect   (:n ) Inspect a variable or the last result with the GUI object browser
                  :purge     (:p ) Purge variables, classes, imports or preferences
                  :edit      (:e ) Edit the current buffer
                  :load      (:l ) Load a file or URL into the buffer
                  .          (:. ) Alias to: :load
                  :save      (:s ) Save the current buffer to a file
                  :record    (:r ) Record the current session to a file
                  :history   (:H ) Display, manage and recall edit-line history
                  :alias     (:a ) Create an alias
                  :set       (:= ) Set (or list) preferences
                  :register  (:rc) Registers a new command with the shell
                  :doc       (:D ) Opens a browser window displaying the doc for the argument
```

 使用inspect命名可以检测对象和变量


---
## 2 使用GroovyConsole

 在终端输入`groovyConsole`即可以打开groovyConsole，在groovyConsole中可以输入脚本代码，使用快捷键`control + R`既可以运行

---
## 3 在命令行使用Groovy

- 对于hello.groovy脚本文件，也可以直接使用`groovy hello`来运行hello中的代码
- 如果想直接运行一些语句，可以使用`groovy -e`命令，比如在终端输入:`groovy -e "println 123"`则会打印出123.