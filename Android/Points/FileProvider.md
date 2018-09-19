# Android N 严苛模式

Android N中默认开启了严苛模式，将对安全做更严格的校验。而从 Android N 开始，将不允许在 App 间，使用 `file://` 的方式，传递一个 File ，否者会抛出 FileUriExposedException 的错误，会直接引发 Crash。

# 解决方案

既然Android N中默认开启了严苛模式，限制了 `file://` 的方式的传递，当然也会提供响应的解决方案。

### 方法1 targetSDKVersion

把targetSDKVersion限制在23，这样Android N便不会对app进行严苛模式的校验，当然长远来讲这种方法并不可取.

### 方法2 配置fileProvider

在Manifest文件中配置fileProvider

```xml
            <!--应用间共享文件-->
            <provider
                android:name="android.support.v4.content.FileProvider"
                android:authorities="${applicationId}.fileProvider"
                android:exported="false"
                android:grantUriPermissions="true">
                <meta-data
                    android:name="android.support.FILE_PROVIDER_PATHS"
                    android:resource="@xml/file_path"/>
            </provider>
```

在`res/xml/`中创建一个xml，定义需要传递的文件加路径:

```xml
    <paths>
    
        <files-path
            name="app_internal"
            path="/"/>
    
        <cache-path
            name="app_internal_cache"
            path="/"/>
    
        <external-cache-path
            name="app_external_cache"
            path="/"/>
    
        <external-files-path
            name="app_external"
            path="/"/>
    
        <external-path
            name="sd_external"
            path="com.mondial.fashiontech"/>
    
        <external-path
            name="sd_external_dcim"
            path="DCIM"/>

    </paths>
```

paths节点下可以配置的节点名和对应的目录如下：

*   **root-path**：表示根目录，『/』。
*   **files-path**：表示 content.getFileDir() 获取到的目录。
*   **cache-path**：表示 content.getCacheDir() 获取到的目录
*   **external-path**：表示Environment.getExternalStorageDirectory() 指向的目录。
*   **external-files-path**：表示 ContextCompat.getExternalFilesDirs() 获取到的目录。
*   **external-cache-path**：表示 ContextCompat.getExternalCacheDirs() 获取到的目录。

最后在传递Uri的时候，使用FileProvider把File类型的Uri转换为content类型的，比如创建一个拍照获取图片的Intent：

```java
      static Intent makeCaptureIntent(Context context, File targetFile, String authority) {
            makeFilePath(targetFile);
            Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
            if (Build.VERSION.SDK_INT < 24) {
                Uri fileUri = Uri.fromFile(targetFile);
                intent.putExtra(MediaStore.EXTRA_OUTPUT, fileUri);
            } else {
                Uri fileUri = FileProvider.getUriForFile(context, authority, targetFile);
                intent.putExtra(MediaStore.EXTRA_OUTPUT, fileUri);
                intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_GRANT_READ_URI_PERMISSION);
            }
            return intent;
        }
```

**注意：**authority必须与xml中配置的authority一致。

---
## 引用

- [FileProvider](https://developer.android.com/reference/android/support/v4/content/FileProvider.html)
- [我想把 FileProvider 聊的更透彻一些](https://juejin.im/post/5974ca356fb9a06bba4746bc)