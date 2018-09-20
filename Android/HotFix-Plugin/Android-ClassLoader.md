# Android类加载器

---
## 1 Android中的类加载器

Java中的ClassLoader机制我们应该已经非常熟悉了，而Android只是在此基础之上添加了扩展了新的ClassLoader，用于加载Dex中的Class。

Android中的类加载器继承关系为：

    ClassLoader
        |-->BaseDexClassLoader
             |-->PathClassLoader
             |-->DexClassLoader

PathClassLoader和DexClassLoader是Android平台的两个主要的ClassLoader，它们都继承自BaseDexClassLoader。

### PathClassLoader

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
```

PathClassLoader用来加载本地文件系统上的文件或目录，但不能从网络上加载，**它被用来加载我们的应用程序中的类**。

- dexPath表示包含类和资源的jar或 apk文件列表，多个路径使用路径分隔符分开
- libraryPath表示so路径

### DexClassLoader

```java
public class DexClassLoader extends BaseDexClassLoader {

    public DexClassLoader(String dexPath, String optimizedDirectory,
            String libraryPath, ClassLoader parent) {
        super(dexPath, new File(optimizedDirectory), libraryPath, parent);
    }
}
```

DexClassLoader只是简单的继承了BaseDexClassLoader，DexClassLoader的类加载路径是在创建DexClassLoader对象时指定的，所以它可以加载任何目录下的Dex文件，其主要用来加载`jar 、apk ，其实还包括 zip 文件或者直接加载dex文件`，它可以被用来执行未安装的代码或者未被应用加载过的代码。

参数说明：

- dexPath : 需要被加载的文件地址，可以多个，用路径分隔符分开
- optimizedDirectory : dex文件被加载后会被编译器优化，优化之后的dex存放路径(即将要用来存放ODEX的路径)，不可以为null。注释中也提到需要一个应用私有的可写的一个路径，以防止应用被注入攻击，并且给出了例子` File dexOutputDir = context.getDir("dex", 0) ;`
- libraryPath ：包含libraries的目录列表，plugin中有so文件，需要将so拷贝到本地目录，这里传入将要用来存放so目录的路径
- parent ：父类构造器

PathClassLoader的两个构造函数中调用父类构造器的时候第二个参数传null。即optimizedDirectory为null值，因为Apk在安装过程中其内部的dex已经被提取并且执行过优化了，优化之后放在系统目录` /data/dalvik-cache `(Dalvik模式)下。所以PathClassLoader不直接从APK中加载类，而是从优化过的dex中加载应用程序中的类。

### BaseDexClassLoader

PathClassLoader和DexClassLoader的并没有具体的实现，只是定义了不同的构造方法，功能全都继承自BaseDexClassLoader，而且BaseDexClassLoader的实现也比较简单，其内部有一个关键字段`private final DexPathList pathList;`，DexPathList代码如下：

```java
public class BaseDexClassLoader extends ClassLoader {

    private final DexPathList pathList;

    public BaseDexClassLoader(String dexPath, File optimizedDirectory,
            String libraryPath, ClassLoader parent) {
        super(parent);
        this.pathList = new DexPathList(this, dexPath, libraryPath, optimizedDirectory);
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        List<Throwable> suppressedExceptions = new ArrayList<Throwable>();
        Class c = pathList.findClass(name, suppressedExceptions);
        if (c == null) {
            ClassNotFoundException cnfe = new ClassNotFoundException("Didn't find class \"" + name + "\" on path: " + pathList);
            for (Throwable t : suppressedExceptions) {
                cnfe.addSuppressed(t);
            }
            throw cnfe;
        }
        return c;
    }

    @Override
    protected URL findResource(String name) {
        return pathList.findResource(name);
    }

    @Override
    protected Enumeration<URL> findResources(String name) {
        return pathList.findResources(name);
    }

    @Override
    public String findLibrary(String name) {
        return pathList.findLibrary(name);
    }


