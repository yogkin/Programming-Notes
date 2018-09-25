## 1 模拟一个正弦波

默认情况下，正选的周长为2π，要在长度width下实现一个完整的正弦波：

```java
            final int width = 400;
            double sin;
            for (int i = 0; i < width; i++) {
                 sin = Math.sin(  （i * 1.0F / width） * 2 * Math.PI);
                System.out.println(i + "   sin = "+sin);
            }
```

首先 `i * 1.0F / width` 的范围是`0-1`，所以完整的范围就是`[0-2π]`


## 2 根据角度计算椭圆上的坐标

```java
        private float mA; 椭圆的长轴
        private float mB; 椭圆的短轴

        private float mCenterX;  椭圆中心x坐标
        private float mCenterY;  椭圆中心y坐标

        float newX = (float) (Math.cos(Math.toRadians(angle)) * mA + mCenterX);
        float newY = (float) (Math.sin(Math.toRadians(angle)) * mB + mCenterY);
```

## 3 Matrix主要使用的地方

- canvas
- canvas画bitmap
- shader
- path.transform

## 4 0-1的梯度如果转换为两个0-0.5的梯度

```java
     //假设mValue的变化梯度为0-1,利用取绝对值的方法可以转换为两个梯度
    ((0.5 - Math.abs(mValue - 0.5))
```