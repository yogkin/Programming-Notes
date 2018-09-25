# Bitmap 总结

---
## 1 Bitmap

Android中使用Bitmap表示一张图片，常用的图片格式如下：

- png格式：高质量的图片，用于计算机和网络(文件的体积比较小)，采用无损的图形压缩算法
- JPG格式：良好质量的图片，用于计算机和网络(文件体积也比较小)，图形压缩算法类似于rar算法，眼睛的精度有限，比较深的红色和稍微淡点的红色 人眼分辨不出，就把相邻空间，类似的颜色统一用同一种颜色表示，降低了图片的精度。所以jpg采用的是又算压缩算法。

由于Bitmap在Android中比较特殊，稍有不慎就有可能造成内存溢出，这是因为安卓一般每个应用程序都是一个独立的进程，操作系统会给进程创建一个Dalvik虚拟机，虚拟机都有自己的内存限制(比如16M)，如果应用程序需要的空间超过了这个内存限制 就会抛出异常。所以在Android开发中对于Bitmap的处理，需要额外的小心。

---
## 2 bitmap的内存占用与优化

### BitmapConfig说明

```
    android图片压缩质量参数 Bitmap.Config.RGB_565 或 ARGB_8888

    android中的大图片一般都要经过压缩才显示，不然容易发生oom，一般我们压缩的时候都只关注其尺寸方面的大小，其实除了尺寸之外，影响一个图片占用空间的还有其色彩细节。

    打开Android.graphics.Bitmap类里有一个内部类Bitmap.Config类，在Bitmap类里createBitmap(intwidth, int height, Bitmap.Config config)方法里会用到，打开个这个类一看

        枚举变量
            public static final Bitmap.Config ALPHA_8
            public static final Bitmap.Config ARGB_4444
            public static final Bitmap.Config ARGB_8888
            public static final Bitmap.Config RGB_565

    一看，有点蒙了，ALPHA_8, ARGB_4444,ARGB_8888,RGB_565 到底是什么呢？

    其实这都是色彩的存储方法：我们知道ARGB指的是一种色彩模式，里面A代表Alpha，R表示red，G表示green，B表示blue，其实所有的可见色都是右红绿蓝组成的，所以红绿蓝又称为三原色，每个原色都存储着所表示颜色的信息值

    说白了就ALPHA_8就是Alpha由8位组成
        ARGB_4444就是由4个4位组成即16位，
        ARGB_8888就是由4个8位组成即32位，
        RGB_565就是R为5位，G为6位，B为5位共16位

    由此可见：
        ALPHA_8 代表8位Alpha位图
        ARGB_4444 代表16位ARGB位图
        ARGB_8888 代表32位ARGB位图
        RGB_565 代表8位RGB位图

    位图位数越高代表其可以存储的颜色信息越多，当然图像也就越逼真。
```

### 优化策略

1. 首先bitmap的内存占用计算：计算机加载图片所需内存与图片的大小无关，与图片的分辨率有关，比如分辨率为 2500*1500的图片，使用Bitmap.Config ARGB_8888的压缩参数加载，所需要的内存空间是 2500*1500*4。而计算机显示图形实际上是显示图形的每个像素点，所以受计算机或手机的屏幕分辨率的限制，没有必要把所有的像素都加载到内存里面。而是显示一个缩略图。
2. 可以对bitmap进行缓存，以减少不必要的内存分配开销与网络请求等。
3. 对Bitmap进行适当的压缩。

所以bitmap的相关知识点可以总结如下：

1. bitmap的高效加载，如何防止内存溢出
2. bitmap的缓存，包括磁盘缓存和内存缓存，LRU算法实现
3. bitmap的压缩算法

### 压缩方式

1. 质量压缩：compress，该压缩不减少像素
2. 尺寸压缩：canvas.drawBitmap
3. 采样率压缩：inSampleSize

---
## 3 如何高效的加载Bitmap

Android提供了一下BitmapFactory来加载一种图片：

