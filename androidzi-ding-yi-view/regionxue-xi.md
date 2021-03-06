# Region学习

Region代表了屏幕上的一个区域，它是有一个或多个Rect组成。

Op是Region类中的一个枚举，有6个值。

通过代码和效果测试一下：

```java
        Paint mp = new Paint();
        mp.setColor(Color.parseColor("#FF4081"));
        mp.setStyle(Paint.Style.FILL);

        RectF rectF1 = new RectF(100,100,300,300);
        RectF rectF2 = new RectF(200,200,400,400);

        canvas.drawColor(Color.BLUE);
        canvas.save();
        canvas.clipRect(rectF1);
        canvas.clipRect(rectF2, Region.Op.DIFFERENCE);

        canvas.restore();
        canvas.drawRect(rectF1,mp);
        canvas.drawRect(rectF2,mp);
```

Region.Op.DIFFERENCE 取第一次裁切的非交集部分

![](/assets/device-2017-08-04-170547.png)



Region.Op.INTERSECT 取两次裁切的交集部分

![](/assets/device-2017-08-04-170603.png)



Region.Op.REPLACE 第二次替代第一次裁切

![](/assets/device-2017-08-04-171128.png)

Region.Op.UNION  两次裁切的并集

![](/assets/device-2017-08-04-170858.png)

Region.Op.REVERSE\_DIFFERENCE 取的第二次裁切的非交集区域

![](/assets/device-2017-08-04-171252.png)



 

