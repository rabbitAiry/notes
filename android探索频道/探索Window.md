- 新建window（悬浮窗）
	- 设置LayoutParams
	- 获取windowManager
	- manager.addView(view, params)
- 更新window的params
	- 设置LayoutParams
	- manager.updateViewLayout(view, params)
- 销毁window
- Activity中初始化windowmanager
	```Java
	mWindow = new PhoneWindow(/**/);
	mWindow.setWindowManager(/**/);
	mWindowManager = mWindow.getWindowManager();
	```

- Window类
	- 持有WindowManager
	- 持有并负责生成DecorView
- WindowManagerImpl类
	- 持有Window
- WindowManagerGlobal类
	- 为每个添加的View生成一个ViewRootImpl