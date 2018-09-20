# Camera开发

使用Camera可以拍摄照片和录制视频，有两种方式可以选择，第一种方式就是调用系统相机，而第二种方式就是根据Android提供的API开发自己的相机。此笔记记录的是第二种方式。

---
## 1 Camera介绍

**Camera开发涉及的技术点**

- 判断设备是否有摄像头或者直接过滤没有相机的设备
- Camera的API，5.0之后推荐使用Camera2
- 摄像头的个数(前后、双摄)
- 打开摄像头，摄像头是公共资源，如果有其他应用正在使用摄像头，则会导致摄像头打开失败
- 防止拍照变形
 -  SurfaceView尺寸：即自定义相机应用中用于显示相机预览图像的View的尺寸，手机预览图像
 -  Previewsize：相机硬件提供的预览帧数据尺寸，预览帧数据传递给SurfaceView，实现预览图像的显示，相机预览图像
 -  Picturesize：相机硬件提供的拍摄帧数据尺寸。拍摄帧数据可以生成位图文件，最终保存成.jpg或者.png等格式的图片，相机拍摄图像
- Camera的配置
 - 闪光灯
 - 对焦模式
 - 场景模式
 - 相机的预览方向：默认的预览方向、自拍时设置的预览方向

**相机开发步骤**

1. 检查是否存在相机，在manifest文件中声明权限
2. 创建SufaceView与holder，监听surfaceView生命周期
3. 在回调用开启相机，根据屏幕方向设置相机方向，调整预览尺寸与图片尺寸，自动对焦等
4. 实现拍照，获取图片、优化优化、保存图片。
5. 在合适的时机回收相机

**需要注意的地方**

1. camera需要先关联SurfaceView才能打开
2. camera默认打开的是后置摄像头，前置摄像头需要做好相关判断
3. 关联SrufaceView后就可以开启相机进行预览了，相机默认的取景色方向是横屏方向，如果是竖屏应用看到景象是与手机屏幕垂直的。这是需要设置相机的取景方向。
4. 相机中有预览尺寸、图片尺寸，而我们有显示预览图形的SurfaceView，为了保证相机拍摄的照片不变形，最好保证SurfaceView、预览尺寸、图片尺寸的宽高比例保持一致，相关参数可以调用Camera的Parameters设置
5. 防止设置Parameters是的各种崩溃:try-catch
6. 拍摄后的照片方向问题，有可能是90°、有可能是0°，使用**`ExifInterface`**进行获取与设置。


### 1.1 检查相机与申请权限

```xml
    //camear权限
    <uses-permission android:name="android.permission.CAMERA"/>
    //要求设备有相机
    <uses-feature
        android:name="android.hardware.camera"
        android:required="true"/>
    <uses-feature android:name="android.hardware.camera.autofocus" android:required="false"/>
```

检查设备是否存在相机

```
    /** 检查设备是否存在相机*/
    private boolean checkCameraHardware(Context context) {
        if (context.getPackageManager().hasSystemFeature(PackageManager.FEATURE_CAMERA)){
            // this device has a camera
            return true;
        } else {
            // no camera on this device
            return false;
        }
    }
```

### 1.2 Camera配置

#### 对焦

对焦的方案：

1. `FOCUS_MODE_CONTINUOUS_PICTURE`(某些机型不支持)
2. 手动触发调用`autoFocus()`
3. 利用传感器判断手机一定，调用`autoFocus()`自动对焦

####   防止拍照变形

- SurfaceView尺寸：即自定义相机应用中用于显示相机预览图像的View的尺寸，手机预览图像
- Previewsize：相机硬件提供的预览帧数据尺寸，预览帧数据传递给SurfaceView，实现预览图像的显示，相机预览图像
- Picturesize：相机硬件提供的拍摄帧数据尺寸。拍摄帧数据可以生成位图文件，最终保存成.jpg或者.png等格式的图片，相机拍摄图像

可能存在的三种变形：

