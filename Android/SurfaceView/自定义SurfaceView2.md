# 自定义SurfaceView（二）：线程

---

write in *2016-07-21*

#### 概述

这是自定义SurfaceView的第二篇，虽然SurfaceView允许使用非UI线程进行绘制，但其本身并没有给我们提供一个用来绘图的线程，我们仍需要自己新建一个线程（`Thread`），而且这个线程不仅可以用来绘图。因此，本篇将以SurfaceView中的线程为中心，介绍其创建使用直至销毁的整个过程。

## 线程的创建（启动）

---

一般来说线程的创建是依赖于surface的状态的，因此应该把创建或者启动线程放到SurfaceHolder的回调方法 `surfaceCreated`中。

``` Java
    ...
    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        mThread = new Thread(new Runnable() {
            @Override
            public void run() {
                while(true) {
                    try{
                        Thread.sleep(50);
                     } catch (InterruptExpection e) {
                     }
                     //do something
                     ...
                }
            }
        });
        mThread.start();
    }
    ...
```

但是有时候由于需要，你可能在自定义SurfaceView被构造的时候就创建并启动这个线程（比如用SurfaceView线程做游戏主线程，需要进行初始化处理），这样你需要加上一个标志（flag），并在回调中改变这个标志的状态，确保在surface被创建后才会进行SurfaceView的绘制。

## 线程的使用——Runnable

---

线程的使用即你要利用线程做些什么，大多数情况下，SurfaceView的线程不会仅仅用来绘制视图，还会有一些逻辑运算等事件要处理。一般的，我们在Runnable的`run`方法里写一个`while`死循环，并在循环里调用`mLogic()`和`mDraw()`两个私有方法，分别实现这两个方法用来处理非绘图的逻辑事件与绘图事件。

如果你需要用线程做更多工作,务必安排好各个工作的顺序，一般来说绘图操作放到一次操作的最后进行，这样可以保证显示的是当前的最新数据。

如上一节中的代码，我习惯先使线程休眠，当然也可以先处理工作中再休眠线程。`sleep(long t)`的参数是休眠时间，单位是毫秒，使用1000/t就是SurfaceView的刷新频率。

注意不要在线程里调用一些强制在主线程中的函数，如果需要建议使用Handle来处理。

## 线程的销毁（中断）

---

线程的销毁在这里就是中断线程并置空，具体的销毁过程就交给Java来处理了。与线程的启动相对应，中断应该在回调方法`surfaceDestoryed`里。

``` Java
    ...
    @Override
    public void surfaceDestoryed(SurfaceHolder holder) {
        mThread.interrupt();
        mThread = null;
    }
    ...
```

如果你需要在一个不确定的时间中断线程，同启动时类似，你需要设置标记并且在surface被销毁的时候改变标记状态，确保线程不再进行绘制操作。

# END