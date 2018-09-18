[TOC]
# Android Transform API学习

---
## 1 Transform API介绍

Android项目使用gradle进行构建，具体由`Android gradle plugin`实现，Android gradle plugin开始于1.5.0-beta1添加了Transform API， 可以用于在android打包过程中加入开发者自定义的打包处理逻辑，现在Android打包的`multi-dex、jar2dex`流程都已经使用这个Transform API来实现了，

**自定义的Transform一经注册便会自动添加到Task执行序列中，并且正好是项目被打包成dex之前。**


---
## 2 Transform

### 2.1 流程

Transform每次都是将一个输入进行处理，然后将处理结果输出，而输出的结果将会作为另一个Transform的输入，在整个打包过程中可以有无数个这样的Transform流程，最让将最后一个Transform的输出结果与其他资源打包生成APK文件。

![](images/gradle_android_transform_steps.png)

从项目`build/intermediates/transforms`文件夹也可以找到这些Transform的输出。

![](images/gradle_android_transform_project.png)

### 2.2 API使用

一般添加自定义的Transform流程是：

- 自定义`gradle plugin`
- 实现`com.android.build.api.transform.Transform`类
- 在自定的`gradle plugin`中注册我们实现的Transform
- 在项目中应用这个自定义`gradle plugin`

注入自定的Transform很简单，代码如下：

```groovy
    class TransformPlugin implements Plugin<Project> {
        @Override
        void apply(Project project) {
            //得到AppExtension
            def android = project.extensions.findByType(AppExtension)
            android.registerTransform(new PrintTransform(project))
        }
    }
```

重点在于实现Transform，一般需要实现的方法如下：

```groovy
    class PrintTransform extends Transform {

        @Override
        String getName() {
            //自定义的Transform对应的Task名称
            return 'PrintTransform'
        }

        @Override
        Set<QualifiedContent.ContentType> getInputTypes() {
            //指定输入的类型，通过这里的设定，可以指定我们要处理的文件类型，确保其他类型的文件不会传入
            return TransformManager.CONTENT_CLASS
        }

        @Override
        Set<? super QualifiedContent.Scope> getScopes() {
            //指定Transform的作用范围
            return TransformManager.SCOPE_FULL_PROJECT
        }

        @Override
        boolean isIncremental() {
            //是否为增量transform
            return false
        }

        @Override
        void transform(Context context, Collection<TransformInput> inputs, Collection<TransformInput> referencedInputs, TransformOutputProvider outputProvider, boolean isIncremental) throws IOException, TransformException, InterruptedException {
            //在这里实现我们的transform逻辑
        }
    }
```

方法说明：

- `getName()`:用于返回我们自定义Transform的名称
- `getInputTypes()`:用于指定要处理的输入的类型，通过这里的设定，可以指定我们要处理的文件类型，确保其他类型的文件不会传入
- `getScopes()`:用于指定Transform的作用范围
- `transform()`:transform的逻辑

#### transform方法

transform方法参数说明:

- `Collection<TransformInput> inputs`：表示所有输入
- `Collection<TransformInput> referencedInputs`：TransformInput的referencedInputs集合。这仅供参考，不应改变。接收到的内容通过getReferencedScopes（）进行控制。
- `TransformOutputProvider outputProvider`：用于提供输出目录
- `boolean isIncremental`:是否为增量转换


这个方法于实现transform的逻辑，默认情况下如果不需要做任何transform，这个方法也需要把对应的输入复制到目标输出地址，否则整个打包流程会因为没有出入而终止运行。输出地址不是我们定义的，而是由TransformOutputProvider对象提供的，通过它的getContentLocation方法获取输出的目标地址。

所以默认情况下也需要做如下逻辑处理：

```groovy
    @Override
        void transform(Context context, Collection<TransformInput> inputs, Collection<TransformInput> referencedInputs, TransformOutputProvider outputProvider, boolean isIncremental) throws IOException, TransformException, InterruptedException {
            // Transform的inputs有两种类型，一种是目录，一种是jar包，要分开遍历
            inputs.each {
                TransformInput input ->
                    //对类型为“文件夹”的input进行遍历
                    dirProcess(input, outputProvider)
                    //对类型为jar文件的input进行遍历
                    jarProcess(input, outputProvider)
            }
        }
    
        private
        static ArrayList<DirectoryInput> dirProcess(TransformInput input, TransformOutputProvider outputProvider) {
            input.directoryInputs.each {
                DirectoryInput directoryInput ->
                    //文件夹里面包含的是我们手写的类以及R.class、BuildConfig.class以及R$XXX.class等
                    def dest = outputProvider.getContentLocation(directoryInput.name, directoryInput.contentTypes, directoryInput.scopes, Format.DIRECTORY)
                    // 将input的目录复制到output指定目录
                    FileUtils.copyDirectory(directoryInput.file, dest)
            }
        }
    
        private
        static ArrayList<JarInput> jarProcess(TransformInput input, TransformOutputProvider outputProvider) {
            input.jarInputs.each {
                JarInput jarInput ->
                    //jar文件一般是第三方依赖库jar文件
                    //重命名输出文件，防止同目录copyFile冲突
                    println "-----------------" + jarInput.file
                    def jarName = jarInput.name
                    def md5Name = DigestUtils.md5Hex(jarInput.file.getAbsolutePath())
                    if (jarName.endsWith(".jar")) {
                        jarName = jarName.substring(0, jarName.length() - 4)
                    }
                    //生成输出路径
                    def dest = outputProvider.getContentLocation(jarName + md5Name, jarInput.contentTypes, jarInput.scopes, Format.JAR)
                    //将输入内容复制到输出目录
                    FileUtils.copyFile(jarInput.file, dest)
            }
        }
```


---
## 3 其他

- 自定义的transform是不能处理Dex文件的,因为对应的DexTransform和MultiDexTransform注册在最后面。
- **避免文件重复**：Transform会根据Transform插件为本次transform生成对应的文件夹，在transform方法中利用outputProvider生成的文件夹都在此文件夹中，在下一次的Transform中，又会把改文件夹中所有的内容作为transform方法的输入。如果我们在一次变换中，经过操作生成生成另一种形态的相同的内容(比如把Class打包成jar)，务必要原始的形态的内容删除。

---
## 引用

- [Transform API](http://tools.android.com/tech-docs/new-build-system/transform-api)
- [通过自定义Gradle插件修改编译后的class文件](http://blog.csdn.net/huachao1001/article/details/51819972)
- [gradle transform api 初探](http://www.jianshu.com/p/c9ce643e2f22)








