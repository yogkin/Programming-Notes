# BitmapRegionDecoder

BitmapRegionDecoder可以根据一个Rect指定的区域生成表示原始图片此区域的Bitmap对象，使用方式如下：

```java
                open = getContext().getAssets().open("tupian.jpg");//获取图片的输入流
                BitmapFactory.Options options = new BitmapFactory.Options();
                options.inJustDecodeBounds = true;//只对图片进行采样
                BitmapFactory.decodeStream(open, null, options);
    
                //获取宽高
                int outWidth = options.outWidth;
                int outHeight = options.outHeight;
                //根据输入流创建一个BitmapRegionDecoder
                BitmapRegionDecoder brd = BitmapRegionDecoder.newInstance(open, false);
    
                BitmapFactory.Options optionsBrd = new BitmapFactory.Options();
                options.inPreferredConfig = Bitmap.Config.RGB_565;
                //生成原始图片的一部分
                Bitmap bitmap = brd.decodeRegion(new Rect(0, 0, outWidth / 2, outHeight / 2), optionsBrd);
```





有时候面对一张巨大的图片，在不缩放巨图的情况下，只能根据用户的手势拖动来显示巨图的某一部分。这时候BitmapRegionDecoder就可以派上用场。如下面清明上河图：

![](index_files/d.gif)

代码：

```java
public class LargeImageView extends View {

    private int mImageWidth, mImageHeight;
    private Rect mRect;
    private BitmapFactory.Options mOptions;
    private BitmapRegionDecoder mBitmapRegionDecoder;
    private float mLastX, mLastY;

    public LargeImageView(Context context) {
        this(context, null);
    }

    public LargeImageView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public LargeImageView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int measuredWidth = getMeasuredWidth();
        int measuredHeight = getMeasuredHeight();
        int left = mImageWidth / 2 - measuredWidth / 2;
        int top = mImageHeight / 2 - measuredHeight / 2;
        mRect.set(left, top, left + measuredWidth, top + measuredHeight);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        if (mBitmapRegionDecoder != null) {
            canvas.drawBitmap(mBitmapRegionDecoder.decodeRegion(mRect, mOptions), 0, 0, null);
        }
    }


    public void setImageStream(InputStream imageStream) {
        try {
            mBitmapRegionDecoder = BitmapRegionDecoder.newInstance(imageStream, false);
            BitmapFactory.Options tmpOptions = new BitmapFactory.Options();
            tmpOptions.inJustDecodeBounds = true;
            BitmapFactory.decodeStream(imageStream, null, tmpOptions);
            mImageWidth = tmpOptions.outWidth;
            mImageHeight = tmpOptions.outHeight;
            invalidate();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (imageStream != null) {
                try {
                    imageStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    private void init() {
        mRect = new Rect();
        mOptions = new BitmapFactory.Options();
        mOptions.inPreferredConfig = Bitmap.Config.RGB_565;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int action = event.getAction();
        float x = event.getX();
        float y = event.getY();
        switch (action) {
            case MotionEvent.ACTION_DOWN: {
                mLastX = x;
                mLastY = y;
                break;
            }
            case MotionEvent.ACTION_MOVE: {
                float dx = mLastX - x;
                float dy = mLastY - y;
                onMove(dx, dy);
                mLastX = x;
                mLastY = y;
                break;
            }
            case MotionEvent.ACTION_CANCEL:
            case MotionEvent.ACTION_UP: {
                break;
            }
        }
        return true;
    }

    private void onMove(float moveX, float moveY) {
        if (mImageWidth > getWidth()) {
            mRect.offset((int) moveX, 0);
            invalidate();
        }
        if (mImageHeight > getHeight()) {
            mRect.offset(0, (int) moveY);
            invalidate();
        }
    }
}
```