    @Override
    protected synchronized Package getPackage(String name) {
        if (name != null && !name.isEmpty()) {
            Package pack = super.getPackage(name);

            if (pack == null) {
                pack = definePackage(name, "Unknown", "0.0", "Unknown", "Unknown", "0.0", "Unknown", null);
            }
            return pack;
        }
        return null;
    }

    public String getLdLibraryPath() {
        StringBuilder result = new StringBuilder();
        for (File directory : pathList.getNativeLibraryDirectories()) {
            if (result.length() > 0) {
                result.append(':');
            }
            result.append(directory);
        }

        return result.toString();
    }

    @Override public String toString() {
        return getClass().getName() + "[" + pathList + "]";
    }
}
```

从源码可以看出，BaseDexClassLoader的方法调用都由DexPathList实现，DexPathList中就保存了ClassLoader对应的dex路径和so路径等。

### DexPathList的分析
```
/**
 *与ClassLoader关联的一对条目列表。其中一个列表是dex /资源路径
 *
 *  类路径条目可以是以下任何一个：
 *      1. 可选的jar或zip文件
 *      2. 顶级{classes.dex}文件以及任意资源
 *      3. 一个普通的dex文件（不能关联资源）
 */
final class DexPathList {

    private static final String DEX_SUFFIX = ".dex";
    private static final String zipSeparator = "!/";

    /** class definition context */
    private final ClassLoader definingContext;

    // dex /资源（类路径）元素的列表。
    private final Element[] dexElements;

    /** List of native library path elements. */
    private final Element[] nativeLibraryPathElements;

    /** List of application native library directories. */
    private final List<File> nativeLibraryDirectories;

    /** List of system native library directories. */
    private final List<File> systemNativeLibraryDirectories;

    private final IOException[] dexElementsSuppressedExceptions;

    /**
     * 构造方法：
     *
     * optimizedDirectory参数说明中说到如果optimizedDirectory为null则使用系统默认路径代替，这个默认路径也就是/data/dalvik-cache/目录，
     * 这个一般情况下是没有权限去访问的，所以这也就解释了为什么我们只能用DexClassLoader去加载类而不用PathClassLoader。
     */
    public DexPathList(ClassLoader definingContext, String dexPath,
            String libraryPath, File optimizedDirectory) {

        //用来验证参数的合法性的
        if (definingContext == null) {
            throw new NullPointerException("definingContext == null");
        }

        if (dexPath == null) {
            throw new NullPointerException("dexPath == null");
        }

        if (optimizedDirectory != null) {
            if (!optimizedDirectory.exists())  {
                throw new IllegalArgumentException("optimizedDirectory doesn't exist: "+ optimizedDirectory);
            }

            if (!(optimizedDirectory.canRead() && optimizedDirectory.canWrite())) {
                throw new IllegalArgumentException( "optimizedDirectory not readable/writable: "   + optimizedDirectory);
            }
        }


        this.definingContext = definingContext;

        /*接下里初始化各个成员*/
        ArrayList<IOException> suppressedExceptions = new ArrayList<IOException>();
        // 
        // 解析或保存dex路径
        this.dexElements = makePathElements(splitDexPath(dexPath), optimizedDirectory, suppressedExceptions);

        //下面是解析和初始化native lib的路径，并且添加了系统so的路径
        this.nativeLibraryDirectories = splitPaths(libraryPath, false/*directoriesOnly*/);
        this.systemNativeLibraryDirectories =  splitPaths(System.getProperty("java.library.path"), true/*directoriesOnly*/);
        List<File> allNativeLibraryDirectories = new ArrayList<>(nativeLibraryDirectories);
        allNativeLibraryDirectories.addAll(systemNativeLibraryDirectories);

        this.nativeLibraryPathElements = makePathElements(allNativeLibraryDirectories, null, suppressedExceptions);

        if (suppressedExceptions.size() > 0) {
            this.dexElementsSuppressedExceptions = suppressedExceptions.toArray(new IOException[suppressedExceptions.size()]);
        } else {
            dexElementsSuppressedExceptions = null;
        }
    }

