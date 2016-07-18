# 自定义SurfaceView（一）：SurfaceHolder

---

write in *2016-07-18*

#### 概述

本篇将分享如何自定义一个SurfaceView并作为一个自定义控件在layout文件中使用。同时展示一些我在工程中做好的自定义SurfaceView并简单说明它的特性。由于没有了解过SurfaceView的相关实现原理等，所以这里我尽量避开对其原理相关描述，若仍有不当之处，欢迎指正。

由于内容较多，我内容将分成由简到繁多篇，这是第一篇，主要是关于SurfaceHolder的内容。

## SurfaceView简介

---

SurfaceView继承自View但是与其他继承自View的控件有一个很大的不同——SurfaceView允许你使用一个其他线程（非主线程）来执行绘制等操作，因此它不会阻塞主线程，可以用来做一些简单的2D游戏（但它的绘制效率不高，因此做太复杂的绘制操作会导致闪屏等问题）。 

### SurfaceHolder简介

---

SurfaceHolder根据表面意思理解就认为是Surface的夹具，surface的概念我并不清楚所以就不谈了（下文中的surface就按照“显示的视图”来理解），我只在这说明这个“夹具”在surfaceView中扮演了一个重要的角色。它提供了几个回调接口可以让你来判断surface的状态并进行相应的处理。同时，它还是跟surface交互的重要“工具，我们要用它来获取用来绘制的Canvas（画布）。

一般来说，一个自定义SurfaceView需要在创建的时候取得对应的SurfaceHolder（`this.getHolder（）`），并添加一个回调来正常的控制SurfaceView。

####  添加SurfaceHolder的回调（Callback）

``` Java
        new SurfaceHolder.Callback() {
            @Override
            public void surfaceCreated(SurfaceHolder holder) {
                
            }

            @Override
            public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {

            }

            @Override
            public void surfaceDestroyed(SurfaceHolder holder) {
                
            }
        }
```

这是一个SurfaceHolder.Callback接口类的实例，可以看到它包含了3个接口函数：surfaceCreated（surface被创建）、surfaceChanged（surface发生改变，如大小改变）和surfaceDestroyed（surface被销毁）。当这些事件发生时，对应的接口函数就会被调用。

一般来说我们会在surfaceCreated里启动绘制线程，在surfaceDestroyed里中断线程，而surfaceChanged中我们可能会根据新的surface参数来调整我们的绘制参数。

写好接口类的实例就要给SurfaceHolder的实例添加这个接口类实例了，这要用到SurfaceHolder实例的addCallback(Callback)方法,如下：

``` Java
SurfaceHolder holder = this.getHolder();
holder.addCallback(new SurfaceHolder.Callback(){
  ...
});
```

#### SurfaceHolder的绘图相关操作

SurfaceHolder实例的另外两个重要函数分别是lockCanvas（锁定画布）和unlockCanvasAndPost(解锁画布并提交），调用时机分别是开始绘制前和绘制完成后。下面是一段示例代码：

``` Java
SurfaceHolder holder = this.getHolder();
Canvas canvas = holder.lockCanvas();
if(canvas != null) {
    //这里进行绘制操作
    ...
    holder.unlockCanvasAndPost(canvas);
}
```

上段代码是SurfaceView中最基本的绘制流程，仅用这段代码（补全绘图操作）就可以实现简单的SurfaceView的绘制操作了。当然实际的绘图操作可能会很复杂，容易出错，所以一般会把绘制操作放到`try...catch`的代码块里。

简单说明一下，我们先用lockCanvas锁定（取得）了一个Canvas（画布），如果锁定成功（`canvas != null`），我们就在画布上绘制出我们需要的图像，**这时图像只是在画布上并不能显示到屏幕上**，因此我们要使用unlockCanvasAndPost解锁画布并把绘制的内容提交给surface，具体绘制到屏幕上的实现就不用我们关心了。

# END