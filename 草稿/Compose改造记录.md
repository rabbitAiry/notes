#### 垂直风格
整个UI由3部分组成，分别用于显示线路信息、显示当前站点/宣传内容/下一站的走字屏、以及站点列表

从走字屏开始讲起，走字屏存在这两种动画效果，分别是展示“尊老爱幼..”的一大段文字时，文字从右往左逐渐移动；以及切换文字时，文字带有从上而下的进场效果。走字屏是使用TextView改造而成的，以下先来阐述一下它的运作原理

首先提供一个setText()方法用于更换文字，由于setText方法在TextView中带有final标志位导致无法重写，于是只好使用一个新的setText方法（传入的参数不同）再调用其父类方法的方式实现设置文本。

设置文本时，先取消所有动画，然后通过文本长度来判断传入的文字适合哪一种动画。文本长度是通过Paint类的measureText方法来完成的，而与之对比的宽度则是当前View的宽度。判断后直接调用自己写的scrollHorizontal()或者scrollVertical()方法设置动画状态

```Kotlin
fun setText(text: String) {  
    super.setText(text, BufferType.NORMAL)  
    cancelAnimation()  
    val textWidth = paint.measureText(text)  
    if(textWidth>width)scrollHorizontal(textWidth)  
    else scrollVertical()  
}
```

为什么要设置动画状态呢，因为动画是通过不断通过invalidate()方法让文字在相应的位置重新得到绘制

scrollHorizontal()设置了横向滚动文字的动画状态，动画基本上通过ValueAnimator进行设置，指定了文本开始绘制的位置以及介绍位置是从View的最右侧开始，直到所有文本都向左移动直到隐藏起来为止。每次更新值后都会调用一次invalidate()触发重新绘制，使得文本看起来真的像是一直在移动一样。这里我将这个动画设置成了无限期运行的动画

```kotlin
private fun scrollHorizontal(textWidth: Float) {  
    status = Status.HORIZONTAL_ANIMATION  
    horizontalAnimator = ValueAnimator.ofFloat(width.toFloat(), -textWidth)  
    horizontalAnimator?.let{  
        it.addUpdateListener { va ->  
            animateX = va.animatedValue as Float  
            invalidate()  
        }  
        it.interpolator = LinearInterpolator()  
        it.duration = text.length*800L  
        it.repeatCount = -1  
        it.start()  
    }  
}
```
垂直动画的设置基本同理。因为动画只会执行一次，所以我需要在动画结束时将状态更改为没有动画的状态，这需要使用监听器AnimatorListenrAdapter并重写其onAnimationEnd()实现

```kotlin
private fun scrollVertical() {  
    status = Status.VERTICAL_ANIMATION  
    verticalAnimator = ValueAnimator.ofFloat(0f, mBaseline)  
    verticalAnimator?.let{  
        it.addListener(object:AnimatorListenerAdapter(){  
            override fun onAnimationEnd(animation: Animator) {  
                status = Status.NO_ANIMATION  
                invalidate()  
            }  
        })  
        it.addUpdateListener { va ->  
            animateY = va.animatedValue as Float  
            invalidate()  
        }  
        it.duration = 300  
        it.start()  
    }  
}
```

回到实现绘制的onDraw()方法中，文字的绘制不再是使用之前TextView的方式了（其实我也不知道TextView是怎么绘制文本的，没有读懂），而是自己定义了一个drawAllText()方法负责将文本绘制到相应位置中。至于这个方法，其实只是调用canvas.draw()而已

```kotlin
override fun onDraw(canvas: Canvas) {  
    val n = text.length  
    paint.getTextBounds(text.toString(), 0, n, rect)  
    mBaseline = if(mBaseline!=0f) mBaseline else baseline.toFloat()  
    when(status){  
        Status.NO_ANIMATION -> drawAllText(canvas, paddingLeft.toFloat(), mBaseline)  
        Status.HORIZONTAL_ANIMATION -> drawAllText(canvas, animateX, mBaseline)  
        Status.VERTICAL_ANIMATION -> drawAllText(canvas, paddingLeft.toFloat(), animateY)  
    }  
}

private fun drawAllText(canvas: Canvas, x: Float, y: Float){  
    canvas.drawText(text, 0, text.length, x, y, paint)  
}
```

为了防止内存泄漏，我还设置了在onDetachedFromWindow()时结束所有动画

```kotlin
override fun onDetachedFromWindow() {  
    super.onDetachedFromWindow()  
    cancelAnimation()  
}
```

