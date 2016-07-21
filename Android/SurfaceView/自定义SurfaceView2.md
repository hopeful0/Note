# 自定义SurfaceView（二）：线程

---

write in *2016-07-21*

#### 概述

这是自定义SurfaceView的第二篇，虽然SurfaceView允许使用非UI线程进行绘制，但其本身并没有给我们提供一个用来绘图的线程，我们仍需要自己新建一个线程（`Thread`），而且这个线程不仅可以用来绘图。因此，本篇将以SurfaceView中的线程为中心，介绍其创建使用直至销毁的整个过程。

## 线程的创建

---

一般来说线程的创建是依赖于surface的状态的，因此应该把创建或者启动线程放到SurfaceHolder的回调方法 `surfaceCreated`中。

``` Java
    ...
    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        mThread = new Thread(new Runnable() {
            while(true) {
                //do something
                ...
            }
        }
    }
    ...
```


