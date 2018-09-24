# 单一职责原则

单一职责原则(Signle Responsibility Principle)：**只有一种原因能引起类的变化**，单一职责用 **职责** 和 **变化原因** 来衡量接口或者类设计的是否优良，但是 **职责** 和 **变化原因** 都不可度量，因项目和环境而异。所以：**对于接口设计师尽量做到职责单一，对于类尽量做到只有一个原因引起变化**。

## 单一职责的好处

*   类的复杂性降低，实现什么职责都非常清晰
*   复杂性降低，可读性自然就提高了
*   变更引起的风险降低，变更是必不可少的，如果接口的单一职责做的好，一个接口的变更只影响其单一的实现类，对其他接口无影响，这对系统的扩展性和可维护性都有很多的帮助

## 实例 1

设计一个电话接口用于拨打电话

```
    public interface Iphone{
        /**
         * 拨打电话
         */
        void dial(String  phoneNumber);
    
        /**
         * 聊天
         */
        void chat(Object chatObj);
    
        /**
         * 挂电话
         */
        void hangup();
    }
```    

这个电话现在有协议管理（dial，hangup）和数据传输（chat）的功能，协议接通的变化会引起实现类的变化，而数据传输的变化也会引起类的变化，所以这里出现了两个原因可能引起了类的变化，电话拨号只要能拨通就行，并不用关心电话接通之后传输的什么数据，挂机也是一样，所以这里有两个相互不影响的职责，那就可以考虑分成两个接口：

```
/**
协议连接管理器
 */
public interface IConnectionManager {

    void dial(String phoneNumber);

    void hangup();

}

/**
 * 数据传输管理器
 */
public interface IDataTransfer {

    void chat(Object object);
}


public class IPhone {

    private IConnectionManager iConnectionManager;
    private IDataTransfer iDataTransfer;

    public IPhone_G(IConnectionManager connectionManager , IDataTransfer dataTransfer) {
        iConnectionManager = connectionManager ;
        iDataTransfer = dataTransfer;
    }

    public void dial(String phoneNumber) {
        iConnectionManager.dial(phoneNumber);
    }

    public void hangup() {
        iConnectionManager.hangup();
    }

    public void chat(Object object) {
        iDataTransfer.chat(objects);
    }
}
```


java是面向接口编程的，**对外只需要公布接口，而非实现类**，这里采用两个接口分别定义了连接协议与数据传输两个职责，而IPhone与两个接口采用**组合模式**，定义连接协议与数据传输的方法，而真正的实现是两个接口的实现类，从而简化的IPhone的复杂性，类的职责简单清晰，让变更引起的风险降低，一个接口修改只对相应的实现类有影响。



## 实例 2

ImageLoader伪代码：

```
    public class ImageLoader {
    
        private LruCache<String, Bitmap> mLruCache;
        private Executor mExecutor = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());
    
        public ImageLoader() {
            init();
        }
    
        private void init() {
            mLruCache = new LruCache<String, Bitmap>((int) (Runtime.getRuntime().maxMemory() / 1024 / 4)) {
                @Override
                protected int sizeOf(String key, Bitmap value) {
                    return value.getRowBytes() * value.getHeight() / 1024;
                }
            };
        }

        public void displayImage(final ImageView imageView, final String url) {
            Bitmap bitmap = mLruCache.get(url);
            if (bitmap != null) {
                imageView.setImageBitmap(bitmap);
            } else {
                imageView.setTag(url);
                mExecutor.execute(new Runnable() {
                    @Override
                    public void run() {
                        Bitmap bitmap = downloadImage(url);
                        mLruCache.put(url, bitmap);
                        if (url.equals(imageView.getTag())) {
                            imageView.setImageBitmap(bitmap);
                        }
                    }
                });
            }
        }

        private Bitmap downloadImage(String url) {
            return null;
        }
    }
```

从代码可以看出，此ImageLoader没有遵守单一职责，内存缓存、下载图片、加载逻辑组织都放在了一个类里面，将要如果需要修改缓存或者下载逻辑，都需要修改ImageLoader，而且所有的代码放在一个类里面，可读性也不高，所以做如下重构：

1. 抽出缓存功能,由单独的类复杂图片缓存的实现
2. 抽出下载图片的功能个，由单独的类复杂图片下载的实现

重构代码：根据职责划分抽离出接口：

```
    public interface ImageCache {
        void cacheImage(String url, Bitmap bitmap);
    
        Bitmap getCache(String key);
    }
    
    public interface ImageDownloader {
    
        Bitmap downloadImage(String url);
    
    }


    public class SRPImageLoader {
    
        private ImageCache mImageCache;
    
        private ImageDownloader mImageDownloader;
    
        private Executor mExecutor = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());
    
        public SRPImageLoader() {
            init();
        }
    
        private void init() {
            mImageCache = new ImageCacheImpl();
            mImageDownloader = new ImageDownloaderImpl();
        }
    
    
        public void displayImage(final ImageView imageView, final String url) {
            Bitmap bitmap = mImageCache.getCache(url);
            if (bitmap != null) {
                imageView.setImageBitmap(bitmap);
            } else {
                imageView.setTag(url);
                mExecutor.execute(new Runnable() {
                    @Override
                    public void run() {
                        Bitmap bitmap = downloadImage(url);
                        mImageCache.cacheImage(url, bitmap);
                        if (url.equals(imageView.getTag())) {
                            imageView.setImageBitmap(bitmap);
                        }
                    }
                });
            }
        }
    
    
        private Bitmap downloadImage(String url) {
            return mImageDownloader.downloadImage(url);
        }
    
    
    }
```