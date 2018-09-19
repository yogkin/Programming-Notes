## 开启严格模式

使用严格模式可以确保你不会在主线程上做一些不应该做的事情。记住要在发布版本中把该模式关闭，如果你忘记关掉该模式，那么它会影响性能、导致程序崩溃。

```
   if (BuildConfig.DEBUG) {
     StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
           .detectAll()
           .penaltyLog()
           .build());
     StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
           .detectAll()
           .penaltyLog()
           .penaltyDeathOnNetwork()
           .build());
    }
```

---
## windowSoftInputMode与windowFullscreen，fitsSystemWindows冲突

如果你在 manifest 中把一个 activity 设置成 `android:windowSoftInputMode="adjustResize"`，那么 `ScrollView`（或者其它可伸缩的 `ViewGroups`）会缩小，从而为软键盘腾出空间。但是，如果你在 activity 的主题中设置了 `android:windowFullscreen="true"`，那么`ScrollView` 不会缩小。这是因为该属性强制 `ScrollView` 全屏显示。然而在主题中设置 `android:fitsSystemWindows="false"` 也会导致 `adjustResize` 不起作用。

---
## 注意对隐式Intent的运行时检查保护

类似打开相机，发送图片等隐式Intent，是并不一定能够在所有的Android设备上都正常运行。例如打开相机的隐式Intent，如果系统相机应用被关闭或者不存在相机应用，又或者是相机应用的某些权限被关闭等等情况都可能导致这个隐式的Intent无法正常工作。一旦发生隐式Intent找不到合适的调用组件的情况，系统就会抛出`ActivityNotFoundException`的异常，如果我们的应用没有对这个异常做任何处理，那应用就会发生Crash。

```
    Intent intent = new Intent(Intent.ACTION_XXX);
    ComponentName componentName = intent.resolveActivity(getPackageManager());
    if(componentName != null) {
        String className = componentName.getClassName();
    }
```

---
## 使用 ActivityManager 清理用户数据

```
      public boolean clearApplicationUserData() {
            return clearApplicationUserData(mContext.getPackageName(), null);
        }
```