走字屏就讲完了，接下来阐述站点列表。站点列表其实只是单纯地使用RecyclerView罢了，不过我对它进行了一些自定义，所以提一下。首先我的项目希望RecyclerView是不能够响应触摸事件的，所以在RecyclerView外围，我设置了一个用于消耗点击事件的View，它只会消耗点击事件

```kotlin
class ConsumeTouchView : View {  
    constructor(context: Context): super(context)  
    @JvmOverloads constructor(context: Context,  
                attributeSet: AttributeSet?,  
                defStyleAttr: Int = 0,  
                defStyleRes: Int = 0):super(context, attributeSet, defStyleAttr, defStyleRes)  
  
    @SuppressLint("ClickableViewAccessibility")  
    override fun onTouchEvent(event: MotionEvent?): Boolean = true  
}
```

之后，项目还希望RecyclerView能够根据站点在列表中的位置自动滚动。RecyclerView的smoothScrollToPosition就是适合的方法，还自带了一些动画。因为我希望显示的站点信息不要总是在最底部的item中，所以想了一些骚操作，干脆移动到当前站点加上一个偏移值后的位置吧。这个方法最大的好处是不会出现溢出，看来RecyclerView帮我处理了溢出的问题了。

```kotlin
override fun displayNextStation() {  
    binding.listStations.smoothScrollToPosition(stations.currIdx + 6)  
    adapter.nextStation()  
}
```

但我又嫌弃RecyclerView的滚动动画太快了，不够生动。通过查看源码的方式，重写了layoutmanager的smoothScrollToPosition()，让这个动画放慢20倍吧

```kotlin
binding.listStations.adapter = adapter  
binding.listStations.layoutManager = object : LinearLayoutManager(context) {  
    override fun smoothScrollToPosition(  
        recyclerView: RecyclerView?,  
        state: RecyclerView.State?,  
        position: Int  
    ) {  
        val scroller = object : LinearSmoothScroller(context) {  
            override fun calculateTimeForScrolling(dx: Int): Int =  
                super.calculateTimeForScrolling(dx) * 20  
        }  
        scroller.targetPosition = position  
        startSmoothScroll(scroller)  
    }  
}  
binding.listStations.itemAnimator = null  
binding.listStations.setHasFixedSize(true)
```

列表里面同样有可以讲的内容，注意到下一站的站点左侧圆圈是一直在闪烁的，所以我需要在adapter中为对应的item设置动画。闪烁效果的实现只是单纯地两个Drawable之间频繁切换而已，既然如此，使用Handler就可以实现了。因为整个列表中只有一项是需要频繁切换drawable的，所以我会选择使用notifyItemChanged()

```kotlin
fun nextStation() {  
    currStationIdx++  
    handler.removeCallbacksAndMessages(null)  
    notifyItemChanged(currStationIdx-1)  
    if(statusRunning)shine()  
}  
  
fun stationArrived(){  
    statusRunning = false  
    notifyItemChanged(currStationIdx)  
    handler.removeCallbacksAndMessages(null)  
}  
  
fun busRunning(){  
    statusRunning = true  
    shine()  
}  
  
private fun shine(){  
    shiningOn = !shiningOn  
    notifyItemChanged(currStationIdx)  
    handler.postDelayed({ shine() }, 500)  
}
```


#### 广州公交风格
对于这个风格的pids，在站点到站时和车辆运行时展示的是两个界面，前者是十分简单的展示当前站点位置，后者则是展示站点列表。两者之间的切换全靠经典的设置visibility实现。这里只讲后者

站点列表特殊之处在于文本是竖向的，同时每个item都是等宽的。当文本高度超过界面时，文本会有来回上下滚动的动画。同时下一站的站点上方的圆圈会一直地在红色和灰色之间变化。

首先这个站点列表没有使用RecyclerView实现，而是单纯地使用了LinearLayout实现，这是因为所有的站点都必须展示在屏幕上。

在Fragment初始化时，通过post传递线路信息，以确保在此时能够获取View的宽高信息。获取信息后，第一步先测量每个View可以拥有的宽度，然后再测量每个文字可以拥有的大小。获取了view的宽度和文本的宽度后，不断创建表示单个站点的view并添加到LinearLayout中。而这个View是我自定义的FlowingText，它并没有与继承自TextView。

