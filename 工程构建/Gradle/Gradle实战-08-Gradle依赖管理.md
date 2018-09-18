# Gradle依赖管理

---
## 1 依赖管理概述

几乎所有的基于JVM的软件项目都需要依赖外部类库来重用现有功能。

### 不完善的依赖管理技术

手动管理

### 自动化依赖管理的重要性


- 知道依赖确切的版本
- 管理传递性依赖：尝试手动确定对一个特定类库的所有依赖性传递是非常困难的。通常在类库的文档中这种信息无处可寻，结果就是遇到像编译错误和运行是类加载错误的问题，比如流行的Java开发栈Spring和Hibernate的组合可以很容易的从一开始就引入20多个附加的类库。


### 使用自动化依赖管理


在Java世界里，支持声明和自动化依赖管理的占主要地位的两个项目是：**Apache Ivy和Maven**，在Ivy和Maven中依赖配置是通过**xml**描述符来表达的，配置有两部分组成：**依赖标识符加各自的版本和二进制仓库的地址**，依赖管理器解析这些信息并自动从目标仓库下载依赖到本地，类库中可以定义传递性依赖作为元数据的一部分。依赖管理器在获取依赖的过程中能分析这些信息并解析依赖性。Maven仓库中的依赖通过项目对象模型(POM `project object model`)来表示。**依赖管理器让你摆脱了手动复制和组织jar文件的的困扰**。

### 自动化依赖管理的挑战

- 潜在不可靠的中央托管仓库：最广泛使用的仓库之一是MavenCentral，如果构建只依赖于MavenCentral你的系统就可能发生单点故障。
- 坏元数据和依赖缺失

**使用Gradle可以很好的解决这些问题**

---
## 2 通过示例学习依赖管理

以依赖项目开源项目**Cargo**为例，Cargo提供了支持将Web应用程序部署到各种Servlet容器和应用程序服务器中的功能。

### 2.1 非常重要的概念：依赖配置

java插件引入了各种标准的配置来定义java构建生命周期所应用的依赖，例如通过`compile`配置可以添加编译产品源代码所需要的依赖。**compile就是java插件提供的配置中的一个**

#### 理解配置API表示


配置可以直接在项目的根级别目录添加和访问，我们可以声明自己的配置也可以使用插件所提供的配置。每个项目都有一个`ConfigurationContainer`类的容器来管理相应的配置，配置在行为方面可以表现得很灵活，我们可以控制依赖解决方案中是否包含传递性依赖，定义解决策略(例如工件版本冲突)，甚至可以配置扩展。


**Interface：Project**

```
       void configurations(Closure configureClosure);
       ConfigurationContainer getConfigurations();
```

**Interface：ConfigurationContainer:**

```
        add(String name);
        add(String name,Closure c);
        Configuration getByName(String name)
        Configuration getByName(String name, Closure configureClosure)
```

**Interface：Configuration:**

```
        ResolutionStrategy getResolutionStrategy();
        Configuration resolutionStrategy(Closure closure);
        State getState();
        String getName();
        Configuration setVisible(boolean visible);
        boolean isTransitive();
        Configuration setTransitive(boolean t);
        String getDescription();
        Configuration setDescription(String description);
```

一个Project对应ConfigurationContainer，一个ConfigurationContainer包含0或多个Configuration，**Java插件提供了6个现成的配置**:

- compile
- runtime
- testCompile
- testRuntime
- archives
- default

应该针对不同的需求使用不同的配置，比如testCompile配置的依赖不会被打包到发布包中，在运行时将不必要的类库添加到发布包中可能导致不可预见的副作用


#### 自定义配置

添加cargon配置

```
    configurations{
            cargo{//通过名称定义一个新的配置
                description = 'classpath for cargo ant tasks.'//设置配置的的描述信息和可见性
                visible = false//不要让不必要的配置涉及到其他项目
            }
    }
```


这个cargo就像java提供的`compile`一样，也是一个配置，会被映射为一个Configuration实例，**而可以通过Configuration提供的各种api来决定如果管理相应
配置的依赖。**

运行`gradle dependencies`：

```
    ------------------------------------------------------------
    Root project
    ------------------------------------------------------------


    archives - Configuration for archive artifacts.//java提供的配置
    No dependencies


    cargo - classpath for cargo ant tasks.
    No dependencies

    ......省略其他配置

```

