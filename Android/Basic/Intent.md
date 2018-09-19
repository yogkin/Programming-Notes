# Intent

---
## 1 Intent介绍

Intent 是一个消息传递对象，可以使用它从其他应用组件请求操作。

```java
    public class Intent implements Parcelable, Cloneable {
        //......
    }
```

从Intent声明来看，其支持Android的序列化(因为需要跨进程传递)和克隆，Intent常用方法：

```java
           //构造方法
            Intent intent = new Intent();
            Intent intent1 = new Intent(this, MainActivity.class);
            Intent intent2 = new Intent(Intent.ACTION_CALL);
            Intent intent3 = new Intent(Intent.ACTION_CALL, Uri.parse("tel:123"));
            //添加类别
            intent.addCategory(Intent.CATEGORY_ALTERNATIVE);
            //添加flag源数据
            intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK);
            //设置一些数据
            intent.putExtra("", "");
            intent.setType()
            intent.setAction()
            intent.setClass()
            intent.setClassName()
            intent.setData()
            intent.setDataAndType();
            intent.setComponent()
             //获取uri数据
            intent.getData()
            //验证意图将解决一个活动
            ComponentName componentName = intent.resolveActivity(getPackageManager());
```

## 2 Intent的类型

- 显式的Intent：按名称（完全限定类名）指定要启动的组件。通常，您会在自己的应用中使用显式 Intent 来启动组件
- 隐式的Intent：不会指定特定的组件，而是声明要执行的常规操作，从而允许其他应用中的组件来处理它。

>为了确保应用的安全性，启动 Service 时，请始终使用显式 Intent，且不要为服务声明 Intent 过滤器。使用隐式 Intent 启动服务存在安全隐患，因为您无法确定哪些服务将响应 Intent，且用户无法看到哪些服务已启动。从 Android 5.0（API 级别 21）开始，如果使用隐式 Intent 调用 bindService()，系统会抛出异常。

---
## 3 Intent主要包含的信息

### 组件名称

要启动的组件名称。如果没有指定组件名称，则Intent是隐式的

### action

表示要执行的动作，action用于隐式的Intent，此时系统会根据Intent的action以及其他信息来决定由谁来处理这个Intent，系统常见的Intent为：

- ACTION_VIEW 如果您拥有一些某项 Activity 可向用户显示的信息（例如，要使用图库应用查看的照片；或者要使用地图应用查找的地址），请使用 Intent 将此操作与 startActivity() 结合使用。
- ACTION_SEND 这也称为“共享” Intent。如果您拥有一些用户可通过其他应用（例如，电子邮件应用或社交共享应用）共享的数据，则应使用 Intent 中将此操作与 startActivity() 结合使用。


如果定义自己的操作，最后确保应用的软件包名称作为前缀。：

```
    static final String ACTION_TIMETRAVEL = "com.example.action.TIMETRAVEL";
```

### 数据

引用待操作数据和/或该数据 MIME 类型的 URI（Uri 对象）。提供的数据类型通常由 Intent 的操作决定。例如，如果操作是 ACTION_EDIT，则数据应包含待编辑文档的 URI。

**指定数据的MIME类型有助于Android系统找到接收Intent的最佳组件**

>若要同时设置 URI 和 MIME 类型，请勿调用 setData() 和 setType()，因为它们会互相抵消彼此的值。请始终使用 setDataAndType() 同时设置 URI 和 MIME 类型。

### 类别

一个包含应处理 Intent 组件类型的附加信息的字符串。可以将任意数量的类别描述放入一个 Intent 中，但大多数 Intent 均不需要类别。以下是一些常见类别：

>为了接收隐式 Intent，必须将 CATEGORY_DEFAULT 类别包括在 Intent 过滤器中。方法 startActivity() 和 startActivityForResult() 将按照已申明 CATEGORY_DEFAULT 类别的方式处理所有 Intent。 如果未在 Intent 过滤器中声明此类别，则隐式 Intent 不会解析为您的 Activity。

### Extra

携带完成请求操作所需的附加信息的键值对

### Flag

一般用于指示 Android 系统如何启动 Activity（例如，Activity 应属于哪个 任务 ），以及启动之后如何处理（例如，它是否属于最近的 Activity 列表）。

---
## 4 隐式的Intent使用注意

###  安全的使用