    @Override public String toString() {
        List<File> allNativeLibraryDirectories = new ArrayList<>(nativeLibraryDirectories);
        allNativeLibraryDirectories.addAll(systemNativeLibraryDirectories);

        File[] nativeLibraryDirectoriesArray =
                allNativeLibraryDirectories.toArray(
                    new File[allNativeLibraryDirectories.size()]);

        return "DexPathList[" + Arrays.toString(dexElements) +
            ",nativeLibraryDirectories=" + Arrays.toString(nativeLibraryDirectoriesArray) + "]";
    }

    public List<File> getNativeLibraryDirectories() {
        return nativeLibraryDirectories;
    }

    private static List<File> splitDexPath(String path) {
        return splitPaths(path, false);
    }

    /**
     *使用separator将给定的路径字符串拆分为文件元素，合并结果并过滤掉不存在的元素，不可读或不是常规文件或目录。
     *directoriesOnly为ture时才会进行校验
     */



    private static List<File> splitPaths(String searchPath, boolean directoriesOnly) {
        List<File> result = new ArrayList<>();

        if (searchPath != null) {
            for (String path : searchPath.split(File.pathSeparator)) {
                if (directoriesOnly) {
                    try {
                        StructStat sb = Libcore.os.stat(path);
                        if (!S_ISDIR(sb.st_mode)) {
                            continue;
                        }
                    } catch (ErrnoException ignored) {
                        continue;
                    }
                }
                result.add(new File(path));
            }
        }

        return result;
    }

    /**
     * 创建一个dex / resource路径元素数组
     */
    private static Element[] makePathElements(List<File> files, File optimizedDirectory, List<IOException> suppressedExceptions) {
        List<Element> elements = new ArrayList<>();
        /*
         * Open all files and load the (direct or contained) dex files
         * up front.
         */
        for (File file : files) {
            File zip = null;
            File dir = new File("");
            DexFile dex = null;
            String path = file.getPath();
            String name = file.getName();
            /* zipSeparator = !/    */
            if (path.contains(zipSeparator)) {
                String split[] = path.split(zipSeparator, 2);
                zip = new File(split[0]);
                dir = new File(split[1]);
            } else if (file.isDirectory()) {//如果是目录
                // We support directories for looking up resources and native libraries.
                // Looking up resources in directories is useful for running libcore tests.
                elements.add(new Element(file, true, null, null));
            } else if (file.isFile()) {
                //如果是 .dex 文件
                if (name.endsWith(DEX_SUFFIX)) {    /*DEX_SUFFIX = .dex*/
                    // Raw dex file (not inside a zip/jar).
                    try {
                        dex = loadDexFile(file, optimizedDirectory);
                    } catch (IOException ex) {
                        System.logE("Unable to load dex file: " + file, ex);
                    }
                } else {//其他文件处理方式
                    zip = file;
                    try {
                        dex = loadDexFile(file, optimizedDirectory);
                    } catch (IOException suppressed) {
                        /*
                         * IOException might get thrown "legitimately" by the DexFile constructor if
                         * the zip file turns out to be resource-only (that is, no classes.dex file
                         * in it).
                         * Let dex == null and hang on to the exception to add to the tea-leaves for
                         * when findClass returns null.
                         */
                        suppressedExceptions.add(suppressed);
                    }
                }
            } else {
                System.logW("ClassLoader referenced unknown path: " + file);
            }
            //一个File对应一个Element，一般情况下也只会存在一个file
            if ((zip != null) || (dex != null)) {
                elements.add(new Element(dir, false, zip, dex));
            }
        }
        return elements.toArray(new Element[elements.size()]);
    }

