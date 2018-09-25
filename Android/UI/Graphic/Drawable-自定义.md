# 自定义drawable

自定义Drawable，相比View来说，Drawable属于轻量级的、使用也很简单。以后自定义实现一个效果的时候，可以改变View first的思想，尝试下Drawable first。

下面提供两个自定义drawable的案例：


```java
    public class CircleBackgroundDrawable extends Drawable {
    
        private Paint mPaint;
    
        public CircleBackgroundDrawable(int color) {
            mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
            mPaint.setColor(color);
    
        }
    
        @Override
        public void draw(Canvas canvas) {
            Rect bounds = getBounds();
            float centerX = bounds.exactCenterX();
            float centerY = bounds.exactCenterY();
            canvas.drawCircle(centerX, centerY, Math.min(centerX, centerY), mPaint);
        }
    
        @Override
        public void setAlpha(int alpha) {
            mPaint.setAlpha(alpha);
            invalidateSelf();
        }
    
        @Override
        public void setColorFilter(ColorFilter colorFilter) {
            mPaint.setColorFilter(colorFilter);
            invalidateSelf();
        }
    
        @Override
        public int getOpacity() {
            return PixelFormat.TRANSLUCENT;
        }
    
    
        @Override
        public int getIntrinsicHeight() {
            return 100;
        }
    
    
        @Override
        public int getIntrinsicWidth() {
            return 100;
        }
    }

    public class CircleDrawable extends Drawable {
    
        private Bitmap mBitmap;
        private Paint mPaint;
        private RectF rectF;
    
        public CircleDrawable(Bitmap bitmap) {
            mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
            BitmapShader bs = new BitmapShader(bitmap, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);
            mPaint.setShader(bs);
        }
    
        @Override
        public void setBounds(int left, int top, int right, int bottom) {
            super.setBounds(left, top, right, bottom);
            rectF = new RectF(left, top, right, bottom);
        }
    
        @Override
        public void draw(Canvas canvas) {
    
            canvas.drawRoundRect(rectF, 20,20,mPaint);
        }
    
        @Override
        public void setAlpha(int alpha) {
            mPaint.setAlpha(alpha);
            invalidateSelf();
        }
    
        @Override
        public void setColorFilter(ColorFilter colorFilter) {
            mPaint.setColorFilter(colorFilter);
            invalidateSelf();
        }
    
        @Override
        public int getOpacity() {
            return PixelFormat.TRANSLUCENT;
        }
    
        @Override
        public int getIntrinsicWidth() {
            if (mBitmap != null) {
                return mBitmap.getWidth();
            }
            return super.getIntrinsicWidth();
        }
    
        @Override
        public int getIntrinsicHeight() {
            if (mBitmap != null) {
                return mBitmap.getHeight();
            }
            return super.getIntrinsicHeight();
        }
    }
```

各个方法说明：

- `setColorFilter` 调用自己的paint设置后，调用invalidateSelf即可
- `setAlpha` 同上
- `getOpacity`
- `getIntrinsicHeight与getIntrinsicHeight`，默认的drawable并没有宽高都，这两个默认返回-1，如果要处理view的wrap_content，需要返回一个有效值。
- setBounds与getBounds，getBounds返回的是自身的区域大小，与view的大小有关
- draw 用于绘制自身
