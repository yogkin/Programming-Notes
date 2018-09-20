# MultiDex

---
## 1 方法数组越界

Android中单个dex文件所能包含的最大的引用数为65536，这包含AndroidFrameWork、依赖的jar包，以及本身包含的所有代码，65536是一个很大的数，一般一个简单的应用很难大大65536，但是对于比较大的应用就容易达到，当应用的方法数达到65536后，编译器就无法完成编译工作并抛出异常。**这是一个构建期异常。**

    UNEXPECED TOP_LEVEL EXCEPTION:
    com.android.dex.DexIndexOverflowException:method ID not in [0,oxffff]

>**关于64k限制**：65536。这个数字很重要，因为它代表的是单个 Dalvik Executable (DEX) 字节码文件内的代码可调用的引用总数。
Android 应用 (APK) 文件包含 Dalvik Executable (DEX) 文件形式的可执行字节码文件，其中包含用来运行您的应用的已编译代码。Dalvik Executable 规范将可在单个 DEX 文件内可引用的方法总数限制在 65,536，其中包括 Android 框架方法、库方法以及您自己代码中的方法。在计算机科学领域内，术语千（简称 K）表示 1024（或 2^10）。由于 65,536 等于 64 X 1024，因此这一限制也称为“64K 引用限制”。——官方文档


另外一种情况，有时方法数并没有达到65536，并且编译器也能正常完成工作，但是应用在低版本手机安装时异常终止，异常信息如下：

    E/dalvikvm:Optimization failed

dexopt是一个程序，应用安装时，系统会通过dexopt来优化dex文件，在优化的过程中dex采用一个固定大小的缓冲区来存储应用中所有方法的信息，这个缓冲区就是LinearAlloc，LinearAlloc缓冲区在新版本中的Android系统中大小是8M或者16M，但是在Android2.2和2.3中只有5M，当安装的apk中的方法数比较多时，尽管她还没有达到65536这个上限，但是它的存储空间仍然可能超过5M，这种情况下dexopt程序就会报错。

---
## 2 AndroidClassLoader

MultiDex解决方案可以解决放法数的65536问题，但是在学习MultiDex之前应该先补充Android的DexClassLoader的知识，以便更好的理解MultiDex解决方案

Java中的ClassLoader机制我们应该已经非常熟悉了，而Android只是在此基础之上扩展了新的ClassLoader，用于加载Dex中的Class。

>下面代码基于Android5.1

Android中的类加载器继承关系为

    ClassLoader
        |-->BaseDexClassLoader
             |-->PathClassLoader
             |-->DexClassLoader

ClassLoader的getSystemClassLoader方法被改写了

```java
private static ClassLoader createSystemClassLoader() {
        String classPath = System.getProperty("java.class.path", ".");
        return new PathClassLoader(classPath, BootClassLoader.getInstance());
}

public static ClassLoader getSystemClassLoader() {
    return SystemClassLoader.loader;
}

static private class SystemClassLoader {
       public static ClassLoader loader = ClassLoader.createSystemClassLoader();
}
```
可以看出现在返回的是`PathClassLoader`

PathClassLoader和DexClassLoader都继承自BaseDexClassLoader，只是用途不一样：

- PathClassLoader用于加载data/app目录下的apk文件，从这个目录可以看出，PahClassLoader主要用来加载已经安装了的apk
- DexClassLoader的类加载路径是在创建DexClassLoader对象时指定的，所以它可以加载任何目录下的Dex文件

PathClassLoader和DexClassLoader的并没有复杂的实现，功能全都继承自BaseDexClassLoader