    /**
     * 加载dex文件
     */
    private static DexFile loadDexFile(File file, File optimizedDirectory)
            throws IOException {
        //如果 optimizedDirectory == null 则直接new 一个DexFile
        if (optimizedDirectory == null) {
            return new DexFile(file);
        } else {
            //否则就使用DexFile.loadDex来创建一个DexFile实例。这个方法用来构建一个odex路径，后面加上了.dex后缀
            String optimizedPath = optimizedPathFor(file, optimizedDirectory);
            //loadDex只是构建一个DexFile并返回
            return DexFile.loadDex(file.getPath(), optimizedPath, 0);
        }
    }

    /**
     * 将dex / jar文件路径和输出目录转换为关联的优化dex文件的输出文件路径。
     */
    private static String optimizedPathFor(File path,
            File optimizedDirectory) {

        String fileName = path.getName();
        if (!fileName.endsWith(DEX_SUFFIX)) {
            int lastDot = fileName.lastIndexOf(".");
            if (lastDot < 0) {
                fileName += DEX_SUFFIX;
            } else {
                StringBuilder sb = new StringBuilder(lastDot + 4);
                sb.append(fileName, 0, lastDot);
                sb.append(DEX_SUFFIX);
                fileName = sb.toString();
            }
        }

        File result = new File(optimizedDirectory, fileName);
        return result.getPath();
    }

    public Class findClass(String name, List<Throwable> suppressed) {
        for (Element element : dexElements) {
            DexFile dex = element.dexFile;
            if (dex != null) {
                Class clazz = dex.loadClassBinaryName(name, definingContext, suppressed);
                if (clazz != null) {
                    return clazz;
                }
            }
        }
        if (dexElementsSuppressedExceptions != null) {
            suppressed.addAll(Arrays.asList(dexElementsSuppressedExceptions));
        }
        return null;
    }

    public URL findResource(String name) {
        for (Element element : dexElements) {
            URL url = element.findResource(name);
            if (url != null) {
                return url;
            }
        }

        return null;
    }

    public Enumeration<URL> findResources(String name) {
        ArrayList<URL> result = new ArrayList<URL>();

        for (Element element : dexElements) {
            URL url = element.findResource(name);
            if (url != null) {
                result.add(url);
            }
        }

        return Collections.enumeration(result);
    }

    public String findLibrary(String libraryName) {
        String fileName = System.mapLibraryName(libraryName);

        for (Element element : nativeLibraryPathElements) {
            String path = element.findNativeLibrary(fileName);

            if (path != null) {
                return path;
            }
        }

        return null;
    }

    static class Element {
        private final File dir;
        private final boolean isDirectory;
        private final File zip;
        private final DexFile dexFile;

        private ZipFile zipFile;
        private boolean initialized;

        public Element(File dir, boolean isDirectory, File zip, DexFile dexFile) {
            this.dir = dir;
            this.isDirectory = isDirectory;
            this.zip = zip;
            this.dexFile = dexFile;
        }

        @Override public String toString() {
            if (isDirectory) {
                return "directory \"" + dir + "\"";
            } else if (zip != null) {
                return "zip file \"" + zip + "\"" +
                       (dir != null && !dir.getPath().isEmpty() ? ", dir \"" + dir + "\"" : "");
            } else {
                return "dex file \"" + dexFile + "\"";
            }
        }

        public synchronized void maybeInit() {
            if (initialized) {
                return;
            }

            initialized = true;

            if (isDirectory || zip == null) {
                return;
            }

            try {
                zipFile = new ZipFile(zip);
            } catch (IOException ioe) {
                /*
                 * Note: ZipException (a subclass of IOException)
                 * might get thrown by the ZipFile constructor
                 * (e.g. if the file isn't actually a zip/jar
                 * file).
                 */
                System.logE("Unable to open zip file: " + zip, ioe);
                zipFile = null;
            }
        }

        private boolean isZipEntryExistsAndStored(ZipFile zipFile, String path) {
            ZipEntry entry = zipFile.getEntry(path);
            return entry != null && entry.getMethod() == ZipEntry.STORED;
        }