Gradle dependencies task展示了project中的各种配置。为项目添加一个配置后，可以直接通过名称来访问

### 2.2 声明依赖

#### 依赖的类型

- 外部模块依赖：依赖仓库中的外部依赖库，包括它提供的元数据
- 项目依赖：依赖其他Gradle项目
- 文件依赖：依赖文件系统中的一系列文件
- 客户端模块依赖：依赖仓库中的外部依赖库，具有声明元数据的能力
- Gradle运行时依赖：依赖GradleAPI扩展封装Gradle运行时的类库

#### 理解依赖API


每个项目都有依赖管理器实例，由`DepecndencyHandler`接口来表现，通过使用项目的getDependencies()方法来获取引用，每一个依赖都是Dependency类型的一个实例，使用`gruop、name、version、classifier`属性明确的标识一个依赖。


**Interface：Project**

```
        void dependencies(Closure configureClosure);
        DependencyHandler getDependencies();
```

**Interface：DependencyHandler**

```
        Dependency add(String configurationName, Object dependencyNotation);
        Dependency add(String configurationName, Object dependencyNotation, Closure configureClosure);
        Dependency create(Object dependencyNotation);
        Dependency create(Object dependencyNotation, Closure configureClosure);
        Dependency module(Object notation);
        Dependency module(Object notation, Closure configureClosure);
        Dependency project(Map<String, ?> notation);
        Dependency gradleApi();
        Dependency localGroovy();
```

**Interface：Dependency**

```
        String getGroup();
        String getName();
        String getVersion();
        Dependency copy();
        boolean contentEquals(Dependency dependency);
```

一个Project对应DependencyHandler，一个ConfigurationContainer包含0或多个Dependency。当我们在dependencies代码块中添加` cargo 'org.codehaus.cargo:cargo-ant:1.3.1`时，其实就是像`DepecndencyHandler`add一个`Depecndency`。

#### 外部依赖模块

在Gradle术语中，外部类库通常以Jar文件的形式存在，被称为外部模块依赖，这种类型依赖在仓库中能够通过属性明确的标识。

#### 依赖的属性


- grouop  用来标识一个组织、公司或者项目。group可以使用点标记法，比如Hibernate库的group是`org.hibernate`
- name  一个工件的名称，唯一的描述了依赖,比如Hibernate库的name是`hibernate-core`
- version  一个库可以由很多个版本，版本字符串一般包含主版本和次版本
- classifier  有时一个工件也定义了另一个属性，classifier，用来区分具有相同的grouop、name、version属性的工件。Hibernate库没有定义classifier属性。

#### 依赖标记


依赖的标记分为两种：1 使用map形式给出属性名称及其值；2 以字符串形式使用快捷标记，用冒号分隔每一个属性。

```
    //如果项目需要处理大量的依赖，那么将常用的依赖作为扩展属性是很有帮助的
    ext.cargoGroup = 'org.codehaus.cargo'
    ext.cargoVersion = '1.3.1'


    dependencies {
        //方式1
        cargo group: cargoGroup, name: 'cargo-core-uberjar', version: cargoVersion
        //方式2
        cargo "$cargoGroup:cargo-ant:$cargoVersion"
    }
```

gradle不会为我们选择默认的仓库，没有没有配置仓库，构建是会导致错误的。所以需要通过repositories来声明使用的仓库。

```
    repositories {
        //mavenCentral是内置的一个仓库
        mavenCentral()
    }
```

#### 检查依赖报告

helps任务组中有一个`dependencies task`，用来显示完整的依赖树，包括顶层依赖或依赖传递

比如上面cargo的依赖图为：`cmd: gradle dependencies`