如果系统中没有组件可以处理隐式的Intent，则调用将会失败，且应用会崩溃

正确的使用方式应该为：

```java
    Intent sendIntent = new Intent();
    sendIntent.setAction(Intent.ACTION_SEND);
    sendIntent.putExtra(Intent.EXTRA_TEXT, textMessage);
    sendIntent.setType("text/plain");
    // Verify that the intent will resolve to an activity 
    if (sendIntent.resolveActivity(getPackageManager()) != null) {
        startActivity(sendIntent);
    } 
    或者可以使用PackageManager的query类方法
```

### 强制使用应用选择器

如果多个应用可以响应 Intent，且用户可能希望每次使用不同的应用，则应采用显式方式显示选择器对话框：

```java
    Intent sendIntent = new Intent(Intent.ACTION_SEND);
    ... 
    // Always use string resources for UI text. 
    // This says something like "Share this photo with" 
    String title = getResources().getString(R.string.chooser_title);
    // Create intent to show the chooser dialog 
    Intent chooser = Intent.createChooser(sendIntent, title);
     
    // Verify the original intent will resolve to at least one activity 
    if (sendIntent.resolveActivity(getPackageManager()) != null) {
        startActivity(chooser);
    }
```

## 5 接收隐式的Intent

要公布应用可以接收哪些隐式Intent，需要在清单文件的对应组件中声明Intent-Fliter。示例：

```xml
    <activity android:name="ShareActivity">
        <!-- 处理send的action，并且数据类型为text -->
        <intent-filter>
            <action android:name="android.intent.action.SEND"/>
            <category android:name="android.intent.category.DEFAULT"/>
            <data android:mimeType="text/plain"/>
        </intent-filter>
        
        <!-- 处理"SEND" and "SEND_MULTIPLE"的action，并且数据类型为图片或者视频 -->
        <intent-filter>
            <action android:name="android.intent.action.SEND"/>
            <action android:name="android.intent.action.SEND_MULTIPLE"/>
            <category android:name="android.intent.category.DEFAULT"/>
            <data android:mimeType="application/vnd.google.panorama360+jpg"/>
            <data android:mimeType="image/*"/>
            <data android:mimeType="video/*"/>
        </intent-filter>
    </activity>
```

IntentFliter中包含三个部分：

- action指定要处理的动作
- category指定处理Intent的类型，只要要添加`android.intent.category.DEFAULT`
- data部分 指定可以后处理的数据，而data又可以分为以下部分
    - **android:scheme**  匹配url中的前缀，除了`“http”、“https”、“tel”...`之外，我们可以定义自己的前缀
    - **android:host**  匹配uri中的主机名部分，如“google.com”，如果定义为“*”则表示任意主机名
    - **android:port**  匹配uri中的端口
    - **android:path**  匹配uri中的路径
    - **android:pathPrefix** 匹配uri中的路径前缀，如果不想把路径写死，可以使用`pathPrefix`对路径的部分进行约束，如`android:pathPrefix="/target"`表示处理的Intent的uri中的路径应该以target开头。
    - **android:pathPattern**`：匹配Uri的中的路径模式，如android:pathPattern=".*//.pdf"`，表示处理最后以pdf结果的uri。

### 区分path/pathPrefix/pathPattern

- **path**用来匹配完整的路径，如：`http://example.com/blog/abc.html`，这里将 path 设置为 `/blog/abc.html` 才能够进行匹配；
- **pathPrefix**用来匹配路径的开头部分，拿上面的 Uri 来说，这里将 pathPrefix 设置为 /blog 就能进行匹配了；
- **pathPattern**用表达式来匹配整个路径，这里需要说下匹配符号与转义。

**匹配符号**：

1. ` “*”` 用来匹配0次或更多，如：`“a*”` 可以匹配 `“a”、“aa”、“aaa”...`
2.  `“.”` 用来匹配任意字符，如：`“.”` 可以匹配 `“a”、“b”，“c”...`
3.  因此 `“.*”` 就是用来匹配任意字符0次或更多，如：`“.*html”` 可以匹配 `“abchtml”、“chtml”，“html”，“sdf.html”...`


**转义**：

