# 人脸识别与人脸检测

---
## 1 人脸识别与人脸检测介绍

>人脸检测（face detection）是一种在任意数字图像中找到人脸的位置和大小的计算机技术。它可以检测出面部特征，并忽略诸如建筑物、树木和身体等其他任何东西。[1]有时候，人脸检测也负责找到面部的细微特征，如眼睛、鼻子、嘴巴等的精细位置。——维基百科

>人脸识别，是基于人的脸部特征信息进行身份识别的一种生物识别技术。用摄像机或摄像头采集含有人脸的图像或视频流，并自动在图像中检测和跟踪人脸，进而对检测到的人脸进行脸部的一系列相关技术，通常也叫做人像识别、面部识别。——百度百科

---
## 2 可选的技术方案

### 1 使用Android Camera提供的API

Android4.0之后Camera提供了基于Google算法的人脸追踪。功能相对简单，但某些机型无法使用(各种定制，你懂的)。

具体如何集成，可以参考:

- [玩转Android Camera开发(五):基于Google自带算法实时检测人脸并绘制人脸框(网络首发，附完整demo)](http://blog.csdn.net/yanzi1225627/article/details/38098729)
- [WoodyFaceDetection](https://github.com/blundell/WoodyFaceDetection)

### 2 使用开源的人脸识别算法

查了一下[知乎](https://www.zhihu.com/question/19561362)，貌似目前并没有比较成熟的开源方案，下面是一些搜索记录

1. 使用[使用OpenCV Android SDK从摄像头帧实时检测人脸](http://blog.lwons.com/archieve/use_opencv_in_android_to_detect_faces_from_camera_frames.html)
2. [FaceDetector](https://github.com/Fotoapparat/FaceDetector)
3. [faceCapture](https://github.com/Li-Shang/faceCapture)+dlib


### 3 使用第三方技术支持

1. 使用[MobileVision](https://developers.google.com/vision/introduction)，但是MobileVision需要GooglePlay的支持，所在国内环境不适合使用
2. [科大讯飞](http://www.xfyun.cn/)
    - 免费
    - 支持在线和离线识别
    - 目前支持的分辨率有限
3. [相芯科技](http://www.faceunity.com/stickers.html)，需要付费
4. [商汤科技](https://www.sensetime.com/index/)，需要付费，效果非常好
5. [虹软科技](http://www.arcsoft.com.cn/index.html)，永久免费，集成可以参考[Android人脸识别开发入门--基于虹软免费SDK实现](http://www.jianshu.com/p/75733cff88a3)
6. [Aliyun](https://data.aliyun.com/product/face?spm=5176.8142029.388261.73.oMTRit) 需要付费，不支持离线识别