注意有以下方法：
```
     public static Bitmap decodeByteArray(byte[] data, int offset, int length, Options opts) 
     public static Bitmap decodeFile(String pathName, Options opts)
     public static Bitmap decodeStream(InputStream is, Rect outPadding, Options opts)
     public static Bitmap decodeResource(Resources res, int id, Options opts)
```
分别对应于**字节数组**，**文件体统**，**字节流**，**资源文件**

如果高效的加载一张图片呢？其实就是使用BitmapFactory.Options，主要的流程如下：
- 由于bitmap需要一个控件来显示，而这个控件是有宽高的，所以先获取这个控件的宽高
- 然后通过Options对Bitmap进行抽样，把BitmapFactory.Options的inJustDecodeBuounds设置为true，这样BitmapFactory分析图片的宽高信息，而不会真的把图片加载到内存中去，
- 根据抽样获取的bitmap宽高信息和空间的宽高信息，计算出一个合适的缩放值。
- 最后把BitmapFactory.Options的inJustDecodeBuounds设置为false,把计算的缩放值赋值给OPtions.inSampleSize，把图片加载到内存中来。

>**关于inSampleSize说明：**当inSampleSize为1时，加载的图片和原始图片的宽高信息一样，当值为2时，那么加载后的bitmap的宽高为原始图片宽高的一半，，那么占有的内存为原始图片的1/4,当值为4时，宽高为原来的1/4，而内存占有为原来的1/16，可以看出随着inSampleSize的变化，内存占有按照缩放值的2次方进行缩小，另外官方文档推荐inSampleSize的值总是为2的指数，比如1，2，4，8，16等等，当inSampleSize小于1时，其缩放值相当于1，并没有什么效果。！如果要对bitmap表示的图片进行放大操作，应该使用matrix

另外图片被加载到内存的大小还与他所在的资源文件夹有关。这是由于android对资源文件的加载机制有关。

上述步骤用代码实现：

```java
    BitmapFactory.Options options = new BitmapFactory.Options();
            options.inJustDecodeBounds = true;

            int measuredHeight = mImageView.getMeasuredHeight();
            int measuredWidth = mImageView.getMeasuredWidth();
            Log.d(TAG, "measuredHeight:" + measuredHeight);
            Log.d(TAG, "measuredWidth:" + measuredWidth);
            BitmapFactory.decodeResource(getResources(), R.drawable.welcome, options);
            Log.d(TAG, "options.outHeight:" + options.outHeight);
            Log.d(TAG, "options.outWidth:" + options.outWidth);
    
            int sampleSize = 1;
            int heightSampleSize = 1;
            int widthSampleSize = 1;
            if (options.outWidth > measuredWidth) {
                widthSampleSize = options.outWidth / measuredWidth;
            }
            if (options.outHeight > measuredHeight) {
                heightSampleSize = options.outHeight / measuredHeight;
            }
            sampleSize = Math.min(heightSampleSize, widthSampleSize);
            Log.d(TAG, "sampleSize:" + sampleSize);
    
            options.inJustDecodeBounds = false;
            options.inSampleSize = sampleSize;
            Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.welcome, options);
            mImageView.setImageBitmap(bitmap);
```

上面只是对 inSmapleSize 进行简单的计算，真实的开发中需要考虑更多。

---
## 4 Bitmap的缓存策略

缓存策略在Android中应用的非常广泛，主要考虑原因是用户的流量节省与使用体验方面，缓存主要包括内存缓存和磁盘缓存

- 磁盘缓存，通过网络加载到图片后，根据图片的唯一的url计算出一个key，根据此key把图片存储在本地，下次加载图片的时候，先在本次磁盘上加载图片，只要图片的url没有发生改变，就可以加载到此图片，很大程度上节省了用户的流量，也在一定程度上优化了用户统一，因为从本地加载一张图片肯定比从网络加载的更快
- 内存缓存 内存缓存是把加载到的图片缓存到内存中，下次加载图片的时候，先从内存中加载，这样的加载速度是非常快的，而且在一定程度上优化了开辟内存创建对象的开销