```
    Parallel execution with configuration on demand is an incubating feature.
    :dependencies


    ------------------------------------------------------------
    Root project
    ------------------------------------------------------------


    cargo - Classpath for Cargo Ant tasks.
    +--- org.codehaus.cargo:cargo-core-uberjar:1.3.1       //构建中声明的顶层依赖
    |    +--- commons-discovery:commons-discovery:0.4
    |    |    \--- commons-logging:commons-logging:1.0.4
    |    +--- jdom:jdom:1.0
    |    +--- dom4j:dom4j:1.4               //指明版本请求和所选择的版本来解决类库的版本冲突
    |    |    +--- xml-apis:xml-apis:1.0.b2 -> 1.3.03
    |    |    +--- jaxen:jaxen:1.0-FCS
    |    |    +--- saxpath:saxpath:1.0-FCS
    |    |    +--- msv:msv:20020414
    |    |    +--- relaxngDatatype:relaxngDatatype:20020414
    |    |    \--- isorelax:isorelax:20020414
    |    +--- jaxen:jaxen:1.0-FCS
    |    +--- saxpath:saxpath:1.0-FCS
    |    +--- msv:msv:20020414
    |    +--- relaxngDatatype:relaxngDatatype:20020414
    |    +--- isorelax:isorelax:20020414
    |    +--- com.sun.xml.bind:jaxb-impl:2.1.13
    |    |    \--- javax.xml.bind:jaxb-api:2.1
    |    |         +--- javax.xml.stream:stax-api:1.0-2
    |    |         \--- javax.activation:activation:1.1
    |    +--- javax.xml.bind:jaxb-api:2.1 (*)
    |    +--- javax.xml.stream:stax-api:1.0-2
    |    +--- javax.activation:activation:1.1
    |    +--- org.apache.ant:ant:1.7.1
    |    |    \--- org.apache.ant:ant-launcher:1.7.1
    |    +--- org.apache.ant:ant-launcher:1.7.1
    |    +--- xerces:xercesImpl:2.8.1
    |    |    \--- xml-apis:xml-apis:1.3.03
    |    +--- xml-apis:xml-apis:1.3.03
    |    \--- commons-logging:commons-logging:1.0.4
    \--- org.codehaus.cargo:cargo-ant:1.3.1
         \--- org.codehaus.cargo:cargo-core-uberjar:1.3.1 (*)


    (*) - dependencies omitted (listed previously)//标记在依赖图中排除的传递性依赖


    BUILD SUCCESSFUL
    //带有星号的依赖都被排除了
```

>如果是子项目可使用`gradle dependencies -pSubModuleName`来执行task


针对版本冲突gradle默认的做法是选择最新的版本，比如上面dom4j的版本选择了`1.3.03`


#### 排除依赖传递性


通过`exclude`可以排除某一个依赖的传递


通过`transitive`可以排除所有的依赖传递

```
    dependencies {
        cargo('org.codehaus.cargo:cargo-ant:1.3.1') {
          exclude group: 'xml-apis', module: 'xml-apis'//通过快捷或者map形式来声明排除依赖
        }
        cargo 'xml-apis:xml-apis:2.0.2'
    }
```

排除属性和常用的依赖标记略有不同，可以使用`group和/或module`属性，version在此处是不可用的，因为不允许排除某一个特定的版本。

```
    dependencies {
        cargo('org.codehaus.cargo:cargo-ant:1.3.1') {
            transitive = false//排除所有的依赖
        }
        // Selectively declare required dependencies
        cargo 'org.codehaus.cargo:cargo-core-uberjar:1.3.1'
    }
```

#### 动态声明版本


通过一个加号可以动态的声明版本，gradle会为你选择依赖的最新版本

```
    dependencies {
        cargo 'org.codehaus.cargo:cargo-ant:1.+'
    }
```

这里gradle会选择最新的1.x版本。但是最好少用或者不用动态版本，可靠的和可重复的构建是最重要的，使用动态版本必然有不稳定的隐患。


#### 文件依赖


可以在dependencies中声明对本地jar或其他库的依赖：

```
    task copyDepends(type: Copy){//拷贝依赖到指定目录
        from configurations.cargo.asFileTree
        into "${System.properties['user.home']}/libs/cargo"
    }


    dependencies {
        cargo fileTree(dir: "${System.properties['user.home']}/libs/cargo", include: '*.jar')//声明依赖的位置
    }
```


### 2.3 配置和使用仓库


Gradle支持的仓库：


- Maven：常用的有jcenter和mavenCentral，本地文件系统和远程服务器的Maven仓库，或者预配置的MavenCentral
- Ivy：本地文件系统或者远程服务器中的Ivy仓库，具有特定的结构模式
- 扁平的目录：本地文件系统中的仓库，没有元数据


#### 理解仓库API


