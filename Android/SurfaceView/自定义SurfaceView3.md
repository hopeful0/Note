# 自定义SurfaceView（三）：CustomeView

---

write in *2016-07-27*

#### 概述

这是自定义SurfaceView的第三节，本节主要介绍如何将自定义SurfaceView作为CustomView写在layout文件中。 

## 构造方法

---

一个简单的自定义SurfaceView只需要如下一个构造函数就够了。

``` Java
    public MySurfaceView (Context context) {
        super(context);
    }
```

但如果需要在layout文件里使用自定义SurfaceView就需要额外的构造函数了。这些有更多参数的构造方法能够使自定义SurfaceView类从layout文件中读到一些配置。

``` Java
    public MySurfaceView (Context context, AttributeSet attrs) {
        super(context,attrs);
    }

    public  MySurfaceView (Context context, AttributeSet attrs,int defStyleAttr) {
        super(context,attrs,defStyleAttr);
    }
```

这两个构造函数可以参照其父类SurfaceView或者View中的构造方法。实际上View还有另一个4参数的构造函数，但这里并不用引入这个构造函数就能正常使用。

## 重写`onMeasure`

---

