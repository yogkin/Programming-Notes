# 开闭原则

开闭原则(Open-Close Principle)：**一个软件实体，如类，模块和函数应该对扩展开发，对修改关闭，意思就是一个软件实体应该通过扩展来来实现变化，而不是通过修改已有代码。**

开闭原则指导我们如何创建一个稳定的，灵活的系统。在软件的生命周期内，因为变化，升级和维护等原因需要对原有代码进行修改时，可**能将错误引入原本已经经过测试的代码中，破坏原有的系统。**因此在应对软件需求变化时，应该尽量通过扩展的方式来实现变化，而不是通过修改现有的稳定的代码。但是开闭原则对修改关闭，对扩展开发，并不意味着不做任何修改，底层模块的变更，必然要有高层模块的耦合，否则就是一个孤立无意义的代码块

**开闭原则的重要性：可以说开闭原则是其他原则的抽象，是面向对象原则的基础**

*   开闭原则对测试的影响，修改不打破原有的测试效果
*   开闭原则可提高复用性
*   开闭原则可提高可维护性
*   面向对象开发的要求

**如何使用开闭原则**

1. 抽象约束，通过接口可以约束潜在的变化
 1. 通过接口扩展抽象类约束扩展，对扩展实现边界限定，不运行出现在接口和抽象类中不存在的 public方法
 2. 参数类型，对象引用尽量使用接口或者抽象类
 3. 抽象层尽量保持稳定，一旦确定，不要修改
2.  用元数据控制程序行为，元数据比如就是xml中定义类的全路径

## 实例 

ImageLoader的缓存实现，前面的ImageLoader通过重构，实现了单一职责，但是现在需求变更，缓存需要加入本地缓存，那么如何实现呢？

```
     public class ImageLoader {
    
        private ImageCache mMemoryCache = new MemoryCache();//内存缓存
        private ImageCache mDiskCache = new DiskCache();//磁盘缓存
        private ImageCache mDoubleCache = new DoubleCache();//双缓存

        private boolean mUseMemoryCache;
        private boolean mUseDiskCache;
    
        private ImageDownloader mImageDownloader;
    
        private Executor mExecutor = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());
    
    
        public void useMemoryCache(boolean useMemoryCache) {
            mUseMemoryCache = useMemoryCache;
        }
    
        public void useDiskCache(boolean useDiskCache) {
            mUseDiskCache = useDiskCache;
        }
    
    
        public void displayImage(final ImageView imageView, final String url) {
    
            Bitmap bitmap = null;
            if (mUseMemoryCache && !mUseDiskCache) {
                bitmap = mMemoryCache.getCache(url);
            }else  if (!mUseMemoryCache && mUseDiskCache) {
                bitmap = mDiskCache.getCache(url);
            }else(mUseMemoryCache && mUseDiskCache){
                bitmap = mDoubleCache.getCache(url);
            }
    
    
            if (bitmap != null) {
                imageView.setImageBitmap(bitmap);
            } else {
                imageView.setTag(url);
                mExecutor.execute(new Runnable() {
                    @Override
                    public void run() {
                        Bitmap bitmap = downloadImage(url);

                         if (mUseMemoryCache && !mUseDiskCache) {
                            mMemoryCache.cacheImage(url, bitmap);
                         }else  if (!mUseMemoryCache && mUseDiskCache) {
                            mDiskCache.cacheImage(url, bitmap);
                         }else(mUseMemoryCache && mUseDiskCache){
                             bitmap = mDoubleCache.getCache(url);
                         }

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

现在ImageLoader实现了磁盘缓存功能，但是可以看出有各种问题：

1. 代码中出现多很多的if,else判断，随着以后的功能迭代，各种if，else判断充斥在一起，引发bug的几率将会大大增加
2. ImageLoader中定义了多个ImageCache,而用户设置只会使用其中一种缓存，那么这也是一种浪费。而且很不优雅。
3. ImageLoader中ImageCache的实现被写死，用户无法扩展自己的ImageCache，没有扩展性
4. 随着缓存的需求更改，ImageLoader中的逻辑还是需要被修改，可能会越来越复杂

**所以把ImageLoader重构如下**：

```
    public class ImageLoader {
    
        private ImageCache mImageCache = new MemoryImageCache();//默认使用内存缓存
        private ImageDownloader mImageDownloader;
    
        private Executor mExecutor = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());
    
    
        /**
         * 用户可以注入各种ImageCache的实现
         * @param imageCache
         */
        public void setImageCache(ImageCache imageCache) {
            mImageCache = imageCache;
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
                        mImageCache.cacheImage(url,bitmap);
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
```

按照上面第一个版本的ImageLoader实现方式，每次加入新的缓存实现是都需要修改ImageLoader的代码，这就是没有扩展性的表现。

现在我们只在ImageLoader中定义一个ImageCache，默认实现为内存缓存，然后通过暴露setImageCache方法，ImageLoader的缓存实现具有了可扩展性，这样用户可以随意注入自己的缓存实现，而不需要修改原有的代码，各种if，else判断也被移除，代码又变的简洁了。

软件中的对象(类、模块、函数)应该对于扩展时开发的，但是对于修改是关闭的，而可扩展性是框架的最重要特性之一。

可以看出，**要实现开闭原则首先要需要有抽象**，使用抽象避免变更引起的风险，也为以后的需求更改提供了可扩展性，所以在对参数和对象引用时，尽量使用抽象类或者接口。而不关心具体的实现，从而 **让现有的系统拥有可扩展性，这样才能通过扩展来应对需求的变化。**