        public String findNativeLibrary(String name) {
            maybeInit();

            if (isDirectory) {
                String path = new File(dir, name).getPath();
                if (IoUtils.canOpenReadOnly(path)) {
                    return path;
                }
            } else if (zipFile != null) {
                String entryName = new File(dir, name).getPath();
                if (isZipEntryExistsAndStored(zipFile, entryName)) {
                  return zip.getPath() + zipSeparator + entryName;
                }
            }

            return null;
        }

        public URL findResource(String name) {
            maybeInit();

            if (isDirectory) {
                File resourceFile = new File(dir, name);
                if (resourceFile.exists()) {
                    try {
                        return resourceFile.toURI().toURL();
                    } catch (MalformedURLException ex) {
                        throw new RuntimeException(ex);
                    }
                }
            }

            if (zipFile == null || zipFile.getEntry(name) == null) {
                /*
                 * Either this element has no zip/jar file (first
                 * clause), or the zip/jar file doesn't have an entry
                 * for the given name (second clause).
                 */
                return null;
            }

            try {
                return new URL("jar:" + zip.toURL() + "!/" + name);
            } catch (MalformedURLException ex) {
                throw new RuntimeException(ex);
            }
        }
    }//Element end
}

```

### DexFile分析

```java
/** DexFile表示包含类路径或其他资源的文件 */
public final class DexFile {

    private Object mCookie;
    private final String mFileName;
    private final CloseGuard guard = CloseGuard.get();

    /**
     * 从给定的File对象打开一个DEX文件。这通常是一个ZIP / JAR，里面有一个“classes.dex”文件。
     *
     * VM将在/ data / dalvik-cache中生成相应文件的名称并将其打开，如果系统权限允许，
     * 可能会先创建或更新它。不要传入/ data / dalvik-cache中的文件名称，因为指定的文件应该处于其原始（pre-dexopt）状态
     */
    public DexFile(File file) throws IOException {
        this(file.getPath());
    }

    public DexFile(String fileName) throws IOException {
        mCookie = openDexFile(fileName, null, 0);
        mFileName = fileName;
        guard.open("close");
        //System.out.println("DEX FILE cookie is " + mCookie + " fileName=" + fileName);
    }

    /**
     * 从给定的文件名打开DEX文件，使用指定的文件来保存优化的数据
     */
    private DexFile(String sourceName, String outputName, int flags) throws IOException {
        if (outputName != null) {
            try {
                String parent = new File(outputName).getParent();
                if (Libcore.os.getuid() != Libcore.os.stat(parent).st_uid) {
                    throw new IllegalArgumentException("Optimized data directory " + parent
                            + " is not owned by the current user. Shared storage cannot protect"
                            + " your application from code injection attacks.");
                }
            } catch (ErrnoException ignored) {
                // assume we'll fail with a more contextual error later
            }
        }

        mCookie = openDexFile(sourceName, outputName, flags);
        mFileName = sourceName;
        guard.open("close");
        //System.out.println("DEX FILE cookie is " + mCookie + " sourceName=" + sourceName + " outputName=" + outputName);
    }


    static public DexFile loadDex(String sourcePathName, String outputPathName,int flags) throws IOException {
        return new DexFile(sourcePathName, outputPathName, flags);
    }

    public String getName() {
        return mFileName;
    }

    @Override public String toString() {
        return getName();
    }

    public void close() throws IOException {
        if (mCookie != null) {
            guard.close();
            closeDexFile(mCookie);
            mCookie = null;
        }
    }

  
    public Class loadClass(String name, ClassLoader loader) {
        String slashName = name.replace('.', '/');
        return loadClassBinaryName(slashName, loader, null);
    }

  
    public Class loadClassBinaryName(String name, ClassLoader loader, List<Throwable> suppressed) {
        return defineClass(name, loader, mCookie, suppressed);
    }

    private static Class defineClass(String name, ClassLoader loader, Object cookie,
                                     List<Throwable> suppressed) {
        Class result = null;
        try {
            result = defineClassNative(name, loader, cookie);
        } catch (NoClassDefFoundError e) {
            if (suppressed != null) {
                suppressed.add(e);
            }
        } catch (ClassNotFoundException e) {
            if (suppressed != null) {
                suppressed.add(e);
            }
        }
        return result;
    }


