# Android 技能总结

---
## 1 Android基础

- Activity
    - 生命周期
    - 启动模式
    - 状态保存
    - 任务栈
- Service
    - 生命周期
    - 基本使用
    - 启动方式
    - Service与Activity通信
- ContentProvider
    - 编写自己的ContentProvider
    - 通过ContentProvider获取系统资源
    - 图片/音频/视频/联系人/系统账户等获取
- Broadcast
    - 本地广播
    - 有序广播
    - 无顺广播
    - 优先级
- 四大组件在manifest.xml中配置
- 掌握Fragment的使用与常见问题
- Intent
    - Intent传递数据
    - Intent的flag
- 传感器
    - 方向传感器
    - 加速度传感器
- 多媒体API使用
    - 音频播放
    - 视频播放
    - 拍照与录像
- Sqlite的使用
    - Cursor
    - 其他数据库API
- Handler机制
- AsyncTask原理
- HandlerThread原理
- IntentService使用
- Notification使用
- 桌面Widget的开发
- 理解RomateView
- 系统权限
    - 静态权限
    - 动态权限
- Android的数据存储
- 屏幕适配
- 常用UI控件使用
- ActivityManager
- PackageManager
- adb、aapt、dex、shell等命令

---
## 2 Android网络技术

- socket
- tcp/ip
- udp
- http
- https
- 证书
- 网络优化
- 即时通讯、直播

---
## 3 View体系

### View的绘制流程

- ViewRootImpl
- View树的构建
- View的测量
- View的布局
- View的绘制
- 绘制流程源码分析

### 事件分发

- ViewGroup的事件分发
- View的事件分发
- 事件分发的流程与源码分析
- 滑动冲突的解决
- MotionEvent

### 自定义控件

- 绘制控件
- 组合控件
- 自定义Layout
- Scroller与OvserScroller
- ViewDragHelper
- VelocityTracker
- ViewConfiguration
- GestureDetector
- 滑动的实现方式
- 事件冲突的解决
- 嵌套滑动
- 重写dispatchTouchEvent

### 绘图技能

- Canvas
    - 绘图操作
    - 画布变换
    - 画布裁剪
    - 图层操作
    - drawBitmapMesh
- Paint
    - Paint常用API
    - Paint的Shader:五大着色器
    - xfermode：图形混合模式
    - mask 滤镜效果，模糊滤镜与浮雕滤镜
    - 字体绘制与测量
    - 颜色矩阵
    - 颜色混合模式
- Path
    - 各种Path基础API
    - 填充模式
    - 贝塞尔曲线。使用贝塞尔曲线组合圆形
- PathMeasure
- Rect与Region
- Bitmap像素的分析
- 自定义Drawable
- Matrix
- Camera投影

### 动画

- 帧动画
- 补间动画、补间动画原理、补间动画xml写法、自定义动画
- 属性动画
- 属性动画xml写法
- 属性动画源码
- SVG图片、SVG动画
- GIF图片
- 转场动画

### MaterialDesign

- Touch Feedback
- Reveal effect 揭露效果
- Activity Transitions
- Fragment Transitions
- ShareElement
- Curved motion 曲线运动
- View state changes
- Animate Vector Drawable 矢量动画

### MaterialDesingn库

- MaterialDesingn theme/style
- DrawerLayout
- NavigationView
- TextInputLayout
- Snackbar
- Toolbar
- 百分比控件
- 半透明状态栏
- Palette
- FloatActionButton
- CardView
- CoordinateLayout
- AppBarLayout
- CollcapsingToolbarLayout
- CoordinateLayout原理、自定义Behavior

### 常用控件

- TextView、EditText、富文本编辑、各种Span
- RecyclerView技术栈
- Drawable原理
- WebView
- ViewPager
- SurfaceView
- TextureView
- ConstraintLayout

### UI资源

- 常用的系统theme和style属性
- theme和style原理
- 自定义theme和style
- xml drawable

### 扩展

- 下拉刷新
- 自动加载
- 视差效果
- 图片编辑
- 图片模糊效果
- 引导页特性