在项目中定义仓库的关键是`RepositoryHandler`接口，其提供了各种类型仓库的方法。这些方法在repositories配置块中被调用，可以声明多个仓库，gradle会按照声明的顺序来检查仓库，仓库提供了依赖优先原则，对于特定的依赖后续的仓库声明不会进一步检查。

**Interface：Project**

```
        RepositoryHandler getRepositories();
        void repositories(Closure configureClosure);
```

**Interface：RepositoryHandler**

```
        FlatDirectoryArtifactRepository flatDir(Map<String, ?> args);
        MavenArtifactRepository jcenter();
        MavenArtifactRepository mavenCentral(Map<String, ?> args);
        MavenArtifactRepository mavenCentral();
        MavenArtifactRepository mavenLocal();
        IvyArtifactRepository ivy(Closure closure);
        MavenArtifactRepository maven(Closure closure);
```

**Interface：ArtifactRepository**

```
        String getName();
        void setName(String name);
       扩展有：
            MavenArtifactRepository Maven仓库
            IvyArtifactRepository Ivy仓库
            FlatDirectoryArtifactRepository 扁平目录仓库
```

一个Project对应一个RepositoryHandler，一个RepositoryHandler可以添加0个或多个ArtifactRepository。**Gradele支持Maven，Ivy，扁平的目录三种仓库的实现**




#### Maven仓库


Maven仓库是最常用的仓库类型之一，类库以jar文件形式表示，元数据通过xml表示，并且使用POM(`Project Object Model`)文件(可以在Gradle缓存目录中找到)描述了类库修改信息及其传递性依赖。

```
    ├─cargo-ant
    │  └─1.3.1
    │      └─cargo-ant-1.3.1.pom  //元数据
    │
    ├─cargo-core
    │  └─1.3.1
    │      └─cargo-core-1.3.1.pom
```

pom.xml:


```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.codehaus.cargo</groupId>
    <artifactId>cargo-extensions-ant</artifactId>
    <version>1.3.1</version>
  </parent>
  <artifactId>cargo-ant</artifactId>
  <name>Cargo Ant tasks</name>
  <packaging>jar</packaging>
  <description>Ant tasks for Cargo</description>
  <dependencies>
    <dependency>
      <groupId>org.codehaus.cargo</groupId>
      <artifactId>cargo-core-uberjar</artifactId>
      <version>${cargo.core.version}</version>
    </dependency>


    <dependency>
      <groupId>org.codehaus.cargo</groupId>
      <artifactId>cargo-core-api-container</artifactId>
      <version>${cargo.core.version}</version>
      <type>test-jar</type>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

RepositoryHandler提供了两个方法来运行你定义预配置的Maven仓库：
- mavenCentral()方法用来将MavenCentral引用添加到一系列仓库中
- mavenLocal方法用来在文件系统中管理一个本地的maven仓库

**添加预定义的MavenCentral仓库：**

```
     repositories {
            mavenCentral()
     }
```

**添加预配置的本地Maven仓库：**

```
    repositories {
             mavenLocal()
    }
```

`<UserHome>/.m2/repository`目录是默认的可用的本地仓库，**何时使用本地仓库**：开发阶段的类库，防止每一次小小的改动都需要将类库发布到远程Maven仓库。


**添加一个自定义的Maven仓库**

```
    repositories {
        maven {
            name 'Custom Maven Repository'
            url 'http://repository-gradle-in-action.forge.cloudbees.com/release/'
        }
    }
```

#### Ivy仓库


Maven仓库的工件必须以一种固定的布局存储，任何结构偏差都有可能导致依赖关系发生错误，Ivy提出支持完全自定义的默认布局。


#### 扁平的目录仓库


flat目录仓库是最简单的最基本的仓库形式，他在文件系统中就要一个目录，只包含jar文件，没有元数据，当声明依赖时只能使用name和version属性，不能使用group属性：

```
    dependencies {
        cargo name: 'activation', version: '1.1'
        cargo name: 'ant', version: '1.7.1'
        cargo name: 'ant-launcher', version: '1.7.1'
        cargo name: 'cargo-ant', version: '1.3.1'
        cargo name: 'cargo-core-uberjar', version: '1.3.1'
        cargo name: 'commons-discovery', version: '0.4'
        cargo name: 'commons-logging', version: '1.0.4'
        cargo name: 'dom4j', version: '1.4'
        cargo name: 'isorelax', version: '20020414'
        cargo ':jaxb-api:2.1', ':jaxb-impl:2.1.13', ':jaxen:1.0-FCS', ':jdom:1.0', ':msv:20020414',
                ':relaxngDatatype:20020414', ':saxpath:1.0-FCS', ':stax-api:1.0-2', ':xercesImpl:2.8.1',
                ':xml-apis:1.3.03'
    }


    repositories {
        flatDir(dir: "${System.properties['user.home']}/libs/cargo", name: 'Local libs directory')
    }