对于缓存的操作主要包括-(添加，删除，获取），添加与获取比较好理解，对于删除，是因为无论是磁盘缓存和内存缓存，大小都是有限制的。我们不可能无限制的使用磁盘空间和内存空间，所以需要当缓存的大小达到一定程度时，需要优先删除使用率不高或者占用控件较大的缓存。这涉及到一些缓存算法了


### LRU(lease recently used)

LRU是缓存算法的一种，也叫最近最少使用算法，核心思想是当缓存满时，优先淘汰哪些近期最少使用的缓存对象，采用LRU算法的缓存有两种：
- LruCache 用于实现内存缓存
- DiskLruCache 用于实现磁盘缓存
通过上面的两个lru的实现，我们就可以写出一个不错的图片缓存工具了。

#### LruCache的使用

LruCache是Android3.1提供的一个缓存工具类，通过v4包可以兼容到早期的Android版本，

LruCache是一个泛型类，它内部采用一个LinkedHashMap以强引用的方式存储外界的缓存对象，其提供的get和put方法用于完成缓存的添加与获取，当缓存满是，LruCache会移除较早使用的缓存对象。

LruCache的使用笔记简单，代码如下：
```
      LruCache<String, Bitmap> bitmapLruCache = new LruCache<String, Bitmap>(size) {
                @Override
                protected int sizeOf(String key, Bitmap value) {
                    return (value.getRowBytes() * value.getHeight()/1024);//byte to kb
                }
            };
        }
```
我们只需要指定泛型，然后重写sizeOf方法即可

####  DiskLruCache的使用

DiskLruCache得到了官方的推荐，但是并不是sdk源码的一部分，通过google我们可以快速的搜到源码。下面从DiskLruCache的创建，缓存查找，和添加提供三个方面来解释DiskLruCache的使用方式。


DiskLruCache的创建：

```
      public static DiskLruCache open(File directory, int appVersion, int valueCount, long maxSize)
          throws IOException {......}
```

DiskLruCache通过其静态方法获取对象，其四个参数分别为：

- directory 用于存储缓存文件的目录。
- appVersion app的版本号，如果两次创建传入的版本号不一致，那么前一此存储的文件都会被删除，但是考虑到应用升级时，我们仍然希望保留上个版本的缓存，所以这个值一般传1即可。
- valueCount 此值表示单个节点所对应数据的个数。一般传1即可
- maxSize 为磁盘缓存的总大小，比如50M,

#### DiskLruCache的缓存添加

DiskLruCache的缓存添加的操作是用过Editor完成的，Editor表示对一个缓存操作的编辑对象，这里以图片操作为例，首先获取url所对应的key，然后根据key可以通过edit()来获取Editor对象，如果这个缓存对象正在被编辑，那edit()返回null，即DiskLruCache不允许同时编辑同一个对象，之所以把url转换成key，是因为url可能存在特殊字符，不便于使用，代码实现如下：

```java
    //计算key值
    public static String hashKey(String originalKey) {
            try {
                MessageDigest messageDigest = MessageDigest.getInstance("MD5");
                messageDigest.update(originalKey.getBytes());
                String string = bytesToHexString(messageDigest.digest());
                return string;
            } catch (NoSuchAlgorithmException e) {
                e.printStackTrace();
            }
            return null;
        }
    
        private static String bytesToHexString(byte[] digest) {
            StringBuilder stringBuilder = new StringBuilder();
            for (byte b : digest) {
                String hex = Integer.toHexString(b);
                if (hex.length() == 1) {
                    stringBuilder.append('0');
                }
                stringBuilder.append(hex);
            }
            return stringBuilder.toString();
        }
```

通过一个key可以获取到一个Editor方法，然后通过Editor可以获取一个文件输出流，需要注意的是，由于前面在DiskLruCache的open方法中设置了一个节点只有一个数据，因此下面的DISK_KEY_INDEX值为0，然后通过这个输出流来写入一个文件，完成之后，还需要通过editor的commit方法来提交写入操作，如果写入图片失败，则使用abotr方法来回退整个操作，最后调用DiskLruCache的flush方法进行刷新。

```java
    //添加一个缓存
      @Override
        public void put(String key, Bitmap bitmap) throws IOException {
            String hexKey = HashKey.hashKey(key);
            DiskLruCache.Editor edit = mDiskLruCache.edit(hexKey);
            if (edit != null) {
                OutputStream outputStream = null;
                boolean compressed = false;
                try {
                    outputStream = edit.newOutputStream(DISK_KEY_INDEX);
                    compressed = bitmap.compress(Bitmap.CompressFormat.PNG, 90, outputStream);
                } finally {
                    if (outputStream != null) {
                        outputStream.close();
                    }
                    Log.d(TAG, "compressed:" + compressed);
                    if (compressed) {
                        edit.commit();
                    } else {
                        edit.abort();
                    }
                    mDiskLruCache.flush();
                }
            }
        }
```

#### DiskLruCache的缓存查找

和缓存的添加过程类似，缓存查找过程通过key得到一个缓存快照(snapshot),然后通过Snapshot对象即得到缓存的文件输入流，有了文件输入流，自然可以得到Bitmap对象了。
需要注意的是，我们在加载缓存的时候，需要对Bitmap进行缩放，必然会使用FileInputStream两次，一次获取图片的宽高信息，一次真正去加载图片，但是这存在一个问题，原因是一个有序的文件流，而两次decodeStream调用影响了文件流的位置属性，导致第二次decodeStream是获取的null。为了解决这个问题，通过文件流来得到它对应的文件描述符，然后通过BitmapFactory.decodeFileDescriptor方法来加载一种缩放的图片，这个过程的实现如下：

```java
      public Bitmap get(String key) throws IOException {
            String hexKey = HashKey.hashKey(key);
            DiskLruCache.Snapshot snapshot = mDiskLruCache.get(hexKey);
            if (snapshot != null) {
                FileInputStream inputStream = null;
                try {
                    inputStream = (FileInputStream) snapshot.getInputStream(DISK_KEY_INDEX);
                    FileDescriptor fd = inputStream.getFD();
                    Bitmap bitmap = BitmapFactory.decodeFileDescriptor(fd, null, new BitmapFactory.Options());
                    Log.d(TAG, "bitmap:" + bitmap);
                    return bitmap;
    
                } finally {
    
                    if (inputStream != null) {
                        inputStream.close();
                    }
                }
            }
            Log.d(TAG, "snapshot null:" );
            return null;
        }
```

#### 关于DiskLruCache的其他API

缓存的移除：移除缓存主要是借助DiskLruCache的remove()方法实现的，接口如下所示：
```
    public synchronized boolean remove(String key) throws IOException
```

用法虽然简单，这个方法我们并不应该经常去调用它。因为完全不需要担心缓存的数据过多从而占用SD卡太多空间的问题，DiskLruCache会根据我们在调用open()方法时设定的缓存最大值来自动删除多余的缓存。只有你确定某个key对应的缓存内容已经过期，需要从网络获取最新数据的时候才应该调用remove()方法来移除缓存。

其他API：

- size()
这个方法会返回当前缓存路径下所有缓存数据的总字节数，以byte为单位，如果应用程序中需要在界面上显示当前缓存数据的总大小，就可以通过调用这个方法计算出来。

- flush()
这个方法用于将内存中的操作记录同步到日志文件（也就是journal文件）当中。这个方法非常重要，因为DiskLruCache能够正常工作的前提就是要依赖于journal文件中的内容。前面在讲解写入缓存操作的时候我有调用过一次这个方法，但其实并不是每次写入缓存都要调用一次flush()方法的，频繁地调用并不会带来任何好处，只会额外增加同步journal文件的时间。比较标准的做法就是在Activity的onPause()方法中去调用一次flush()方法就可以了。

- close()
这个方法用于将DiskLruCache关闭掉，是和open()方法对应的一个方法。关闭掉了之后就不能再调用DiskLruCache中任何操作缓存数据的方法，通常只应该在Activity的onDestroy()方法中去调用close()方法。

- delete()
这个方法用于将所有的缓存数据全部删除，比如说网易新闻中的那个手动清理缓存功能，其实只需要调用一下DiskLruCache的delete()方法就可以实现了。