    public Enumeration<String> entries() {
        return new DFEnum(this);
    }

    // Helper class.
    private class DFEnum implements Enumeration<String> {
        private int mIndex;
        private String[] mNameList;

        DFEnum(DexFile df) {
            mIndex = 0;
            mNameList = getClassNameList(mCookie);
        }

        public boolean hasMoreElements() {
            return (mIndex < mNameList.length);
        }

        public String nextElement() {
            return mNameList[mIndex++];
        }
    }

  
    @Override protected void finalize() throws Throwable {
        try {
            if (guard != null) {
                guard.warnIfOpen();
            }
            close();
        } finally {
            super.finalize();
        }
    }

    private static Object openDexFile(String sourceName, String outputName, int flags) throws IOException {
        // Use absolute paths to enable the use of relative paths when testing on host.
        return openDexFileNative(new File(sourceName).getAbsolutePath(), (outputName == null) ? null : new File(outputName).getAbsolutePath(),  flags);
    }

    private static native void closeDexFile(Object cookie);
    private static native Class defineClassNative(String name, ClassLoader loader, Object cookie) throws ClassNotFoundException, NoClassDefFoundError;
    private static native String[] getClassNameList(Object cookie);
   
    private static native Object openDexFileNative(String sourceName, String outputName, int flags);

    public static native boolean isDexOptNeeded(String fileName) throws FileNotFoundException, IOException;

    public static final int NO_DEXOPT_NEEDED = 0;

    public static final int DEX2OAT_NEEDED = 1;

    public static final int PATCHOAT_NEEDED = 2;

    public static final int SELF_PATCHOAT_NEEDED = 3;

    public static native int getDexOptNeeded(String fileName, String pkgname,  String instructionSet, boolean defer) throws FileNotFoundException, IOException;

}
```

熟悉DexPathList类后，可以总结出BaseDexClassLoader的构建流程

1. 在创建BaseDexClassLoader时，需要传入对应的参数，分别是
    - dexPath：需要被加载的文件路径，可以是`.apk .jar .dex .zip`文件
    - optimizedDirectory：用于存储odex的路径
    - libraryPath ：用于存放so的路径
    - parent：作为该类加载器的父类构造器
2. BaseDexClassLoader构造函数中会创建一个DexPathList对象，然后将加载类和资源的方法都交给DexPathList去实现
3. DexPathList的创建需要`ClassLoader、dexPath、libraryPath、optimizedDirectory`
4. DexPathList内部首先校验传入的参数，然后初始化`dexElements、nativeLibraryPathElements`等
    - nativeLibraryDirectories保存了libraryPath被分割后的数组，并且加上了系统so库的目录
    - dexElements保存了由dexPath被分割后的对应的file而创建的Element，它只是一个简单实体类，由一个File、ZipFile、DexFile组成
5. 最终DexPathList的findClass方法通过遍历内部的dexElements，通过Element对象中存储的DexFile来加载Class
    - ZipFile是由jar、zip、apk形式的file包装成而来，DexFile使用native方法openDexFile打开了具体的file并输出到优化路径。

---
## 2 类加载的流程

首先看一下Android中类加载器的层级结构

在Application类中执行下面代码：
```java
        Log.d(TAG, "ClassLoader.getSystemClassLoader():" + ClassLoader.getSystemClassLoader());
        
        Log.d(TAG, "getClassLoader():" + getClassLoader());
        
        Log.d(TAG, "String.class.getClassLoader():    " + String.class.getClassLoader());
        
        Log.d(TAG, "MainActivity.class.getClassLoader():" + MainActivity.class.getClassLoader());

        ClassLoader classLoader = MainActivity.class.getClassLoader();
        while (classLoader != null) {
            Log.d(TAG, "" + classLoader);
            classLoader = classLoader.getParent();
        }