1. 因为当读取 Xml 的时候，`“\”` 是被当作转义字符的（当它被用作 pathPattern 转义之前），因此这里需要两次转义，读取 Xml 是一次，在 pathPattern 中使用又是一次。如：`“*”` 这个字符就应该写成 `“\\*”`，“\” 这个字符就应该写成 `“\\\\”`。

---
## 6 Intent的解析(匹配规则)

当系统收到隐式 Intent 以启动 Activity 时，它根据以下三个方面将该 Intent 与 Intent 过滤器进行比较，搜索该 Intent 的最佳 Activity：

*   Intent action
*   Intent 数据（URI 和数据类型）
*   Intent 类别

### Action

要指定接受的 Intent 操作， Intent 过滤器既可以不声明任何action; 元素，也可以声明多个此类元素

```xml
    <intent-filter>
        <action android:name="android.intent.action.EDIT" />
        <action android:name="android.intent.action.VIEW" />
        ...
    </intent-filter>
```

要通过此过滤器，您在Intent中指定的操作必须与过滤器中列出的某一操作匹配。

- 如果该过滤器未列出任何操作，则 Intent 没有任何匹配项，因此所有 Intent 均无法通过测试。
- 但是，如果 Intent 未指定操作，则会通过测试（只要过滤器至少包含一个操作）。

### 类别测试

要指定接受的 Intent 类别， Intent 过滤器既可以不声明任何 category 元素，也可以声明多个此类元素

```xml
    <intent-filter>
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        ...
    </intent-filter>
```

若要 Intent 通过类别测试，则 Intent 中的每个类别均必须与过滤器中的类别匹配。反之则未必然，Intent 过滤器声明的类别可以超出 Intent 中指定的数量，且 Intent 仍会通过测试。因此，不含类别的 Intent 应当始终会通过此测试，无论过滤器中声明何种类别均是如此。

>Android 会自动将 CATEGORY_DEFAULT 类别应用于传递给 startActivity() 和 startActivityForResult() 的所有隐式 Intent。因此，如需 Activity 接收隐式 Intent，则必须将 "android.intent.category.DEFAULT" 的类别包括在其 Intent 过滤器中


### 数据

比如一个Uri为：`content://com.example.project:200/folder/subfolder/etc`，其中架构是 `content`，主机是 `com.example.project`，端口是 `200`，路径是 `folder/subfolder/etc`。

上述每个属性均为可选，但存在线性依赖关系：

*   如果未指定架构，则会忽略主机。
*   如果未指定主机，则会忽略端口。
*   如果未指定架构和主机，则会忽略路径。

将 Intent 中的 URI 与过滤器中的 URI 规范进行比较时，它仅与过滤器中包含的部分 URI 进行比较：

*   如果过滤器仅指定架构，则具有该架构的所有 URI 均与该过滤器匹配。
*   如果过滤器指定架构和权限、但未指定路径，则具有相同架构和权限的所有 URI 都会通过过滤器，无论其路径如何均是如此。
*   如果过滤器指定架构、权限和路径，则仅具有相同架构、权限和路径 的 URI 才会通过过滤器。


数据测试会将 Intent 中的 URI 和 MIME 类型与过滤器中指定的 URI 和 MIME 类型进行比较。规则如下：

- 仅当过滤器未指定任何 URI 或 MIME 类型时，不含 URI 和 MIME 类型的 Intent 才会通过测试。
- 对于包含 URI、但不含 MIME 类型（既未显式声明，也无法通过 URI 推断得出）的 Intent，仅当其 URI 与过滤器的 URI 格式匹配、且过滤器同样未指定 MIME 类型时，才会通过测试。
- 仅当过滤器列出相同的 MIME 类型且未指定 URI 格式时，包含 MIME 类型、但不含 URI 的 Intent 才会通过测试。
- 仅当 MIME 类型与过滤器中列出的类型匹配时，包含 URI 和 MIME 类型（通过显式声明，或可以通过 URI 推断得出）的 Intent 才会通过测试的 MIME 类型部分。如果 Intent 的 URI 与过滤器中的 URI 匹配，或者如果 Intent 具有 content: 或 file: URI 且过滤器未指定 URI，则 Intent 会通过测试的 URI 部分。换而言之，如果过滤器仅列出 MIME 类型，则假定组件支持 content: 和 file: 数据。

---
## 引用

- [Intent 文档](https://developer.android.com/guide/components/intents-filters.html)