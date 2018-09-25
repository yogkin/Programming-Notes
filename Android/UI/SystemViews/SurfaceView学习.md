# SucraceView 学习

## 1 SucraceView介绍

SucraceView提供了一个子线程用于绘图，可以直接从内存或者DMA等硬件接口取得图像数据，所以从性能上来讲是比较高效。
它的特性是：可以在主线程之外的线程中向屏幕绘图上。这样可以避免画图任务繁重的时候造成主线程阻塞，从而提高了程序的反应速度。在游戏开发中多用到SurfaceView，游戏中的背景、人物、动画等等尽量在画布canvas中画出。

## 2 实现方法

### 实现步骤

    1．继承SurfaceView
    2．实现SurfaceHolder.Callback接口


### 需要重写的方法

```java
public void surfaceChanged(SurfaceHolder holder,int format,int width,int height){}　　//在surface的大小发生改变时激发

public void surfaceCreated(SurfaceHolder holder){}　　//在创建时激发，一般在这里调用画图的线程。

public void surfaceDestroyed(SurfaceHolder holder) {}　　//销毁时激发，一般在这里将画图的线程停止、释放。SurfaceHolder
```

SurfaceHolder,surface的控制器，用来操纵surface。处理它的Canvas上画的效果和动画，控制表面，大小，像素等。几个需要注意的方法：

```
(1)、abstract void addCallback(SurfaceHolder.Callback callback);
// 给SurfaceView当前的持有者一个回调对象。
(2)、abstract Canvas lockCanvas();
// 锁定画布，一般在锁定后就可以通过其返回的画布对象Canvas，在其上面画图等操作了。
(3)、abstract Canvas lockCanvas(Rect dirty);
// 锁定画布的某个区域进行画图等..因为画完图后，会调用下面的unlockCanvasAndPost来改变显示内容。
// 相对部分内存要求比较高的游戏来说，可以不用重画dirty外的其它区域的像素，可以提高速度。
(4)、abstract void unlockCanvasAndPost(Canvas canvas);
// 结束锁定画图，并提交改变。
```

### 整个过程总结

  - 继承SurfaceView并实现SurfaceHolder.Callback接口 
  - SurfaceView.getHolder()获得SurfaceHolder对象 
  - SurfaceHolder.addCallback(callback)添加回调函数
  - SurfaceHolder.lockCanvas()获得Canvas对象并锁定画布 
  - Canvas绘画
  - SurfaceHolder.unlockCanvasAndPost(Canvas canvas)结束锁定画图，并提交改变，将图形显示。



### 模版写法

```java
public class LoadingView extends SurfaceView implements Callback, Runnable {
    private SurfaceHolder holder;
    private boolean isRunning;
    private Canvas mCanvas;
    public LoadingView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        init();
    }
    public LoadingView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }
    public LoadingView(Context context) {
        this(context, null);
    }
    public void init() {
        holder = getHolder();
        holder.addCallback(this);
    }
    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        isRunning = true;
        Thread thread = new Thread(this);
        thread.start();
    }
    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width,
            int height) {
    }
    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
    }
    @Override
    public void run() {
        while (isRunning) {
            doDraw();
        }
    }
    private void doDraw() {
        try {
            mCanvas = holder.lockCanvas();
            if (mCanvas != null) {
                draw();
            }
        } catch (Exception e) {
        } finally {
            if (mCanvas != null) {
                holder.unlockCanvasAndPost(mCanvas);
            }
        }
    }
    private void draw() {
    }
}
```


---
## 3 示例

