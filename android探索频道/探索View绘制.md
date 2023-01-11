#### View的初始化
- 对于仅使用context作为参数的构造器
	- 该构造器用于在代码中构建一个view
- 对于使用4个参数的构造器
	- 从attributeSet、style资源文件、defStyleAttr和defStyleRes综合得到属性数组a
	```java
	final TypedArray a = context.obtainStyledAttributes(  
        attrs, 
        com.android.internal.R.styleable.View, 
        defStyleAttr, 
        defStyleRes);
	```
	- 初始化所有的参数，然后从数组a中填入用户选择的参数
	- 回收数组a


#### View的三大流程从何开始
- 在自定义View中，发现setContentView期间没有调用view的三大绘制流程onMeasure、onLayout、onDraw
	- 三大流程不在onCreate阶段完成，setContentView仅仅是将View实例化了而已
```
? D/MainActivity: onCreate: before setContentView()
? D/MainActivity: onCreate: after setContentView()
? D/MainActivity: onCreate: after text set
? D/MainActivity: onMeasure: 
? D/MainActivity: onMeasure: 
? D/MainActivity: onLayout: 
? D/MainActivity: onDraw: 
```

- 尝试通过post()方法找到对应handler被实例化的时候
	- mhandler是静态内部类View$AttachInfo的final成员变量
	- 注释提到该handler由view的android.view.ViewRootImpl提供
	- 时间原因待补充




> recursive 递归的