1. 手机预览画面中物体被拉伸变形。
2. 拍摄照片中物体被拉伸变形。
3. 点击拍照瞬间，手机预览画面会停顿下，此时的图像是拉伸变形的，然后预览画面恢复后图像又正常了。

变形原因：

- 现象1：SurfaceView和Previewsize的长宽比率不一致，因为手机预览视图的图像是由相机预览图像根据SurfaceView大小缩放得来的当长宽比不一致时必然会导致图像变形
- 现象2与3：原因则是Previewsize和Picturesize的长宽比率不一致所致，与具体原因跟某些手机相机硬件的底层实现有关

总之为了避免以上几种变形现象的发生，在开发时最好将`SurfaceView、PreviewSize、PictureSize`三个尺寸保证长宽比例一致

下面是魅族MX5的相机size信息：

supportedPicResolutions

    size:  w = 5248  , h = 3936
    size:  w = 4672  , h = 3504
    size:  w = 4608  , h = 2592
    size:  w = 4080  , h = 2448
    size:  w = 3648  , h = 2192
    size:  w = 3264  , h = 2448
    size:  w = 2624  , h = 1968
    size:  w = 2560  , h = 1920
    size:  w = 2896  , h = 1632
    size:  w = 2336  , h = 1760
    size:  w = 2672  , h = 1504
    size:  w = 2560  , h = 1440
    size:  w = 2048  , h = 1536
    size:  w = 1600  , h = 1200
    size:  w = 1280  , h = 960
    size:  w = 1280  , h = 768
    size:  w = 1280  , h = 720
    size:  w = 1024  , h = 768
    size:  w = 640  , h = 480
    size:  w = 320  , h = 240

supportedPreviewSizes

    size:  w = 1920  , h = 1152
    size:  w = 1920  , h = 1088
    size:  w = 1920  , h = 1080
    size:  w = 1536  , h = 1152
    size:  w = 1280  , h = 720
    size:  w = 960  , h = 720
    size:  w = 960  , h = 540
    size:  w = 800  , h = 600
    size:  w = 864  , h = 480
    size:  w = 800  , h = 480
    size:  w = 720  , h = 480
    size:  w = 640  , h = 480
    size:  w = 480  , h = 368
    size:  w = 480  , h = 320
    size:  w = 352  , h = 288
    size:  w = 320  , h = 240
    size:  w = 176  , h = 144

具体的开发过程中，应该如何做呢？

1. 根据设备支持的分辨力，提供相片的比例设置，比如`16:9，4:3`
2. 设置预览View的横纵比
3. 根据设置的比例找到最合适的预览分辨力和图片分辨率，存在多个分辨率的时候，一般选择最大的即可。
4. 如果需要正方形的拍照效果，一般的做法是使用一个遮罩把预览视图遮成正方形的(实际的预览还是原来的比例)，在拍照完后再对数据进行处理，把照片处理成正方形的

#### 预览方向

1. 相机的默认的成像角度
2. 横屏与竖屏应用
3. 前置摄像头的成像角度

####  照片的EXFI信息

招聘拍摄完毕需要设置照片的元信息，比如地点，设备，角度等。这些都可以使用Android提供的EXFI(Exchangeable Image File)接口设置。

实例：

```
     ExifInterface exifInterface = new ExifInterface(path);
                //设置照片旋转90°
     exifInterface.setAttribute(ExifInterface.TAG_ORIENTATION,      String.valueOf(ExifInterface.ORIENTATION_ROTATE_90));
     exifInterface.saveAttributes();//一定要记得保存，否则没有效果的。
```

或者可以使用support中的exfi库

---
## 2 Camera2

- [ ] todo

---
## 3 封装API

相机应该只提供相机功能，而具体的图片压缩则应该交给对应的压缩模块，这样很大程度上减轻了相机开发的负担。

在设计API时，要根据具体的需求开发，并且确定你封装这个API是提供什么样的功能，总之如果不是去开发一个完整的相机应用，就不要想着把所有的功能都封装进去，毕竟相机这东西涉及的方面太多了。总计就是一句话**确定你封装这个API是提供什么样的功能**。