```java
public class LuckyPan extends SurfaceView implements Callback, Runnable {
    // 画布
    private Canvas mCanvas;
    private SurfaceHolder mHolder;
    private boolean mIsRunning;// 线程标记
    private int mRadius;// 园的半径
    private int mPadding;
    private int testSize;// 文字尺寸
    private int mCenter;// 中心点
    private String[] textArr;// 绘画的文字
    private int count;// 盘块个数
    private float startAngle;// 开始的角度
    private Paint mTextPaint;// 文字画笔
    private Paint arcPaint;// 圆弧画笔
    private int[] pictureIds;// 需要的图片的id;
    private int[] colors;// 盘块颜色
    private RectF mArcRangeRect;// 范围
    private boolean isShouldEnd;// 是否点击了停止
    private Bitmap[] mBitmapArr;// 需要绘画的图片
    private Bitmap mBgBitmap;
    private int mSpeed;// 速度

    public LuckyPan(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        init();
    }

    public LuckyPan(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public LuckyPan(Context context) {
        this(context, null);
    }

    private void init() {
        count = 6;
        testSize = (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_SP,
                20, getResources().getDisplayMetrics());
        textArr = new String[] { "恭喜发财", "iPad", "单反相机", "服装一套", "IPhone",
                "恭喜发财" };
        pictureIds = new int[] { R.drawable.f040, R.drawable.ipad,
                R.drawable.danfan, R.drawable.meizi, R.drawable.iphone,
                R.drawable.f040 };
        colors = new int[] { 0xff12cc43, 0xffddcc12, 0xffffc3d5, 0xffddcc12,
                0xffdcbcba, 0xffc38011 };
        // 获取生命周期holder 添加回调
        mHolder = getHolder();
        mHolder.addCallback(this);
        // 设置可以获取到焦点
        setFocusable(true);
        setFocusableInTouchMode(true);
        // 设置常亮
        setKeepScreenOn(true);
    }

    // 生命周期回调-----------------------------------------
    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        mTextPaint = new TextPaint(Paint.ANTI_ALIAS_FLAG | Paint.DITHER_FLAG);
        mTextPaint.setColor(Color.WHITE);
        mTextPaint.setTextSize(testSize);
        arcPaint = new Paint(Paint.ANTI_ALIAS_FLAG | Paint.DITHER_FLAG);
        mArcRangeRect = new RectF(getPaddingLeft(), getPaddingLeft(),
                getPaddingLeft() + mRadius, getPaddingLeft() + mRadius);
        mBitmapArr = new Bitmap[count];
        mSpeed = 20;
        mBgBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.bg);
        for (int i = 0; i < count; i++) {
            mBitmapArr[i] = BitmapFactory.decodeResource(getResources(),
                    pictureIds[i]);
        }
        startAngle = 0;
        mIsRunning = true;
        Thread thread = new Thread(this);
        thread.start();
    }
    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width,
            int height) {
    }
    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        mIsRunning = false;
    }


    // 生命周期回调-----------------------------------------
    /**
     * 子线程内绘画
     */
    @Override
    public void run() {
        while (mIsRunning) {
            long timeStart = System.currentTimeMillis();
            draw();
            long timeEnd = System.currentTimeMillis();
            if (timeEnd - timeStart < 50) {
                try {
                    Thread.sleep(50 - (timeEnd - timeStart));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    private void draw() {
        try {
            // 因为受SurfaceView生命周期影响 没Canvas可能会为null
            mCanvas = mHolder.lockCanvas();
            if (mCanvas != null) {
                // 画背景
                drawbg();
                // 话其他的元素
                float tmpAngle = startAngle;
                float sweepAngle = (float) (360 / count);
                for (int i = 0; i < count; i++) {
                    // 画园
                    arcPaint.setColor(colors[i]);
                    mCanvas.drawArc(mArcRangeRect, tmpAngle, sweepAngle, true,
                            arcPaint);
                    // 画文字
                    drawText(i, tmpAngle, sweepAngle);
                    drawIcon(i, tmpAngle);
                    tmpAngle += sweepAngle;
                }
                startAngle += mSpeed;
            }
        } catch (Exception e) {
        } finally {
            if (mCanvas != null) {
                mHolder.unlockCanvasAndPost(mCanvas);
            }
        }
    }
    
    /**
     * 画图片
     */
    private void drawIcon(int i, float startAngle) {
        // 设置图片的宽度为直径的1/8
        int imgWidth = mRadius / 8;
        float angle = (float) ((30 + startAngle) * (Math.PI / 180));
        int x = (int) (mCenter + mRadius / 2 / 2 * Math.cos(angle));
        int y = (int) (mCenter + mRadius / 2 / 2 * Math.sin(angle));
        // 确定绘制图片的位置
        Rect rect = new Rect(x - imgWidth / 2, y - imgWidth / 2, x + imgWidth
                / 2, y + imgWidth / 2);
        mCanvas.drawBitmap(mBitmapArr[i], null, rect, null);
    }

    private void drawText(int i, float tmpAngle, float sweepAngle) {
        Path path = new Path();
        // 圆圈的周长/count /2 - textWidth/2
        float xOffset = (float) (mRadius * Math.PI) / 6 / 2
                - mTextPaint.measureText(textArr[i]) / 2;
        path.addArc(mArcRangeRect, tmpAngle, sweepAngle);
        mCanvas.drawTextOnPath(textArr[i], path, xOffset, mRadius / 2 / 6,
                mTextPaint);
    }

    private void drawbg() {
        mCanvas.drawColor(Color.WHITE);
        mCanvas.drawBitmap(mBgBitmap, null, new Rect(mPadding / 2,
                mPadding / 2, getMeasuredWidth() - mPadding / 2,
                getMeasuredWidth() - mPadding / 2), null);
    }

    /**
     * 强制设置为正方形
     */
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, widthMeasureSpec);
        int width = Math.min(getMeasuredWidth(), getMeasuredHeight());
        mRadius = width - getPaddingLeft() * 2;
        mCenter = width / 2;
        mPadding = getPaddingLeft();
        setMeasuredDimension(width, width);
    }
}
```

 
