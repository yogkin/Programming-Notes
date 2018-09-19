# Nexus仓库使用

---
## 1 下载并安装Nexus

下载地址:[点这里](https://www.sonatype.com/download-oss-sonatype)，我这里下载的是windows平台的2.x版本，下载之后解压即可。

**以管理员的身份**启动命令行，进入解压后的nexus的bin目录，然后依次运行下面命令：

```
nexus.bat install //安装服务
nexus.bat start //启动nexus
```

等待nexus启动之后可以在本地打开链接`http://127.0.0.1:8081/nexus/#welcome`,正常的话可以看到nexus的控制台了，然后需要登录，
nexus默认的账户是`admin `,密码是`admin123`,登录之后的界面为：

![](images/nexus_repository.png)

点击左边的Repositories选项，可以进入仓库管理界面，为了测试我们添加三个仓库：

1. 本地仓库(hosted),配置好仓库存储地址即可。
2. 代理仓库(proxy),这里选择阿里云的maven镜像，速度很不错的，地址为：http://127.0.0.1:8081/nexus/content/repositories/aliyun/
3. 仓库组(group),仓库组用于组合多个仓库。这里组合的是上面添加的两个仓库。

到这里nexus相关的工作就完成了。

---
## 2 使用nexus仓库

Android项目默认使用的远程仓库是jcenter，现在把仓库地址完全替换成刚刚搭建的私有仓库地址：

根项目的build.gralde为：

```groovy
buildscript {
    repositories {
        maven{
            url 'http://127.0.0.1:8081/nexus/content/groups/test_group/'//这里就是我们搭建的私有仓库组，其组合了远程仓库和本地仓库
        }
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.0'
    }
}

allprojects {
    repositories {
        maven{
            url 'http://127.0.0.1:8081/nexus/content/groups/test_group/'
        }
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

演示上传library到仓库，新建一个AndroidLibrary，写一个简单的工具类，然后上传到刚刚搭建的私有仓库：

```groovy

apply plugin: 'maven'

...省略android library的默认配置

//uploadArchive任务由maven插件提供
uploadArchives {
    //下面配置maven，具体参考：https://docs.gradle.org/current/userguide/maven_plugin.html
    repositories {
        mavenDeployer {
            //定义library发布到哪个仓库
            repository(url: 'http://127.0.0.1:8081/nexus/content/repositories/test_local/') {//这里的地址是在nexus中创建的本地地仓库
                authentication(userName: 'admin', password: 'admin123')
            }
            //定义项目的pom
            pom {
                setVersion '1.0'//版本
                setArtifactId 'toast'//工件id
                setPackaging 'arr'//打包方式
                setGroupId 'com.ztiany'//工件组
            }
            //仓库地址为groupId:artifactId:version
        }
    }
}
```
运行`gradle uploadArchives`即可上传library到私有仓库。

在app项目使用刚刚上传的library:
```
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:25.3.1'
    compile 'com.ztiany:toast:1.0'//添加library依赖
    testCompile 'junit:junit:4.12'
}
```

同步之后即可发现刚刚上传的library已经依赖成功了。

---
## 引用

- [搭建Maven私有仓库](https://pcyan.github.io/2017/04/08/use-nexus-to-create-private-maven-repo/)
- [Nexus文档](http://books.sonatype.com/nexus-book/3.0/reference/install.html#installation-archive)




