# Android 播放视频组件

## 1 播放视频的几种方法

### 1.1 使用Intent打开本地播放器

```java
    String video = "http://www.nandudu.com/hls/course/video/2/test.m3u8";
    Intent openVideo = new Intent(Intent.ACTION_VIEW);
    openVideo.setDataAndType(Uri.parse(video), "video/*");
    startActivity(openVideo);
```

### 1.2 使用VideoView

`VideoView`是通过继承自`SurfaceView`来实现的。

SurfaceView的大概原理就是在现有View的位置上创建一个新的Window，内容的显示和渲染都在新的Window中。这使得SurfaceView的绘制和刷新可以在单独的线程中进行，从而大大提高效率。这就造成了新的问题，由于SurfaceView显示在新的Window中(每个Widnow都有自己的Surface)，使得SurfaceView的显示不受View的属性控制，**不能进行平移，缩放等变换(这些变换都是通过变换canvas来实现的)，也不能放在其它RecyclerView或ScrollView中，一些View中的特性也无法使用**。

VideoView 常用的几个方法：

```
    public int getDuration()：获得所播放视频的总时间
    public int getCurrentPosition()：获得当前的位置,我们可以用来设置播放时间的显示
    public int getCurrentPosition():获得当前的位置,我们可以用来设置播放时间的显示
    public int pause()：暂停播放
    public int seekTo()：设置播放位置，我们用来总快进的时候就能用到
    public int setOnCompletionListener(MediaPlayer.OnCompletionListener l)：注册在媒体文件播放完毕时调用的回调函数。
    public int setOnErrorListener (MediaPlayer.OnErrorListener l)：注册在设置或播放过程中发生错误时调用的回调函数。如果未指定回调函数， 或回调函数返回false，会弹一个dialog提示用户不能播放
    public void setOnPreparedListener (MediaPlayer.OnPreparedListener l)：注册在媒体文件加载完毕，可以播放时调用的回调函数。
    public void setVideoURI (Uri uri)：设置播放的视频源，也可以用setVideoPath指定本地文件
    public void start()：开始播放
    getHolder().setFixedSize(width,height)：设置VideoView的分辨率
```

### 1.3 使用MediaPlayer+SurfaceView

同1.2，既然使用SurfaceView有很多的问题，所以也就不推荐此种方式了。

### 1.4 使用MediaPlayer+TextureView

TextureView是在4.0引入的，与SurfaceView相比，它不会创建新的窗口来显示内容。它是将内容流直接投放到View中，并且可以和其它普通View一样进行移动，旋转，缩放，动画等变化。**TextureView必须在硬件加速的窗口中使用**。与SurfaceView类似，TextureView创建后不能立即使用，需要添加到View树中，然后设置监听器`setSurfaceTextureListener(new TextureView.SurfaceTextureListener() `，在回调方法中使用。

```java
    TextureView.SurfaceTextureListener() {
        @Override
        public void onSurfaceTextureAvailable(SurfaceTexture surface, int width, int height) {
            // SurfaceTexture准备就绪
        }

        @Override
        public void onSurfaceTextureSizeChanged(SurfaceTexture surface, int width, int height) {
            // SurfaceTexture大小变化
        }

        @Override
        public boolean onSurfaceTextureDestroyed(SurfaceTexture surface) {
            // SurfaceTexture被销毁
            return false;
        }

        @Override
        public void onSurfaceTextureUpdated(SurfaceTexture surface) {
            // SurfaceTexture更新
        }
    }
```
上面就是TextureView对应的回调，当`onSurfaceTextureAvailable`方法被调用时，表示SurfaceTexture准备就绪已经准备就绪，这时就可以让SurfaceTexture关联到MediaPlayer，作为播放视频的图像数据来源，SurfaceTexture作为数据通道，把从数据源(MediaPlayer)中获取到的图像帧数据转为GL外部纹理，交给TextureVeiw作为View heirachy中的一个硬件加速层来显示，从而实现视频播放功能。

### 1.5 如何选择

根据上面的分析，所有一般情况下，应该MediaPlayer+TextureView来播放视频。

---
## 2 MediaPlayer

在官方文档[MediaPlayer](https://developer.android.com/reference/android/media/MediaPlayer.html)中，已经很详细的介绍了MediaPlayer的各种状态以及各种操作方法对各种状态的影响。

![](https://developer.android.com/images/mediaplayer_state_diagram.gif)

在使用MediaPlayer时，必须按照其状态进行操作，一步一步进入到播放状态，MediaPlayer可以进行同步或者异步准备操作，一般推荐使用异步的方式进入已准备状态，以防止发生ANR，在MediaPlayer的各个状态切换与播放适配的过程中，都可以通过注册各种监听器对MediaPlayer的状态以及其他信息进行监听，可用的监听器如下：

- `setOnPreparedListener(OnPreparedListener)`
- `setOnVideoSizeChangedListener(OnVideoSizeChangedListener)`
- `setOnSeekCompleteListener(OnSeekCompleteListener)`
- `setOnCompletionListener(OnCompletionListener)`
- `setOnBufferingUpdateListener(OnBufferingUpdateListener)`
- `setOnInfoListener(OnInfoListener)`
- `setOnErrorListener(OnErrorListener)`


---
## 引用

- [mediaplayer](https://developer.android.com/guide/topics/media/mediaplayer.html?hl=zh-cn)
- [Android 5.0(Lollipop)中的SurfaceTexture，TextureView, SurfaceView和GLSurfaceView](http://blog.csdn.net/jinzhuojun/article/details/44062175)
- [用MediaPlayer+TextureView封装一个完美实现全屏、小窗口的视频播放器](http://www.jianshu.com/p/420f7b14d6f6)