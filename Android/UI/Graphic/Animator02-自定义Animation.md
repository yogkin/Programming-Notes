# 自定义动画

自定义动画用的不多，同时也是比较复杂的事情，因为涉及到一些matrix的变换，而且还需要用到camera类，暂时不做过多介绍：
一般实现自定义动画需要实现Animation的两个方法：

- initialize 一般在此方法中做一些初始化工作。(int width, int height, int parentWidth, int parentHeight)分别表示 当前view和父view的宽高
- applyTransformation(float interpolatedTime, Transformation t) 在applyTransformation实现对matix的变换，而达到动画效果：
- `interpolatedTime`表示一个梯度值
- Transformation中获取matrix

下面是 apidemo 中的一个 3d 旋转动画的 demo

## ApiDemo-Rotate3D

```java
    public class Custome3DAnimation extends Animation {
    
        private static final String TAG = Custome3DAnimation.class.getSimpleName();
        private final float mFromDegress;//开始的角度
        private final float mToDegress;//到底的角度
        private final float mCenterX;//x轴中心
        private final float mCenterY;//y轴中心
        private final float mDepthZ;//
        private final boolean mReverse;
        private Camera mCamera;
    
        public Custome3DAnimation(float fromDegress, float toDegress, float centerX, float centerY, float depthZ, boolean reverse) {
            mFromDegress = fromDegress;
            mToDegress = toDegress;
            mCenterX = centerX;
            mCenterY = centerY;
            mDepthZ = depthZ;
            mReverse = reverse;
        }

        /**
         * 用于做一些初始化工作
         *
         * @param width
         * @param height
         * @param parentWidth
         * @param parentHeight
         */
        @Override
        public void initialize(int width, int height, int parentWidth, int parentHeight) {
            super.initialize(width, height, parentWidth, parentHeight);
            mCamera = new Camera();
        }

        /**
         * 在此方法中，做一些矩阵变换操作，从而达到动画的效果
         *
         * @param interpolatedTime
         * @param t
         */
        @Override
        protected void applyTransformation(float interpolatedTime, Transformation t) {
            super.applyTransformation(interpolatedTime, t);

            final float fromDegress = mFromDegress;
            float degress = fromDegress + ((mToDegress - fromDegress) * interpolatedTime);
    
            final float centerX = mCenterX;
            final float centerY = mCenterY;
            final Camera camera = mCamera;
    
            final Matrix matrix = t.getMatrix();
    
            camera.save();
    
            if (mReverse) {
                camera.translate(0f, 0f, mDepthZ * interpolatedTime);
            } else {
                camera.translate(0f, 0f, mDepthZ * (1f - interpolatedTime));
            }
            camera.rotateY(degress);
            camera.getMatrix(matrix);
            camera.restore();
    
            matrix.preTranslate(-centerX, -centerY);
            matrix.postTranslate(centerX, centerY);
        }
    }
```











