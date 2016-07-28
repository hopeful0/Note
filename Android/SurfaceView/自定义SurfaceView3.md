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

但如果需要在layout文件里使用自定义SurfaceView就需要额外的构造函数。