```kotlin
var stations: StationListInfo? = null  
    set(value) {  
        field = value  
        initTextViewArray(value!!)  
    }

private fun initTextViewArray(stations: StationListInfo) {  
    val n = stations.stations.size  
    val perWidth = width.toFloat() / n  
    textSize = getTextSize(perWidth * 3 / 4)  
    for (i in 0 until n) {  
        val view = FlowingText(  
            context,  
            isFirst = i == 0,  
            isEnd = i == n - 1,  
            text = stations[i].name,  
            status = if (i > stations.currIdx) Status.NOT_ARRIVE  
            else if (i == stations.currIdx) Status.CURR  
            else Status.ARRIVED  
        )  
        val params = LayoutParams(0, ViewGroup.LayoutParams.MATCH_PARENT)  
        params.weight = 1f  
        view.layoutParams = params  
        addView(view)  
    }  
}
```

获取文字宽度是百度来的结果，不过我在此基础上使用了二分查找提高了效率。

```kotlin
/**  
 * 期望是每一行只有一个字  
 */  
private fun getTextSize(width: Float): Float {  
    val max = 200  
    var left = 0  
    var right = max  
    val paint = Paint()  
    while (left < right) {  
        val mid = (left + right + 1) / 2  
        paint.textSize = mid.toFloat()  
        val tempWidth = paint.measureText("测")  
        if (tempWidth > width) right = mid - 1  
        else left = mid  
    }  
    return left.toFloat()  
}
```

接下来看FlowingText。在知道了自身宽度的情况下绘制了顶部的圆圈和下面的文本。圆圈使用canvas.drawCircle()，而文本通过canvas.drawText()完成。在绘制文本后就可以知道文本高度是否超过底部界限了，如果超过了，那就给这行文字开始动画上下徘徊。

```kotlin
@SuppressLint("DrawAllocation")  
override fun onDraw(canvas: Canvas) {  
    super.onDraw(canvas)  
    backgroundPaint.color = if (status == Status.CURR) blue500 else Color.WHITE  
    canvas.drawColor(if (status == Status.CURR) blue500 else Color.WHITE)  
  
    val radius = width / 8f * 3  
    val center = width / 2f  
    val parent = parent as LineMapLinearLayout  
    val textSize = parent.textSize  
    textPaint.textSize = textSize  
    textHead = center * 2 + textSize  
    if (!textInAnimate) textY = textHead  
    textEnd = drawText(width / 8f, textY, canvas, width / 4 * 3f)  
    if (textEnd > height && !textInAnimate) textAnimate(textHead, textHead - (textEnd - height))  
  
    canvas.drawRect(0f, 0f, width.toFloat(), center * 2, backgroundPaint)  
    linePaint.strokeWidth = radius / 2  
    canvas.drawLine(  
        if (isFirst) center else 0f,  
        center,  
        if (isEnd) center else width.toFloat(),  
        center,  
        linePaint  
    )  
    canvas.drawCircle(center, center, radius, circlePaint)  
}
```

先来看文字上的动画。文字的动画相对复杂，先是向下移动，然后停顿一会，然后再向上移动，然后又停顿一会。为此，我需要使用handler的延时功能帮我完成动画内容。

```kotlin
private fun textAnimate(from: Float, to: Float) {  
    textInAnimate = true  
    textAnimation = ValueAnimator.ofFloat(from, to)  
    textAnimation.let {  
        it.addUpdateListener { va ->  
            textY = va.animatedValue as Float  
            invalidate()  
        }  
        it.addListener(object : AnimatorListenerAdapter() {  
            override fun onAnimationRepeat(animation: Animator) {  
                animation.pause()  
                handler?.postDelayed({  
                    animation.resume()  
                }, 2000)  
            }  
        })  
        it.duration = ((from - to) * 20).toLong()  
        it.repeatCount = -1  
        it.repeatMode = ValueAnimator.REVERSE  
        it.start()  
    }  
}
```

圆圈的动画则是可以仅通过invalidate()就完成的，只不过因为涉及到颜色上所以有些棘手

```kotlin
private fun statusAnimate() {  
    statusAnimation = ValueAnimator.ofFloat(0f, 1f)  
    val from = intArrayOf(Color.red(gray500), Color.green(gray500), Color.blue(gray500))  
    val to = intArrayOf(Color.red(red500), Color.green(red500), Color.blue(red500))  
    statusAnimation.let {  
        it.addUpdateListener { va ->  
            val percentage = va.animatedValue as Float  
            circlePaint.color = Color.rgb(  
                (from[0] + (to[0] - from[0]) * percentage).toInt(),  
                (from[1] + (to[1] - from[1]) * percentage).toInt(),  
                (from[2] + (to[2] - from[2]) * percentage).toInt()  
            )  
            invalidate()  
        }  
        it.duration = 1500  
        it.repeatCount = -1  
        it.repeatMode = ValueAnimator.REVERSE  
        it.start()  
    }  
}
```