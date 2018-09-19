# 构建java项目

构建一个项目需要对进行源码编译、然后输出jar等操作。

Gradle插件作为驱动力能够自动化这些任务，**插件通过引入特定领域的约定和任务来扩展项目**，Java插件是gradle自动装载的一个插件。

java插件提供的功能：

- 编译
- 打包
- 为你的项目建议一个标准的项目布局，并且确保有意义，有顺序的执行任务

---
## 1 使用插件

    apply plugin: 'java'//使用此脚本即可应用java插件

默认情况下java插件约定：

- `src/mian/java` 下为java代码
- `src/test/java` 下为测试代码

一个标准的java项目布局如下：

```
    ├──build.gradle
    |
    ├──src
        ├─main
        │  ├─java
        │  │  ├─main
        │  │  │  ├──com   //项目代码
        │  │
        │  └─resources    //资源文件夹
        └─test
            ├─java        //测试代码
            └─resources   //测试资源文件夹
```



**java插件提供了一个默认的任务为`build`**，运行build命令如下：

![](images/gradle_build.png)

>某写任务被标记为`up-to-date`,表示改任务被跳过了，gradle的增量式构建支持自动鉴别不需要被运行的任务。

运行之后，会发现有一个build目录，里面包含任务的所有输出。


---
## 2 定制项目

```
    apply plugin: 'java'//应用java插件

    version = 0.1//指定项目版本，版本号会添加到生存的jar包名称上
    sourceCompatibility = 1.6//指定java编译兼容1.6

    sourceSets {//用于指定文件目录
        main {
            java {
                srcDirs = ['src']//指定java源码目录为src
            }
        }
        test {
            java {
                srcDirs = ['test']//指定测试源码目录为test
            }
        }
    }

    buildDir = 'out'//指定输出为out目录

    //将Main-Class头添加到jar文件的代码清单中，
    //现在jar文件中包含了主类头属性，可以通过java -jar xxx.jar来运行项目了
    jar {
        manifest {
            attributes 'Main-Class': 'com.manning.gia.todo.ToDoApp'
        }
    }
```

上面sourceSets中的`main`是固定写法，`main`表示项目代码，而`test`表示测试代码。

source映射到的api是`org.gradle.api.tasks.SourceSet`,从其源码也可以看出，为什么闭包需要写成`main`和`test`

```
    public interface SourceSet {
        /**
         * The name of the main source set.
         */
        String MAIN_SOURCE_SET_NAME = "main";//项目代码的名称

        /**
         * The name of the test source set.
         */
        String TEST_SOURCE_SET_NAME = "test";//测试代码名称
        ......
    }
```


---
## 3 配置依赖

**在Gradle中，依赖是由configuration分组的，java插件引入的一种configuration是`compile`**


在java的世界，依赖都是以jar文件的形式发布和使用的，许多类库都可以在远程仓库找到，仓库可以是一个文件系统或者一个中心服务器，Gradle要求项目至少定义一个仓库来使用依赖。

```
    repositories {
        mavenCentral()//配置对mavenCentral2仓库http://repol.maven.org/maven2 访问的快捷方式
    }

    dependencies {
        compile group: 'org.apache.commons', name: 'commons-lang3', version: '3.1'//声明一个依赖
    }
```


