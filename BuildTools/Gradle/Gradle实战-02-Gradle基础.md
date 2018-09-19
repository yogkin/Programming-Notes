# Gradle基础

---
## 1 安装Gradle


- 下载gradle，然后配置好Gradle环境变量(与Java环境变量类似)
- 使用`gradle -v` 查看gradle版本和验证gradle是否安装成功

### 设置Gradle的JVM选项

和Java一样，Gradle也支持由环境变量`JAVA_OPTS`设置的JVM选项,如果想要传递特定的参数给Gradle运行时，则使用`GRADLE_OPTS`环境变量，例如：

```
        GRADLE_OPTS="-Xmx1024m"//设置默认的最大堆内存为1GB
```

当然更好的方式是将变量添加到`%GRADLE_HOME%/bin`目录下的Gradle启动脚本中。


---
## 2 开始使用Gradle


每一个Gradle构建都是以一个脚本开始的，默认的脚本名称为`build.gradle`，在命令行执行gradle命令时，Gradle会去寻找名称是`build.gradle`的文件，如果找不到，Gradle会提示一个帮助信息。

### 2.1 Task

在Gradle脚本中定义一个独立的原子性工作，在Grale词汇中这称为**task**。


```
    //定义一个helloWorld的task
    task helloWorld {
        doLast{
            println "hello World"
        }
    }


    // << 操作符重载，映射到doLast闭包
    task startSession <<{
        chant()
    }


    def chant(){
        ant.echo(message:"Repeat after me....")//隐含对ant任务的使用
    }


    //动态的添加三个任务
    3.times { i ->
            task "yayGradle$i"{
                println "Gradle rocks$i"
            }
    }


    //定义任务依赖
    yayGradle0.dependsOn startSession
    yayGradle2.dependsOn yayGradle1 , yayGradle0
    task groupTherapy(dependsOn:yayGradle2)
```


- dependOn是Task的一个方法，用来声明任务间的依赖
- Gradle与Ant由很好的集成，每一个脚本都带有一个ant属性，它赋予了直接访问呢ant任务的能力。


执行任务：


```
    gradle -q helloWorld(也可以使用gradle -q hW,支持以任务名称的首字母缩写来指定任务)
    gradle groupTherapy
```


`-q`为命令选项，下面介绍。


### 2.2 使用Gradle命令行


使用Gradle命令行选项和帮助任务会让工作变得更加高效


- `gradle tasks`：用于列出所有的任务，Gradle会根据分组列出列出所有的任务，gradle提供了任务分组的概念，可以看作它是一个task的集群
- `gradle tasks --all`：用于列出更加详细的任务信息，包括任务之间的依赖关系


#### 任务的执行


通过`gradle task1name task2name`可以运行一个或者多个task，任务通常只会运行一次


**可以使用任务名称的缩写来运行任务**，比如task名为`groupTherapy`可以通过命名`gradle gT`来运行，但是当缩写的名称不能够确定唯一的task时，运行将会失败。**名称缩写注意大小写**




#### 在运行时排除一个task


假设需要在运行时排除task `yayGradle0`可以使用命名:


```
    `gradle groupTherapy -x yayGradle0`//表示运行任务groupTherapy，但是排除任务yayGradle0
```


#### 命名行的选项


Gradle命名由很多选项，并且支持同时定义一个或多个选项，常用的命令行选项为:


##### 命令选项


```
    -? , -h , --help:打印出所有可用的命令选项，包括描述信息
    -b , --build-file:gradle构建脚本的默认名称为build.gradle,使用此命令可以执行一个特定名称的构建脚本，比如gradle -b test.gradle
    --offline:以离线的模式运行够建，仅仅在本地缓存中检查依赖是否存在
```


##### 参数选项


```
    -D , --system-prop:Gradle是以JVM进行运行的，和所有的Java进程一样，你可以提供一个系统参数，就像-Dkey=value这样
    -p , --project-prop:项目参数是构建脚本中可用的变量，可以使用这个选项执行向构建脚本中传入参数(比如-Pkey=value,表示传入key为value的变量)
```


##### 日志选项


```
    -i , --info:在默认的设置中，gradle不会提供大量的输出日志，通过这个选项可以将gradle的日志级别改变到INFO以获得更多的信息
    -s , --stacktrace:如果构建在运行中出错，你会想要知道错误是从哪里开始的，-s选项在有异常抛出的地方会打印出简短的堆栈信息
    -q , --quite:减少构建出错时打印出来的错误日志信息
```


##### 帮助任务


```
    -tasks：显示项目中所有可以运行的任务
    -properties：显示出项目中所有可用的属性，某些属性是由gradle的Project对象提供的，project对象是一个构建本质的表现形式，其他的属性都是用户定义的，要么来自与属性文件或者命令行选项，要么直接在构建脚本中定义
```


### 2.3 Gradle守护进程


守护进程以后台进程的方式运行gradle，一旦启动，Gradle命令就会在后续的构建中重用之前进程的守护进程，以避免创建进程的开销


使用方式为：

```
    gradle groupTherapy --daemon //使用守护进程运行，守护进行只会被创建一次，会在之后的三小时空闲时间后自动过期，启动后进程后运行任务默认都是使用守护进程
    gradle groupTherapy --no-dasmon//在启动守护进程后，任何一次任务若不行使用守护进程，则使用此命令
    gradle --stop用于停止守护进程
```

