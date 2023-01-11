#### 获取字符串的高度宽度
- 自定义View，使用`mPaint.setTextSize(20.0f)`设置文字
	- 使用`getTextBounds()`获取
	```java
	Rect rec = new Rect();
	mPaint.getTextBounds(text, 0, text.length(), rec);
	int width = rec.right - rec.left;
	int height = rec.bottom - rec.top;
	```
	- 使用`measureText获取`
	```java
	float width2 = mPaint.measureText(testString);
	```
	- 两者结果有差距，但差不多


#### TextView文字大小自适应
- 通过不断比较，找到最大的小于TextView宽度的文本
```java
//字符最大的大小
float defaultSize = 40.0f;

for(;;)
{
	Paint paint = mTextView.getPaint();
	paint.setTextSize(defaultSize);
	float wm = paint.measureText(str);
	if(wm <= width)
		break;
	else
		//每次减小的步长
		defaultSize -= 0.1;
}
return defaultSize;
```

#### setText()
- setText()方法
```java
@UnsupportedAppUsage  
private void setText(CharSequence text, BufferType type,  
                     boolean notifyBefore, int oldlen) {  
    // ...
	
    setTextInternal(text);
    
	// ...

    if (mLayout != null) {  
	    // 检查是否需要新的layout还是仅仅一个新的text layout
        checkForRelayout();  
    } 
	// ...
}
```

- checkForLayout()方法
	- 依据是height或者width是否是可变动的，与文字长度无关
```java
// Dynamic width, so we have no choice but to request a new  
// view layout with a new text layout.  
nullLayouts();  // 回收layout
requestLayout();  // 得到新的layout
invalidate();   // 重新调用onDraw()
```



#### onDraw()
- 在onDraw()中决定绘制文字的是这行代码
```java
layout.draw(canvas, highlight, mHighlightPaint, cursorOffsetVertical);
```
- 最终会在Layout的drawText()中看到调用canvas绘制text
```java
canvas.drawText(buf, start, end, x, lbaseline, paint);
```


> constrain 限制