---
## 4 Android性能优化

### 内存泄漏分析与内存优化

- 发生OOM的条件分析
- 避免内存泄漏（如何使用更高效的ArrayMap容器、如何避免不经意的“自动装箱”、Lint，StictMode等工具的使用技巧）
- 内存管理机制（共享内存、分配与回收内存、限制应用的内存、应用切换操作）
- OOM（查看内存使用情况）
- onLowMemory与onTrimMemory的回调

### 工具

- MAT
- LeakCanary
- Memory Monitor
- Allocation Tracking
- Heap Tool
- TraceView
- hierarchyviewer
- MemoryAnalyzer(第三方)
- GT Home(第三方)
- iTest(第三方)

### UI优化

- 列表优化
- 预渲染技术
- 绘图优化
- 布局优化
- UI卡顿分析、过度渲染问题
- Bitmap优化
    - 缩放性能优化
    - 缓存性能优化
    - 重用性能优化
    - PNG压缩性能优化
    - 微信图片终极压缩方案问题

### 网络优化

- Batching批处理技术
- Prefetching预取技术
- GCMNetworkManager高级实践
- Network Traffic Tool工具的使用
- 数据传输：FlatBuffers、WEBP格式图片使用、7Zip极限压缩

### 电量优化

- 分析电量的流失
- 分析电量消耗数据
- 分析充电状态和电池管理
- battery-historian工具的使用
- 窝信号对电量消耗
- Job Schedule

### 构建优化

- 打包流程分析
- aapt资源文件打包原理
- resources_arsc二进制机构分析
- 资源文件压缩、资源动态加载
- Lint工具优化
- 极限压缩
- Proguard混淆

### Service的调优

- 进程常驻
- 优化后台服务的内存消耗

### 其他

- 数据库优化
- 性能优化

### 线程优化

- 线程间通讯
- AsyncTask源码级分析及注意
- HandlerThread的处理
- IntentService使用场景分析和实践
- ThreadPool使用场景和注意
- 多进程APP


---
## 5 JNI与NDK编程(todo)

- C&C++
- JNI
- NDK
- Linux系统编程
- FFmpeg
- OpenSl ES
- opencv

---
## 5 Android架构

### 熟悉Android系统底层

- Apk的安装过程、Dex
- Context原理
- 四大组件的启动流程
- Android资源访问机制、ClassLoader
- InputManager工作原理
- ServiceManage启动流程
- AMS原理
- WMS原理，窗口的创建过程
- 系统的启动流程
- Dalvik和ART
- Root原理

### Android架构师

- MVC
- MVP
- MVVM
- Clean架构
- Databinding
- UML建模
- Android组件化、插件技术、热修复技术

### IPC

- IPC
- Binder机制
- Bindder连接池
- 对象序列化/AIDL
- Messenger原理


### 构建与安全

- AndroidStudio
    - 快捷键
    - 各种分析器
    - 常用插件
- Git版本管理
- Gradle深入
- Gradle构建、Gradle插件
- APK编译流程
- App混淆
- App反编译


---
## 8 Android第三方框架

- RxJava/RxAndroid
    - RxBindding
    - RxLife
- Dagger2
- Okhttp3
- Stetho
- Retorfit
- LeakCanary
- ButterKinfe
- xStream
- Gson/FastJson
- Glide等四大图片加载框架
- xposed
- 热插件框架
- 热修复框架

---
## 9 Android第三方服务

- IM服务
- 数据存储
- 第三方分析与统计
- 消息推送
- Bug管理
- 支付
- 云服务
- APP测试

---
## 10 Android工具类集合

- Bitmap常用工具类
- 资源访问
- 文件操作
- 加密解密
- 网络操作
- AppVersion/AppCompat
- 系统资源获取，照片/文件
- sp工具类
- 缓存工具类
- Adapter封装
- 常用基类封装
- Delegate思想
- 日志管理
- Crash处理
- 键盘处理
- UI单位处理
- 字符串工具类
- 日期工具类
- 数字处理工具类


---
## 12 跨平台Android开发方式

- ReactNative
- Flutter


