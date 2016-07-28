# 自定义SurfaceView（三）：CustomeView

---

write in *2016-07-28*

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

虽然在引入新的构造方法后，我们已经能够在layout中写自定义SurfaceView了，但是这样的CustomView的尺寸是比较奇怪的，它一般会自动充满整个父视图。我们需要重写自定义SurfaceView的`onMeasure(int widthMeasureSpec, int heightMeasureSpec)`方法来自己规划自定义SurfaceView的尺寸。

``` Java
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        //super.onMeasure(widthMeasureSpec, heightMeasureSpec)
        int w = MeasureSpec.getSize(widthMeasureSpec),h = MeasureSpec.getSize(widthMeasureSpec);
        setMeasuredDimension(w,h);
    }
```

上面的代码其实跟不重写是一样的效果，但其中的` setMeasuredDimension(w,h)`是设置自定义尺寸的关键，你可以把它的参数改成一个确定的值，也可以根据从layout中得到的值去运算。函数里的代码没有太大的参考意义，建议根据具体情况来写代码，下面有一篇专门介绍这个的博客。

关于`onMeasure`方法这里不多讲，推荐一篇CSDN的博客：[自定义View之onMeasure()](http://blog.csdn.net/u012604322/article/details/17093421)。

## 应用于layout

---

修改过的自定义SurfaceView自然是要用到layout中的，其使用方法也比较简单。一般的IDE（如Android Studio）的layout是有视图环境的，可以从侧边栏的CustomView里选择你的自定义SurfaceView，然后像其他View一样来摆放 即可。

如果不使用视图环境而是代码环境，你的代码应当这样写：

``` xml
    <cn.demo.MySurfaceView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/customview" />
```

你也可以精确定义视图尺寸，但既然是自定义SurfaceView你可以不用十分关心其他的参数，这些在自定义类里面处理就好了。

## 自定义属性

---

关于自定义属性这块跟SurfaceView关系不大，所以这里不详细讲，再来一篇CSDN的博客：[自定义View自定义属性](http://blog.csdn.net/psh24053/article/details/7517029)。

# END