---
##  引用

###  Blog

- [官方文档](https://developer.android.com/guide/topics/media/camera.html#considerations)
- [官方教程](https://developer.android.com/training/camera/index.html)
- [安卓相机开发那些坑](https://www.zhangningning.com.cn/blog/Android/android_camera_dev.html)
- [Android Camera使用总结与那些坑](https://mp.weixin.qq.com/s?__biz=MzI3MDE0NzYwNA==&mid=2651434751&idx=1&sn=4154b06e059e17a983053a2c4806c24c&chksm=f1288784c65f0e92a443a4112c1f808db72a8595b8ba0cae9d36c661e9bb830e554b0ff87664&mpshare=1&scene=1&srcid=0524pUYORU8DT7LuZ3okUIcQ#rd)
- [Camera基础介绍](http://www.jianshu.com/p/96fc616b7778)
- [Android 自定义拍照，解决图片旋转，拍照参数设置兼容问题](http://www.cnblogs.com/xiaoxiao-study/p/867d2ad9206c8600186c90690f1e7965.html)
- [Android ExifInterface 学习笔记，图片旋转的操作。](http://blog.csdn.net/liuhanhan512/article/details/8277637)
- [EXIF](http://baike.baidu.com/item/EXIF)
- [Camera实时滤镜](http://cblog.cc/2015/09/03/Android-Camera-%E5%AE%9E%E6%97%B6%E6%BB%A4%E9%95%9C/)
- [Android相机实时自动对焦的完美实现](http://blog.csdn.net/huweigoodboy/article/details/51378751)
- [Android Camera 相机开发详解](http://www.jianshu.com/p/7dd2191b4537)
- [camera-is-being-used-after-camera-release-was-called](https://stackoverflow.com/questions/38015019/camera-is-being-used-after-camera-release-was-called-in-galaxy-s6-edge)
- [拍照时在相机的预览界面指定一个区域的大小，形状和位置，只拍摄该指定区域里的图像](https://github.com/CGmaybe10/FocusSurfaceView)
- [Android相机开发系列](https://www.polarxiong.com/archives/Android%E7%9B%B8%E6%9C%BA%E5%BC%80%E5%8F%91%E7%B3%BB%E5%88%97.html)
- [Android 相机预览方向及其适配探索](https://dev.qq.com/topic/583ba1df25d735cd2797004d)
- [Android Camera 模型及 API 接口演变](https://glumes.com/post/android/android-camrea-api-evolution/)

### App和库

- [Fotoapparat](https://github.com/Fotoapparat/Fotoapparat)
- [CameraFragment](https://github.com/florent37/CameraFragment)
- [CameraKit-Android](https://github.com/gogopop/CameraKit-Android)
- [Sticker Camera](https://github.com/Skykai521/StickerCamera)
- [Focal](https://github.com/xplodwild/android_packages_apps_Focal)
- [Magic Camera](https://github.com/wuhaoyu1990/MagicCamera)
- [CameraView](https://github.com/natario1/CameraView)
- [PhotoNoter](https://github.com/yydcdut/PhotoNoter)
- [Android 摄像头实时滤镜](https://github.com/WeLikeVis/CameraFilter)
- [AliCamera是基于Android4.4源码中Camera2重新开发的一款全新相机应用](https://git.oschina.net/way/AliCamera)

### 二维码

- [zxing二维码](https://github.com/zxing/zxing)
- [直接拿来用的二维码View](https://github.com/dm77/barcodescanner)
- [QrCodeScan](https://github.com/SkillCollege/QrCodeScan)
- [BGAQRCode-Android](https://github.com/bingoogolapple/BGAQRCode-Android)
- [高度定制支持屏幕旋转](https://github.com/ThePacific/zxing-barcode)
- [barcodescanner](https://github.com/dm77/barcodescanner)