```java
        public class PathClassLoader extends BaseDexClassLoader {
            public PathClassLoader(String dexPath, ClassLoader parent) {
                super(dexPath, null, null, parent);
            }
            public PathClassLoader(String dexPath, String libraryPath,
                    ClassLoader parent) {
                super(dexPath, null, libraryPath, parent);
            }
        }

        PathClassLoader的构造方法接受三个参数：
        - dexPath apk路径
        - libraryPath so的路径
        - parent 父加载器

        public class DexClassLoader extends BaseDexClassLoader {

            public DexClassLoader(String dexPath, String optimizedDirectory,
                    String libraryPath, ClassLoader parent) {
                super(dexPath, new File(optimizedDirectory), libraryPath, parent);
            }
        }

        其他三个参数都相同
        optimizedDirectory 用于存放odex，odex是对dex优化后的dex
```

BaseDexClassLoader的实现也比较简单，其内部有一个关键字段`private final DexPathList pathList;`，DexPathList中部分代码如下：

```java
    final class DexPathList {
        private static final String DEX_SUFFIX = ".dex";
    
        /** class definition context */
        private final ClassLoader definingContext;
    
        /**
         * List of dex/resource (class path) elements.
         * Should be called pathElements, but the Facebook app uses reflection
         * to modify 'dexElements' (http://b/7726934).
         */
        private final Element[] dexElements;
    
        /** List of native library directories. */
        private final File[] nativeLibraryDirectories;
    
        /**
         * Exceptions thrown during creation of the dexElements list.
         */
        private final IOException[] dexElementsSuppressedExceptions;

    }
```

DexPathList中就保存了ClassLoader对应的dex路径和so路径等。google的multidex方法关键就在于此。

### App中的ClassLoader

在一个app中的Application类中执行下面代码

            Log.d(TAG, "ClassLoader.getSystemClassLoader():" + ClassLoader.getSystemClassLoader());

            Log.d(TAG, "getClassLoader():" + getClassLoader());

            Log.d(TAG, "MainActivity.class.getClassLoader():" + MainActivity.class.getClassLoader());

打印的结果为：

```
ClassLoader.getSystemClassLoader():
   dalvik.system.PathClassLoader[DexPathList[[directory "."],nativeLibraryDirectories=[/vendor/lib64, /system/lib64]]]

getClassLoader():
   dalvik.system.PathClassLoader[DexPathList[[zip file "/data/app/com.ztiany.gradlemultidex-1/base.apk"],nativeLibraryDirectories=[/data/app/com.ztiany.gradlemultidex-1/lib/arm64, /vendor/lib64, /system/lib64]]]

MainActivity.class.getClassLoader():
   dalvik.system.PathClassLoader[DexPathList[[zip file "/data/app/com.ztiany.gradlemultidex-1/base.apk"],nativeLibraryDirectories=[/data/app/com.ztiany.gradlemultidex-1/lib/arm64, /vendor/lib64, /system/lib64]]]

4.4系统目录不一样：
getClassLoader():
   dalvik.system.PathClassLoader[DexPathList[[zip file "/data/app/com.ztiany.gradlemultidex-1.apk", nativeLibraryDirectories=[/data/app-lib/com.ztiany.gradlemultidex-1, /system/lib]]]
```

- getSystemClassLoader的DexPathList是空的
- Application的getClassLoader()和MainActivity.class.getClassLoader()返回的ClassLoader是同一个累加器，可以看到DexPathList的路径包含`/data/app/com.ztiany.gradlemultidex-1/base.apk`，这一般就是apk的安装路径了。



---
## 3 MultiDex解决方案

使用MultiDex可以将一个Dex拆分成多个Dex，从而解决单个dex方法数超过65536的问题，

### 3.1 Ant分包方案

ant一般配合eclipse使用，android的SDK中本身就自动够建ant项目的工具，即`SDK/tools/android.bat`工具(将其添加到环境变量中)，在一个eclipse adt android项目下运行命名`android update project -p .`即可在根目录生成`build.xml`文件，这就是ant打包的够建脚本。

>项目根目录的`build.xml`引用了`SDK/tools/ant`中的`build.xml`，这个`build.xml`定义了Android够建的流程。

ant规定`build.xml`中定义根节点为`project`代表项目名称，`project`下定义的target代表够建流程中的各个任务，比如编译任务，dex任务等。同时ant也提供了一些钩子target用于向用于提供扩展。可以将钩子任务写在`custom_rules.xml`中，`build.xml`会自动导入这个文件

