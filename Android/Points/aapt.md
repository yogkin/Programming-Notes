# aapt


## 1 Android中appt工具使用介绍

aapt即`Android Asset Packaging Tool`，我们可以在SDK的**platform-tools**目录下找到该工具。aapt可以查看、 创建、 更新ZIP格式的文档附件(zip, jar, apk)。 也可将资源文件编译成二进制文件，尽管可能开发中没有直接使用过aapt工具，但是build scripts和IDE插件会使用这个工具打包apk文件构成一个Android 应用程序。

## 2 appt工具基本功能使用介绍

appt工具功能主要又一下几点功能：

- `aapt l[ist]`:**列出资源压缩包里的内容**。
- `aapt d[ump]`：**查看APK包内指定的内容**。
- `aapt p[ackage]`：**打包生成资源压缩包**。
- `aapt r[emove]`：**从压缩包中删除指定文件**。
- `aapt a[dd]`：**向压缩包中添加指定文件**。
- `aapt v[ersion]`:**打印aapt的版本**。

上面只是简单的列出了aapt具有的功能，具体命令可以在终端使用appt命令查看帮助。

>如果aapt命令显示的信息过度，可以将文本信息重定向到文本中去。

## 3 付详细的命令

## 列出zip文件中的文档

```
      aapt l[ist] [-v] [-a] file.{zip,jar,apk}
```

## 显示apk文件的各种信息，(注意各个选项的先后顺序)

```
     aapt d[ump] [--values] [--include-meta-data] WHAT file.{apk} [asset [asset ..]

       strings          打印资源表中的字符串池
       badging          打印AOK中的label and icon
       permissions      打印APK申请的权限
       resources        Print the resource table from the APK.
       configurations   Print the configurations in the APK.
       xmltree          Print the compiled xmls in the given assets.
       xmlstrings       Print the strings of the given compiled xml assets.
```

## 打包Android资源  It will read assets and resources that are

```
       supplied(提供的) with the -M -A -S or raw-files-dir arguments.  The -J -P -F and -R
       options control which files are output.

     aapt p[ackage] [-d][-f][-m][-u][-v][-x][-z][-M AndroidManifest.xml] \
            [-0 extension [-0 extension ...]] [-g tolerance] [-j jarfile] \
            [--debug-mode] [--min-sdk-version VAL] [--target-sdk-version VAL] \
            [--app-version VAL] [--app-version-name TEXT] [--custom-package VAL] \
            [--rename-manifest-package PACKAGE] \
            [--rename-instrumentation-target-package PACKAGE] \
            [--utf16] [--auto-add-overlay] \
            [--max-res-version VAL] \
            [-I base-package [-I base-package ...]] \
            [-A asset-source-dir]  [-G class-list-file] [-P public-definitions-fil
     \
            [-D main-dex-class-list-file] \
            [-S resource-sources [-S resource-sources ...]] \
            [-F apk-file] [-J R-file-dir] \
            [--product product1,product2,...] \
            [-c CONFIGS] [--preferred-density DENSITY] \
            [--split CONFIGS [--split CONFIGS]] \
            [--feature-of package [--feature-after package]] \
            [raw-files-dir [raw-files-dir] ...] \
            [--output-text-symbols DIR]
```

## 添加或者移除资源

```
     aapt r[emove] [-v] file.{zip,jar,apk} file1 [file2 ...]
       Delete specified files from Zip-compatible archive.

     aapt a[dd] [-v] file.{zip,jar,apk} file1 [file2 ...]
       Add specified files to Zip-compatible archive.
```

## 列出版本

```
     aapt v[ersion]
       Print program version.
```

## 其他

```
     aapt c[runch] [-v] -S resource-sources ... -C output-folder ...
       Do PNG preprocessing on one or several resource folders
       and store the results in the output folder.
     aapt s[ingleCrunch] [-v] -i input-file -o outputfile
       Do PNG preprocessing on a single file.
```