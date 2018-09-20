# 二维码开发

二维码框架：

- [zing](https://github.com/zxing/zxing)：纯java开发
- [ZBar](https://github.com/ZBar/ZBar)：使用了so库

开发流程：在Android中打开camera，实时的获取camera传递过来的图像字节码数据，然后传递给解码库进行解码。具体可以参考**[BGAQRCode-Android](https://github.com/bingoogolapple/BGAQRCode-Android)**项目，但是不建议直接使用此项目。