```

#### 搭建自己的内部仓库


可以选择下面两个框架搭建私人仓库：


- Sonatype Nexus
- JForg Artifactory


---
## 3 理解本地依赖缓存

声明依赖后，Gradle会从仓库中下载依赖，并将它们缓存到本地，后续的构建都将重用这些工件，

### 分析缓存结构

```
    task printDependencies << {
        configurations.getByName('cargo').each { dependency ->
            println dependency
        }
    }
```

运行这个task可以知道cargo的缓存都被存在哪里：

```
    D:\Android\.gradle\caches\modules-2\files-2.1\org.codehaus.cargo\cargo-ant\1.3.1\a5a790c6f1abd6f4f1502fe5e17d3b43c017e281\cargo-ant-1.3.1.jar
    D:\Android\.gradle\caches\modules-2\files-2.1\org.codehaus.cargo\cargo-core-uberjar\1.3.1\3d6aff857b753e36bb6bf31eccf9ac7207ade5b7\cargo-core-uberjar-1.3.1.jar
    D:\Android\.gradle\caches\modules-2\files-2.1\commons-discovery\commons-discovery\0.4\9e3417d3866d9f71e83b959b229b35dc723c7bea\commons-discovery-0.4.jar
    D:\Android\.gradle\caches\modules-2\files-2.1\jdom\jdom\1.0\a2ac1cd690ab4c80defe7f9bce14d35934c35cec\jdom-1.0.jar
    D:\Android\.gradle\caches\modules-2\files-2.1\dom4j\dom4j\1.4\8decb7e2c04c9340375aaf7dd43a7a6a9b9a46b1\dom4j-1.4.jar
    .......省略很多依赖
```

在我的机器上，GradleUserHome是`D:\Android\.gradle`。cargo的所有jar文件的存储路劲为：`$GradleUserHome/caches/modules-2/files-2.1`,缓存的目录结构：



```
    D:\ANDROID\.GRADLE\CACHES\MODULES-2\FILES-2.1\ORG.CODEHAUS.CARGO
    ├─cargo-ant
    │  └─1.3.1
    │      ├─a5a790c6f1abd6f4f1502fe5e17d3b43c017e281
    │      │      cargo-ant-1.3.1.jar           //依赖的jar
    │      │
    │      └─cf13fc6a9e07971c96f0e2cc3c726fe6473eb926
    │              cargo-ant-1.3.1.pom          //依赖的元数据
    │
    ├─cargo-core
    │  └─1.3.1
    │      └─adb2f62f8a9169100a8b9afa629da26868d79f58
    │              cargo-core-1.3.1.pom
    │
    ├─cargo-core-uberjar
    │  └─1.3.1
    │      ├─3d6aff857b753e36bb6bf31eccf9ac7207ade5b7
    │      │      cargo-core-uberjar-1.3.1.jar
    │      │
    │      └─8015317c054befa179f963721874609e60b2e57e
    │              cargo-core-uberjar-1.3.1.pom
    │
    ├─cargo-extensions
    │  └─1.3.1
    │      └─d8e1aa7bf944a224ffc13633dee5a4b39df5ccc6
    │              cargo-extensions-1.3.1.pom
```

Gradle的缓存提高了构建的性能，通过比较本地和远程的校验来检查仓库中的工件是否发生变化。如果声明了远程依赖，Gradle就不得不检查仓库是否发生变化，可以通过命令选项`--offset`告诉gradle在离线模式下进行构建，这样gradle就不会检查远程依赖了。

---
## 4 解决依赖冲突

### 应对版冲突

Gradle不会自动通知你项目遇到了版本冲突，不断的运行依赖报告来找出冲突并不是一个好方法，我们可以改变默认的策略，当遇到版本冲突时，让构建失败：

```
    configurations.cargo.resolutionStrategy {
        failOnVersionConflict()
    }
