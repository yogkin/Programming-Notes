#  构建web项目

一个war文件用来绑定web组件。gradle提供了开箱即用的插件，用来组装war文件和将wab应用部署到本地Servlet容器中。

**使用war**

war插件扩展子java插件，为web应用部署和组装war包添加了约定和支持，应用了war插件时它会自动应用java插件，所以不再脚本中声明应用java插件也是可以的

**使用jetty插件**

gradle的jetty插件扩展了war插件，为部署一个web应用到嵌入式容器和运行web应用提供了对应的任务

>Jetty是一个纯粹的基于Java的网页服务器和Java Servlet容器。尽管网页服务器通常用来为人们呈现文档，但是Jetty通常在较大的软件框架中用于计算机与计算机之间的通信。Jetty作为Eclipse基金会的一部分，是一个自由和开源项目

**关于tomcat插件可以参考[gradle-tomcat-plugin](https://github.com/bmuschko/gradle-tomcat-plugin)**

---
## 1 脚本

```
    apply plugin: 'java'//应用java插件
    apply plugin: 'war'//应用war插件
    apply plugin: 'jetty'//应用jetty插件，jetty是一个轻量级的web服务器，与tamcat类似

    repositories {//指定仓库
        mavenCentral()
    }

    //声明依赖
    dependencies {
        providedCompile 'javax.servlet:servlet-api:2.5'//依赖的servlet
        runtime 'javax.servlet:jstl:1.1.2'
    }
    //jetty默认的端口是8080，而默认的路径是项目的jsp文件，使用下面方式可以改变默认配置
    //配置后就可以使用http://localhost:9090/todo来访问web应用
    jettyRun {
        httpPort = 9090  
        contextPath = 'todo'
    }
```

使用`gradle jettyRun`来启动web容器：

配置前

![](images/gradle_web_jettyrun1.png)

配置后

![](images/gradle_web_jettyrun2.png)


**依赖说明：**

使用web应用所需要的类库并不是java标准版本的一部分，所以需要声明外部依赖，war插件提供了两个新的依赖configuration


- providedCompile 表示该依赖在编译时需要，但是由运行时环境提供，这里的运行时环境是jetty，结果就是被标记为provided的依赖不会被打包到war文件中
- rutime 像jstl库这样的依赖，在编译时不需要，但是运行时需要，它们会成为war的一部分，所以声明为runtime的


---
## 2 标准的web项目布局

webapp默认下项目源码目录是:``src/main/java`,`src/main/webapp`

```
    └─build.gradle
    │
    └─src
        └─main
            ├─java                 //存放java代码
            │  └─com
            │      └─manning
            │          └─gia
            │              └─todo
            │                  ├─model
            │                  │      ToDoItem.java
            │                  │
            │                  ├─repository
            │                  │      InMemoryToDoRepository.java
            │                  │      ToDoRepository.java
            │                  │
            │                  └─web
            │                          ToDoServlet.java
            │
            └─webapp
                ├─css
                │      base.css
                │      bg.png
                │
                ├─jsp                   //jsp文件目录
                │      index.jsp
                │      todo-list.jsp
                │
                └─WEB-INF
                        web.xml   //web应用描述文件
```

---
## 3 定制war插件

对于非标准的web项目布局，可以通过war插件来定制化项目：

war暴露了`webAppDirName`约定属性，默认是`src/main/webapp`，通过from方法就可以有选择性的将需要的目录添加到war文件中：

```
    apply plugin: 'java'
    apply plugin: 'war'
    apply plugin: 'jetty'

    repositories {
        mavenCentral()
    }

    dependencies {
        providedCompile 'javax.servlet:servlet-api:2.5'
        runtime 'javax.servlet:jstl:1.1.2'
    }

    webAppDirName = 'webfiles'   //改变web应用的源代码目录

    war {              //将css和jsp目录添加到war文件的跟目录下
        from 'static'
    }
```

现在的结构为：

```
    ├─build.gradle
    │
    ├─src
    │  └─main
    │      └─java
    │          └─com
    │              └─manning
    │                  └─gia
    │                      └─todo
    │                          ├─model
    │                          │      ToDoItem.java
    │                          │
    │                          ├─repository
    │                          │      InMemoryToDoRepository.java
    │                          │      ToDoRepository.java
    │                          │
    │                          └─web
    │                                  ToDoServlet.java
    │
    ├─static
    │  └─css
    │          base.css
    │          bg.png
    │
    └─webfiles
        ├─jsp
        │      index.jsp
        │      todo-list.jsp
        │
        └─WEB-INF
                web.xml
```