```

打印的结果为：

```
//打印各种方法获取的ClassLoader
 ClassLoader.getSystemClassLoader(): 
    dalvik.system.PathClassLoader[DexPathList[[directory "."],nativeLibraryDirectories=[/vendor/lib64, /system/lib64]]]
    
AppContext: getClassLoader():
    dalvik.system.PathClassLoader[DexPathList[[zip file "/data/app/com.ztiany.classloader-1/base.apk"],nativeLibraryDirectories=[/data/app/com.ztiany.classloader-1/lib/arm64, /vendor/lib64, /system/lib64]]]
    
String.class.getClassLoader():    
    java.lang.BootClassLoader@2b0a45f    
    
MainActivity.class.getClassLoader():    
    dalvik.system.PathClassLoader[DexPathList[[zip file "/data/app/com.ztiany.classloader-1/base.apk"],nativeLibraryDirectories=[/data/app/com.ztiany.classloader-1/lib/arm64, /vendor/lib64, /system/lib64]]]

//往上遍历ClassLoader的层次结构
    dalvik.system.PathClassLoader[DexPathList[[zip file "/data/app/com.ztiany.classloader-1/base.apk"],nativeLibraryDirectories=[/data/app/com.ztiany.classloader-2/lib/arm64, /vendor/lib64, /system/lib64]]]
    java.lang.BootClassLoader@2b0a45f
```
从结果可以看出，Android中ClassLoader的层级结构与Java是不同的：

- BootClassLoader是顶级的类加载器，由它加载系统API和Java标准类库
- PathClassLoader用于加载APK中的Class
- Context的getClassLoader()方法返回的就是用于加载APK中的类的ClassLoader
- `/vendor/lib64, /system/lib64`是创建类加载器时自动添加的系统本地库路径

所以默认情况下Android中的类加载器层级结构只有两层，采用的依然是双亲委派机制，由BootClassLoader加载标准库中的Class，由PathClassLoader加载APK中的Class。

---
## 3 hook ClassLoader

可以在PathClassLoader和系统的BootClassLoader之间插入我们自己的ClassLoader，这样就可以加载外部的Class了，方法如下：

```java
public class AppContext extends Application{
    
       @Override
       public void onCreate() {
           super.onCreate();
           hookPluginClassLoader();
       }
    
    private void hookPluginClassLoader() {
        File dexDir = new File(getFilesDir(), "plugins");
        dexDir.mkdir();
        File dexFile = new File(dexDir, "gson.dex");

        File dexOpt = getCacheDir();

        //把Assets下的libs.apk写到dlibs目录下
        try {
            InputStream ins = getAssets().open("gson.dex");
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
        //够建一个新的DexClassLoader
        DexClassLoader dcl = new PluginDexClassLoader(dexFile.getAbsolutePath(), dexOpt.getAbsolutePath(), null, cl.getParent());
        //把DexClassLoader插入到类加载器委托机制的中间，这样就可以加载外部dex中的类了
        try {
            Field f = ClassLoader.class.getDeclaredField("parent");
            f.setAccessible(true);
            f.set(cl, dcl);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

}

    public class PluginDexClassLoader extends DexClassLoader {

        public PluginDexClassLoader(String dexPath, String optimizedDirectory, String librarySearchPath, ClassLoader parent) {
            super(dexPath, optimizedDirectory, librarySearchPath, parent);
        }

        @Override
        protected Class<?> findClass(String name) throws ClassNotFoundException {
            Log.d(TAG, "findClass() called with: name = [" + name + "]");
            return super.findClass(name);
        }
    }
```

---
## 引用

- [Android 类加载机制](https://mp.weixin.qq.com/s?__biz=MzAwOTIwMDQ3MQ==&mid=2451013689&idx=1&sn=92bd00d6227a7bad33d02d39cface6c2&chksm=8c801601bbf79f178cb9de2be1f260e1259309ecc5e96e1f48dc16bb624aa762046544a2105f&mpshare=1&scene=1&srcid=1025tXXQcHP767luBVthUiTm#rd)