```

失败对于调试项目很有帮助。

```
    cmd：gradle deployToLocalTomcat
    Parallel execution with configuration on demand is an incubating feature.
    :deployToLocalTomcat FAILED


    FAILURE: Build failed with an exception.


    * Where:
    Build file 'C:\Users\Administrator\Desktop\GradleInActionSource\gradle-in-action-source\chapter05\cargo-dependencies-fail-on-version-conflict\build.gradle' line: 10


    * What went wrong:
    Execution failed for task ':deployToLocalTomcat'.
    > Could not resolve all dependencies for configuration ':cargo'.
       > A conflict was found between the following modules:
          - xml-apis:xml-apis:1.3.03
          - xml-apis:xml-apis:1.0.b2


    * Try:
    Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.


    BUILD FAILED
```

### 强制指定一个版本

```
    ext.cargoGroup = 'org.codehaus.cargo'
    ext.cargoVersion = '1.3.1'

    dependencies {
        cargo "$cargoGroup:cargo-ant:$cargoVersion"
    }

    repositories {
        mavenCentral()
    }

    configurations.cargo {
        resolutionStrategy.force "$cargoGroup:cargo-ant:1.3.0"
    }
    //或者
     configurations.all {//强制所有配置
        resolutionStrategy.force "$cargoGroup:cargo-ant:1.3.0"
    }
```

运行构建：

```
    cargo - Classpath for Cargo Ant tasks.
    \--- org.codehaus.cargo:cargo-ant:1.3.1 -> 1.3.0
```

现在cargo被强制指定了1.3.0的版本


### 使用依赖观察报告

gradle提供了不同类型的依赖报告：依赖观察报告，他解释了依赖图中依赖时如何选择以及为什么，运行这个报告需要提供两个参数,配置名称(默认是compile)和依赖本身：

执行`gradle -q dependencyInsight --configuration cargo --dependency xml-apis:xml-apis`命令：


```
    gradle -q dependencyInsight --configuration cargo --dependency xml-apis:xml-apis
    xml-apis:xml-apis:1.3.03 (conflict resolution)//显示了一个特定依赖被选择的原因
    +--- org.codehaus.cargo:cargo-core-uberjar:1.3.1
    |    +--- cargo
    |    \--- org.codehaus.cargo:cargo-ant:1.3.1
    |         \--- cargo
    \--- xerces:xercesImpl:2.8.1
         \--- org.codehaus.cargo:cargo-core-uberjar:1.3.1 (*)


    xml-apis:xml-apis:1.0.b2 -> 1.3.03//显示了所需要和选择的特定依赖的版本
    \--- dom4j:dom4j:1.4
         \--- org.codehaus.cargo:cargo-core-uberjar:1.3.1
              +--- cargo
              \--- org.codehaus.cargo:cargo-ant:1.3.1
                   \--- cargo


    (*) - dependencies omitted (listed previously)
```


观察报告展示的视图与普通的依赖报告正好相反。显示了依赖从特定依赖开始到配置。(从下往上看)


### 刷新缓存


为了避免一遍又一遍的为特定的依赖去访问仓库，gradle提供了特定的缓存策略，这种策略主要针对SNAPSHOT版本和使用动态版本模式声明的依赖，一旦获取了依赖，它们会被存储24小时，到期后，gradle会再次检查仓库，如果构建发生变化gradle会下载一个最新版本的工件。


使用命令选项`--refresh-dependencies`可以手动刷新缓存中的依赖，强制检查配置仓库中的工件版本是否发生变化。


还可以该白默认的策略：

```
    //如果总想要获取org.codehaus.cargo:cargo-ant:1.+的最新版本，可以设置动态依赖版本超时为0秒：
    configurations.cargo.resolutionStrategy {
          cacheDynamicVersionsFor 0, 'sencends'
    }


    //如果不许缓存要给外部模块的SNAPSHOT版本，下面配置用于指定不缓存SNAPSHOT版本的依赖
    configurations.cargo.resolutionStrategy {
         cacheChangingModulesFor 0 , 'senends'
    }
```