大概的够建流程：
```xml
        <target name="debug" depends="-set-debug-files, -do-debug, -post-build"
                 description="Builds the application and signs it with a debug key.">
        </target>
    
          <target name="-do-debug" depends="-set-debug-mode, -debug-obfuscation-check, -package, -post-package">
        </target>
    
        <target name="-post-package" />
        <target name="-post-build" />
    
        <target name="-package" depends="-dex, -package-resources">
        </target>
    
        <target name="-dex" depends="-compile, -post-compile, -obfuscate">
        </target>
    
        <target name="-post-compile"/>
    
        <target name="-compile" depends="-pre-build, -build-setup, -code-gen, -pre-compile">
        </target>
    
        <target name="-pre-compile"/>
        <target name="-pre-build"/>
        <target name="-pre-compile"/>
```

开源项目[Dex65536](https://github.com/mmin18/Dex65536)，就是利用钩子target**`post-compile`**对项目进行dex分包的。具体可以查看改开源项目。

Dex65536的原理是可以指定jar包打包在次dex中，在app启动的时候，编写代码把次dex拷贝到app的私有目录下，接着以这个目录为类路径够建一个DexClassLoader，然后把这个DexClassLoader插入到app的PathClassLoader和PathClassLoader的parent中间，这样就符合的类加载器的委托机制。

插入逻辑为：
```java

public class AppContext extends Application{
    
    @Override
    protected void attachBaseContext(Context base) {
        super.attachBaseContext(base);
        dexTool();
    }    
    
    @SuppressLint("NewApi")
    private void dexTool() {

        File dexDir = new File(getFilesDir(), "dlibs");
        dexDir.mkdir();
        File dexFile = new File(dexDir, "libs.apk");
        
        File dexOpt = getCacheDir();
        
        //把Assets下的libs.apk写到dlibs目录下
        try {
            InputStream ins = getAssets().open("libs.apk");
            if (dexFile.length() != ins.available()) {
                FileOutputStream fos = new FileOutputStream(dexFile);
                byte[] buf = new byte[4096];
                int l;
                while ((l = ins.read(buf)) != -1) {
                    fos.write(buf, 0, l);
                }
                fos.close();
            }
            ins.close();
            
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        //获取ClassLoader
        ClassLoader cl = getClassLoader();
        ApplicationInfo ai = getApplicationInfo();
        String nativeLibraryDir = null;
        //设置so路径
        if (Build.VERSION.SDK_INT > 8) {
            nativeLibraryDir = ai.nativeLibraryDir;
        } else {
            nativeLibraryDir = "/data/data/" + ai.packageName + "/lib/";
        }
        //够建一个新的DexClassLoader
        DexClassLoader dcl = new DexClassLoader(dexFile.getAbsolutePath(),dexOpt.getAbsolutePath(), nativeLibraryDir, cl.getParent());
        //把DexClassLoader插入到类加载器委托机制的中间，这样就可以加载libs.apk中的类了
        try {
            Field f = ClassLoader.class.getDeclaredField("parent");
            f.setAccessible(true);
            f.set(cl, dcl);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

Ant分包方案的缺点是**不能混淆**，并且无法解决依赖倒置问题，但是可以方便的指定那些包分到次dex中。

### 3.2 Gradle分包方案

#### Gradle分包方案步骤

Gradle开启MultiDex很简单，在Androidstudio中，在gradle中使用如下配置即可：

```groovy
    defaultConfig {
            applicationId "com.ztiany.bitmaputil"
            minSdkVersion 15
            targetSdkVersion 23
            versionCode 1
            multiDexEnabled true
            versionName "1.0"
        }
    dependencies {
        compile fileTree(dir: 'libs', include: ['*.jar'])
        testCompile 'junit:junit:4.12'
        compile 'com.android.support:appcompat-v7:23.1.1'
        compile 'com.android.support:multidex:1.0.1'
    }
```

在defaultConfig加上`multiDexEnabled true`，在依赖配置中加上multidex即可。


然后再代码中我们需要使用到MultiDexApplication，使用方法如下：

```xml
        <application
            android:name="android.support.multidex.MultiDexApplication"
            android:allowBackup="true"
            android:icon="@mipmap/ic_launcher"
            android:label="@string/app_name"
            android:supportsRtl="true"
            android:theme="@style/AppTheme">
```

如果需要自定义Application也很简单

```java
       public class AppContext extends MultiDexApplication{}
       或者
       public class AppContext extends Application {

        @Override
        protected void attachBaseContext(Context base) {
            super.attachBaseContext(base);
            MultiDex.install(this);
        }
        @Override
        public void onCreate() {
            super.onCreate();
        }
    }
```

**通过Gradle配置主dex中的类**

配置如下：

```groovy
    // app/build.gradle:
    ......
       afterEvaluate {
        println "afterEvaluate"
        tasks.matching {
            it.name.startsWith('dex')
        }.each { dx ->
            def listFile = project.rootDir.absolutePath + '/app/maindexlist.txt'
            println "root dir:" + project.rootDir.absolutePath
            println "dex task found: " + dx.name
            if (dx.additionalParameters == null) {
                dx.additionalParameters = []
            }
            dx.additionalParameters += '--multi-dex'
            dx.additionalParameters += '--main-dex-list=' + listFile
            dx.additionalParameters += '--minimal-main-dex'
        }
    }
    ......
```

- --multi-dex 指定当前方法数越界时则生成过个dex文件
- --main-dex-list 指定要在主dex中打包的类的列表
- --minimal-main-dex指定只有main-dex-list中知道的类才能打包到主dex中

以上命令选择均属于sdk中的dx工具。


而maindexlist.txt在app目录下，语法如下：

```
    com/ztiany/bitmaputil/AppContext.class
    com/ztiany/bitmaputil/MainActivity.class

    // multidex ， 必须在此文件中
    android/support/multidex/MultiDex.class
    android/support/multidex/MultiDexApplication.class
    android/support/multidex/MultiDexExtractor.class
    android/support/multidex/MultiDexExtractor$1.class
    android/support/multidex/MultiDex$V4.class
    android/support/multidex/MultiDex$V14.class
    android/support/multidex/MultiDex$V19.class
    android/support/multidex/ZipUtil.class
    android/support/multidex/ZipUtil$CentralDirectory.class
```

#### Gradle分包方案的原理

原理在于关键代码`MultiDex.install(this);`，MultiDex中对ClassLoader的类加载路径做了修改。把次dex的路径添加到了App的PathClassLoader中。

MultiDex中的针对各个系统版本有不同的实现，比如V19类就是针对Android 4.4及以上的。

```
    private static final class V19 {
    
            private static void install(ClassLoader loader, List<File> additionalClassPathEntries,
                    File optimizedDirectory)throws IllegalArgumentException, IllegalAccessException,
                            NoSuchFieldException, InvocationTargetException, NoSuchMethodException {
        
                Field pathListField = findField(loader, "pathList");
                Object dexPathList = pathListField.get(loader);
                ArrayList<IOException> suppressedExceptions = new ArrayList<IOException>();
                expandFieldArray(dexPathList, "dexElements", makeDexElements(dexPathList,
                        new ArrayList<File>(additionalClassPathEntries), optimizedDirectory,
                        suppressedExceptions));
                ......
                    suppressedExceptionsField.set(loader, dexElementsSuppressedExceptions);
            }
    }
```

在MultiDex中，会把次dex的路径添加到PathClassLoader的类加载路径中，其改变的就是BaseDexClassLoader中DexPathList的dexElements字段。

---
## 4 MultiDex的缺点

- 启动速度变慢，由于启动需要加载更多的dex，导致启动速度变慢，甚至可能出现ANR现象
- 由于Dalvik LinearAlloc的bug，可能导致使用mulitDex的应用无法在4.0以前的手机使用
