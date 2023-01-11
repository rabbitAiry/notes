# Android 基础篇

- 日志工具：级别越下越高
    - Log.v() 打印最为琐碎的日志信息verbose
    - Log.d() 打印调试信息debug
    - Log.i() 打印重要信息info
    - Log.w() 打印警告信息warn
    - Log.e() 打印错误信息error


## Activity篇
##### Activity生命周期
- 返回栈：安卓使用Task(任务)来管理活动，一个Task就是一组存放在栈里的活动的集合，这个栈称为Back Stack(返回栈)
- 4种活动状态：运行、暂停（仍可见）、停止（不可见）、销毁
- 7种生命周期回调：onCreate(), onStart(), onResume(), onPause(), onStop(), onDestroy(), onRestart()
- 3种生存期：完整生存期、可见生存期、前台生存期
- 保存临时数据onSaveInstanceState()。在获取还原时应该先检查其是否为空
- activity的启动模式
	- standard（默认）：常规的栈，即使是同样的activity也会被重复进栈
	- singleTop：仅当返回栈的栈顶也就是该activity，不再创建新的activity实例
	- singleTask：如果返回栈中存在该activity，则不断出栈直到栈顶为该activity
	- singleInstance：包含了singleTask模式的所有特性，除此之外，每次启动新activity时都会放入其他返回栈中
	- 需要在Manifest的activity标签内指定launchMode
- 获取自身信息
	- 返回自身id：this.toString()
	- 返回所在栈id：getTaskId()

##### Fragment
- Fragment概念
	- 代表Activity中用户界面的独立部分，可以完全模块化Activity，也可以在多个Activity中重复使用这些fragment
	- fragment必须始终嵌入到Activity中，fragment拥有自己的生命周期，但其会受到Activity的生命周期的影响
	- 可以在运行时动态地添加或删除
- 静态fragment：只需使用布局文件xml指定
	- 使用fragment标签，并在属性name中指定对应fragment类
  ```xml
  <fragment
      ...
      android:name="com.example.app.LeftFragment"/>
  ```
- 动态Fragment
	- 创建Fragment子类并重写其`onCreateView()`方法
	- 通常使用FrameLayout作为其容器
	- 使用FragmentManager展示
		- fragment中模拟返回栈效果（按下Back键后，返回上一个fragment）
			- 在transaction中多加一步`.addToBackStack(null)`即可
	  ```java
	  // transaction：事务
	  FragmentManager fm = getSupportFragmentManager();
	  fm.beginTransaction()
	    .add(R.id.容器, fragment对象).commit();
	  ```
- fragment的状态保存（bundle）
	- 重写`onSaveInstanceState(Bundle state)`，并将成员变量保存至此
	- 在`onCreateView()`中，尝试接收bundle
	- 在Activity中通过判断bundle是否为空的方式决定是否需要重新创建fragment
- Fragment状态
	- 运行状态
	- 暂停状态：相关的activity进入了暂停状态
	- 停止状态：相关的activity进入了停止状态，或者使用了addToBackStack()方法且fragment仍在栈种
	- 销毁状态：相关的activity被销毁，或调用了replace()、remove()方法，且没有使用addToBackStack()方法
- 主要的生命周期回调
	- onAttach()：碎片和活动建立关联时
	- onCreate()：创建时。先关联再创建
	- onCreateView()：创建（加载）布局时
	- onActivityCreated()：与fragment关联的activity创建完毕时
	- onDestroyView()：布局被移除时（使用栈时，只移除布局，不销毁fragment）
	- onDetach()：解除关联时
	- fragment在切换时的生命周期（fm.replace）
  ```
  D/life:Fragment1: onAttach
  D/life:Fragment1: onCreate
  D/life:Fragment1: onCreateView
  D/life:Fragment2: onAttach
  D/life:Fragment2: onCreate
  D/life:Fragment2: onCreateView
  D/life:Fragment1: onDestroyView
  D/life:Fragment1: onDetach 
  ```
	- fragment的当前activity销毁（如：返回至上一级activity）
  ```
  D/life:Activity2: onStop
  D/life:Fragment2: onDestroyView
  D/life:Fragment2: onDetach 
  D/life:Activity2: onDestroy
  ```
- 获取Fragment、获取Activity
```java
// 获取fragment
MyFragment frag = (MyFragment)getFragmentManager()
.findFragmentById(R.id.my_fragment);

// 获取Activity
MainActivity acti = (MainActivity)getActivity();
```
- 利用Fragment提供平板布局
- 在res文件夹下使用限定符提供平板ui
	- 新建Android Resource Directory：选择layout文件、最小宽度为600dp，然后再新建一个和原来layout文件同名的activity.xml
	- 将需要合并的页面也放在这个布局中
- 在Activity中通过findViewById()查找只有平板布局存在的id，从而判断当前布局
```java
isTwoP = getActivity().findViewById(R.id.frag) != null;
```


##### Widget
- 在xml中设置widget参数
	- 在res/xml下创建xml文件
	- updatePeriodMillis用于设置更新时间，安卓规定大于30min
  ```xml
  <appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
      android:initialLayout="@layout/plant_widget"
      android:minHeight="40dp"
      android:minWidth="40dp"
      android:previewImage="@drawable/launcher_icon"
      android:resizeMode="horizontal|vertical"
      android:updatePeriodMillis="1800000"
      android:widgetCategory="home_screen" />
  ```
- 从Widget打开Activity
	- 创建AppWidgetProvider子类
	- 在静态方法updateAppWidget中
	    - 构建PendingIntent，intent指向要打开的activity
	    - 构建RemoteView，并传入widget布局
	    - 为RemoteView设置点击事件，传入的参数为被点击对象的id以及intent
	    - 更新widget
	- 在Manifest中
		- 为上述WidgetProvider添加一个receiver
		- 并设置intent-filter为APPWIDGET_UPDATE
		- 并设置meta-data指向widget配置文件
```java
static void updateAppWidget(Context context, AppWidgetManager appWidgetManager,
						  int appWidgetId) {

  // Create an Intent to launch MainActivity when clicked
  Intent intent = new Intent(context, MainActivity.class);
  PendingIntent pendingIntent = PendingIntent.getActivity(context, 0, intent, 0);
  // Construct the RemoteViews object
  RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.widget);
  // Widgets allow click handlers to only launch pending intents
views.setOnClickPendingIntent(R.id.widget_plant_image, pendingIntent);
  // Instruct the widget manager to update the widget
  appWidgetManager.updateAppWidget(appWidgetId, views);
}
```
```xml
<receiver android:name=".PlantWidgetProvider">
  <intent-filter>
	  <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
  </intent-filter>
  <meta-data
	  android:name="android.appwidget.provider"
	  android:resource="@xml/plant_widget_info"         />
</receiver>
```

- API14及更高级别后，android会自动在小部件周围留下外边距
	- 在res/values下创建dimens.xml文件并指定margin大小
	- 在res/values-v14下也创建dimens.xml文件并指定margin大小

##### *2 使用服务

> 点击widget上的水滴时，使用服务为所有植物浇水

- 创建服务类继承自IntentService（PlantWateringService）
  
  - 自定义一个Action的tag
  
  ```java
  public static final String ACTION_WATER_PLANTS = "com.example.mygarden.action.water_plants";
  ```
  
  - 构造器中，传入子类名称
  
  ```java
  public PlantWateringService() {
      super("PlantWateringService");
  }
  ```
  
  - 开启服务的方法
    - 构造intent》为intent设置action》使用context启动服务
  
  ```java
  public static void startActionWaterPlants(Context context) {
      Intent intent = new Intent(context, PlantWateringService.class);
      intent.setAction(ACTION_WATER_PLANTS);
      context.startService(intent);
  }
  ```
  
  - 重写onHandleIntent()方法
    - 获取action，如果是自定义的目标action，则执行下一步操作
  
  ```java
  if (intent != null) {
      final String action = intent.getAction();
      if (ACTION_WATER_PLANTS.equals(action)) {
          // handleActionWaterPlants();
      }
  }
  ```
  
  - 上述提到的下一步操作：为所有植物浇水
    - 使用ContentResolver更新数据

- 在Manifest中，注册一个service
  
  ```xml
  <service android:name=".PlantWateringService" />
  ```

- 在自定义的AppWidgetProvider子类中，为水滴添加点击事件
  
  - 构建intent》为intent设置action为PlantWateringService中的自定义事件》pendingIntent》在remoteView中设置点击事件

##### *3 使用服务更新widget

> 服务中查询快死的（上次浇水时间最远的）植物并显示到widget中

- 在IntentService的子类中
  - 自定义一个新的actionTag：查询并更新照片
  - 创建启用该服务的方法（见 *2）
  - 在onHandleIntent中为上述tag增加入口，并调用如下操作
    - 查询数据库（使用ContentProvider）
    - 获取图片，调用AppWidgetProvider中的方法设置图片
- 在AppWidgetProvider的子类中
  - updateAppWidget方法中，为ImageView设置图片
  - onUpdate()方法中改为使用服务查询并更新照片
- 在其他更新数据库的操作中，同样调用服务查询并更新照片

##### *4 widget之间传递信息

> 将“为所有植物浇水”改为“为一棵植物浇水”，根据植物ID浇水，所以需要传递信息。除此以外：若最近浇水时间不足限定范围，则隐藏widget中的浇水按钮；单击该植物时，启动植物详细页面Activity

- 在IntentService的子类中
  
  - 自定义一个新的extraTag：传递id数据
  
  ```java
  public static final String EXTRA_PLANT_ID = "com.example.android.mygarden.extra.PLANT_ID";
  ```
  
  - 在重写方法onHandleIntent()中，获取intent中的id并传递

- 在AppWidgetProvider的子类中
  
  - 如果植物id非空，则放入id（使用上述extraTag）

##### *5 自适应的widget

> widget不大时，只展示一棵植物；够大时，展示多科植物（使用GridView）
> 
> 与普通layout不同，widget layout不使用adapter，而是使用RemoteViewsFactory和RemoteViewsService来构建视图

- 构造GridView版本的布局，在GridView参数中指出栏数numColumns

- 创建RemoteViewsService的子类
  
  - 重写方法onGetViewFactory，这会返回一个GridRemoteViewsFactory的实例

- 创建实现RemoteViewsFactory的类（RemoteViewsService的内部类）
  
  - 构造器：设置context为成员变量，并传入context
  - 重写方法onDataSetChanged：关闭cursor、重新获取
  - 重写方法onDestroy：关闭cursor
  - 重写方法getViewAt
    - 像onBindViewHolder一样使用该方法
    - 因为为每个item都设置一个PendingIntent的代价很高，所以这是不被允许的。应该每个item都创建一个intent，然后在被点击时传递给PendingIntentTemplate构造Pending Intent
    - 设置点击事件但不需要构建pendingIntent（使用views.setOnClickFillInIntent()）

- 在IntentService的子类中
  
  - 在更新植物的方法中，调用AppWidgetManager的notifyAppWidgetViewDataChanged()方法以更新gridView

- 在AppWidgetProvider的子类中
  
  - 在静态方法updateAppWidget中，获取widget长和宽以指定不同的视图
  
  ```java
  {
      Bundle options = appWidgetManager.getAppWidgetOptions(appWidgetId);
      int width = options.getInt (AppWidgetManager.OPTION_APPWIDGET_MIN_WIDTH);
      RemoteViews rv;
      if (width < 300) {
          // ...
      } else {
          // ...
      }
  }
  ```
  
  - 重写onAppWidgetOptionsChanged方法。每当widget大小发生变化都会调用该方法
    - 更新widget然后调用父类方法
  - 在创建gridView的方法中
    - 给views设置PendingIntentTemplate以及emptyView

- 在Manifest中
  
  - 注册RemoteViewsService
  - 申请权限BIND_REMOTEVIEWS
  
  ```xml
  <service
      android:name=".GridWidgetService"
      android:permission="android.permission. BIND_REMOTEVIEWS" />
  ```

[TOC]

## Intent篇

##### Intent
- Intent类型
	- 显示intent：指定了打开的组件
	- 隐式intent：指定了启动活动的action和category，然后交由系统去分析这个intent并找出合适的组件
		- 应该先检查是否有能够响应的activity
	    - 每个intent只能指定一个action，但能指定多个category
	    - data不是必须的，但只有intent中的uri中scheme指向的数据类型相同才能响应（若有uri）
	- 使用隐式intent启动其他程序
		- 指定Intent的action为`Intnet.ACTION_VIEW`，这是一个系统内置动作；同时第二个参数填入URI
```java
Intent intent = new Intent(/*action*/)
if(intent.resolveActivity(getPackageManager()) != null){
	startActivity(intent);
}
```
```xml
<intent-filter>
	<action android:name=" ... "/>
	<category andriod:name="android.intent.category.DEFAULT"/>
	<data android:scheme="http">
</intent-filter>
```
- Uri
	- URI：统一资源标识符（Identifier）
	- URL：统一资源定位符（Loactor）（即网址）
	- URI完整形式：`scheme:[//[user:password@]host[:port]][/]path[?query][#fragment]`
	    - \[]内为可选部分
	    - scheme描述指向的资源类型：http、ftp、https、geo（地图）、tel（拨打电话）……
	    - authority部分（第一个中括号）表示登录用户名、主机（站点）
	    - 资源路径（path）
	    - query部分（？q=）以问号开头
	    - `#`指示路径资源将使用的辅助数据
	- 构建地图Uri(scheme为geo时)
		- 初始化Uri.builder：`Uri.Builder = new Uri.Builder()`
		- 添加参数：`builder.scheme("geo").path("0,0").appendQueryParameter("q","仲恺农业工程学院")`
- 分享文本、图片等媒体类型Uri
	- 媒体类型字符串由类型、子类型、可选参数组成，如：text/html; chaeset=UTF-8
	- 使用ShareCompat.IntentBuilder来共享
```java
String mimeType = "text/plain";
String title = "标题;
ShareCompat.IntentBuilder
	.from(this)
	.setType(mimeType)
	.setChooserTitle(title)
	.setText("分享内容")
	.startChooser();
```

- 使用intent传递数据
	- 向Intent添加数据，获取数据以及检查是否有数据：putExtra()、hasExtra()、getStringExtra()
	- 返回数据给上一个活动：
		- 在activity1中重写：startActivityForResult()、onActivityResult()
		- 在activity2中调用：setResult()、finish()
			- 在按下back键返回时设置：重写onBackPressed()


## 界面篇
- 无障碍：为含有图片的view增加描述，因为含有文字的view会被安卓朗读
	  `android:contentDescription="@string/origin_label"`
- 国际化
	- 在`values`同级处添加文件夹`values-语言代号`
	- 安卓会自动加载和本地语言匹配的语言
	- 如果对不打算翻译的字符串使用字符串资源，可以在`strings.xml`中的属性设为`translatable="false"` 
##### UI部件
- ProgressBar
	- 可以指定不同样式，如：横条。横条样式需要手动增加进度
```xml
<ProgressBar>
...
style="?android:attr/progressBarStyleHorizontal"
android:max="100"
</ProgressBar>
```
  ```java
  int progress = progressBar.getProgress();
  progress += 10;
  progressBar.setProgress(progress);
  ```

- AlertDialog
```java
AlertDialog.Builder dialog = new AlertDialog.Builder(this);
dialog.setTitle("This is a Dialog");
dialog.setMessage("Something important.");
dialog.setCancelable(false);
dialog.setPositiveButton("OK", new DialogInterface.OnClickListener() {
  @Override
  public void onClick(DialogInterface dialog, int which) {
	  // ...
  }
});
dialog.setNegativeButton("Cancel", new DialogInterface.OnClickListener() {
  @Override
  public void onClick(DialogInterface dialog, int which) {
	  // ...
  }
});
dialog.show();
```

- ProgressDialog展示进度的对话框（不建议使用，交互性变差）
  ```java
  ProgressDialog progressDialog = new ProgressDialog(this);
  progressDialog.setTitle("This is a ProgressDialog");
  progressDialog.setMessage("Loading..");
  progressDialog.setCancelable(true);
  progressDialog.show();
  ```

- ActionBar操作栏
	- 隐藏ActionBar
	```kotlin
	supportActionBar?.hide()
	```
	- 返回键（需要自行重写`onOptionsItemSeleted()`）
  ```kotlin
    supportActionBar?.setDisplayHomeAsUpEnabled(true);
  ```
	- 为操作栏添加按钮
	```xml
	<menu xmlns:android="http://schemas.android.com/apk/res/android" >
	  <item
		  android:id="@+id/action_favorite"
		  android:icon="@drawable/ic_favorite_black_48dp"
		  android:title="@string/action_favorite"
		  app:showAsAction="ifRoom"/>
	</menu>
	```
	- 响应操作栏点击操作：重写方法`onOptionsItemSeleted()`
	- 透明的系统状态栏（5.0+）
		- 设置控件的属性：为所有出现在顶部的view及viewgroup都设置此属性
	  ```xml
	  android:fitsSystemWindows="true"
	  ```
		- themes.xml中设置透明顶部栏
	  ```xml
	  <style name="FruitActivityTheme" parent="Theme.Firstline_Material_Design">
	      <item name="android:statusBarColor">@android:color/transparent</item>
	  </style>
	  ```


- 使用View Binding
	- 启用：在module级别的build.gradle中
	```groovy
	viewBinding{
	  enabled = true
	}
	```
	- 默认每个布局文件都会生成一个binding类，名字为“布局名+Binding”
		- 在Activity中使用
	  ```java
	  private ResultProfileBinding binding;
	  
	  @Override
	  protected void onCreate(Bundle savedInstanceState) {
	      super.onCreate(savedInstanceState);
	      binding = ResultProfileBinding.inflate(getLayoutInflater());
	      View view = binding.getRoot();
	      setContentView(view);
	  }
	  ```
		- 在fragment中使用
	  ```java
	  private ResultProfileBinding binding;
	  
	  @Override
	  public View onCreateView (LayoutInflater inflater,
	                            ViewGroup container,
	                            Bundle savedInstanceState) {
	      binding = ResultProfileBinding.inflate(inflater, container, false);
	      View view = binding.getRoot();
	      return view;
	  }
	  
	  @Override
	  public void onDestroyView() {
	      super.onDestroyView();
	      binding = null;
	  }
	  ```

##### Material的UI部件
- DrawerLayout：侧划滑动菜单
	- xml中的DrawerLayout控件内，第一个子控件为主屏幕中显示内容，第二个子控件为菜单中内容
	- 第二个子控件通过设置layout_gravity以表示从哪一侧滑动
	- 代码中控制开关抽屉：`drawerLayout.openDrawer(GravityCompat.START)`
- Snackbar：Toast的拓展
```kotlin
Snackbar.make(v, "Data deleted", Snackbar.LENGTH_SHORT)
	.setAction("Undo"){
	  Toast.makeText(this, "Data restored", Toast.LENGTH_SHORT).show()
	}
	.show()
```
- CoordinatorLayout：增强的FrameLayout
	- 可以监听各种子控件，并自动作出合理的响应以防止控件被遮挡
- AppBarLayout：解决CoordinatorLayout子控件被遮挡的情况。作为其子部件存在
	- 将Toolbar嵌套在AppBarLayout其中，然后为其余的子部件指定布局行为
		- 自动隐藏toolbar（toolbar随着用户操作决定是否出现）
			- 为toolbar设置属性
  ```xml
  app:layout_behavior="@string/appbar_scrolling_view_behavior"
  ```
  ```xml
  app:layout_scrollFlags="scroll|enterAlways|snap"
  ```
- CollapsingToolbarLayout
	- 只能作为AppBarLayout的直接子布局来使用
	- 属性
		- contentScrim：折叠状态下toolbar的背景色
		- scrollFlags
			- scroll：表示layout会随页面滚动而滚动
			- exitUntilCollapsed：完成折叠后就不再移出屏幕
			- enterAlways：向下划动时，toolbar也向下滑动
			- snap：还未完全隐藏时，自动决定是否隐藏
		- layout_collapseMode
			- pin：表示折叠过程中位置始终不变
			- parallax：表示折叠过程中会产生一定的错位偏移
  ```xml
  <com.google.android.material.appbar.CollapsingToolbarLayout
      android:id="@+id/collapsingToolbar"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:theme="@style/Theme.AppCompat.DayNight.DarkActionBar"
      app:contentScrim="@color/purple_200"
      app:layout_scrollFlags="scroll|exitUntilCollapsed">
  
      <ImageView
          android:id="@+id/fruitImageView"
          android:layout_width="match_parent"
          android:layout_height="match_parent"
          android:scaleType="centerCrop"
          app:layout_collapseMode="parallax"/>
      <androidx.appcompat.widget.Toolbar
          android:id="@+id/toolbar"
          android:layout_width="match_parent"
          android:layout_height="?attr/actionBarSize"
          app:layout_collapseMode="pin"/>
  </com.google.android.material.appbar.CollapsingToolbarLayout>
  ```
- NestedScrollView：能够响应滚动事件的ScrollView
- SwiperFreshLayout下拉刷新部件
	- 添加依赖
  ```groovy
  implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.1.0'
  ```
	- 在RecyclerView外嵌套一层SwipeRefreshLayout，即可拥有下拉刷新功能，但此时下拉后进度条不会隐藏
		- 在代码中设置
	  ```kotlin
	  swipeRefresh.setColorSchemeResources(R.color.teal_200)
	  swipeRefresh.setOnRefreshListener {
	      thread {
	          Thread.sleep(2000)
	          runOnUiThread {
	              swipeRefresh.isRefreshing = false
	          }
	      }
	  }
	  ```


##### 通知
- 通知属性
  
  <img src="https://developer.android.google.cn/images/ui/notifications/notification-callouts_2x.png" alt="包含基本详情的通知" style="zoom:50%;" />
  
	1. 小图标：`setSmallIcon()`
	2. 应用名称，由系统提供
	3. 时间戳，由系统提供，可用`setWhen()`替换或`setShowWhen(false)`隐藏
	4. 大图标：可选，`setLargeIcon()`
	5. 标题：可选，`setContentTitle()`
	6. 文本：可选，`setCotentText()`
	7. 设置优先级（重要程度）：`setPriority()`
	8. 设置通知的点按操作，可选：`setContentIntent()`
	9. 设置通知点击后自动取消，可选：`setAutoCancel(true)`
	10. 添加按钮，可选：`addAction()`，传入的intent指向后台服务
- 构建通知
```java
// 使用notificationCompat是因为通知的api不稳定
Intent intent = new Intent(this, AlertDetails.class);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);
PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, 0);

NotificationCompat.Builder builder = new NotificationCompat.Builder(this, CHANNEL_ID)
	  .setSmallIcon(R.drawable.notification_icon)
	  .setContentTitle("My notification")
	  .setContentText("Hello World!")
	  .setPriority(NotificationCompat.PRIORITY_DEFAULT)
	  // Set the intent that will fire when the user taps the notification
	  .setContentIntent(pendingIntent)
	  .setAutoCancel(true);
```
  ```java
  // 更多属性
  .setSound()
  .setVibrate()
  .setLights() // LED灯
  .setDefaults(NotificationCompat.DEFAULT_ALL);
  ```
- 创建通知渠道（仅安卓8用到）
```java
if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.O){
	// 构建NotificationChannel
	NotificationChannel channel = new NotificationChannel(
		  CHANNEL_ID,
		  "ChannelName",
		  NotificationManager.IMPORTANCE_HIGH);
	channel.setDescription("this is description");
	NotificationManager notificationManager = 
		getSystemService(NotificationManager.class);
	notificationManager.createNotificationChannel(channel);
}
```
- 展示通知
```java
NotificationManagerCompat notificationManager = NotificationManagerCompat.from(this);
notificationManager.notify(NOTIFICATION_ID,builder.build());
```
- 通知重要程度
	- Android8.0+：重要程度由通知的频道的priority决定
	- Android7.1-：重要程度均由通知的priority决定
	- urgent级别的通知会以弹出的形式通知


##### ListView
- MVC模式：Model数据模型 - Controller控制器 - View视图
- Adapter家族：
	- BaseAdapter：抽象类，是以下类的父类
		- SimpleAdapter：具有良好的扩展性
		- ArrayAdapter：最简单的adapter，自带了一些布局
- ListView设置表头表尾和分割线
	- addHeaderView()：这个方法需放在`setAdapter()`前
	- addFooterView()
	- xml属性
		- divider、dividerHeight分割线及其高度
		- footer\headerDividersEnabled 头或尾处是否设置分割条
- 属性
	- 设置列表从底部往上显示：xml属性stackFromBottom
	- 设置缓存颜色：xml属性cacheColorHint
	- 隐藏滑动条：setVerticalScrollBarEnabled(true)或xml属性scrollbars
	- 当ListView中数据为空时，展示不同的View：setEmptyView()
- 使用ViewHolder优化：减少findViewById的多次调用（从n个元素降为屏幕可展示的n个元素）
	- 自定义一个ViewHolder类（不需要继承），然后给所有需要绑定的组件添加成员变量
	- 依然使用convertView来绑定，但是绑定结果交给holder
	- 若convertView为空，则执行绑定操作并为convertView设置setTag
	- 若非空，则直接调用convertView.getTag()
```java
if(convertView == null){
	convertView = LayoutInflater
		.from(mContext)
		.inflate(R.layout.item,parent,false);
}
```

##### RecyclerView
- 概念
	- RecyclerView是通过回收已有的结构而不是持续创建新的列表项，所以它可以有效提高应用的时间效率和空间效率
	- Adapter类从数据源获得数据，并且将数据传递给正在更新其所持视图的 ViewHolder
- 在Activity中配置RecyclerView时，如果列表数据总数不变，可以通过调用`.setHasFixedSize(true)`降低消耗
- 应对数据变化
	- Adapter提供的方法
		- 指定任务添加的位置`notifyItemInserted()`
		- 删除`notifyItemRemoved()`
		- 重绘整个视图`notifyDataSetChanged()`
	- 使用ListAdapter：无需重绘视图，自动会审视变化内容，且自带动画
		- 在 Adapter 类中添加 DiffUtil 对象，并且复写 areItemsTheSame() 和 areContentsTheSame()
		- 将 Adapter 的父类由 RecyclerView.Adapter 改为 ListAdapter，并传入 DiffCall对象
		- ListAdapter 通过 submitList() 方法获取数据，该方法提交了一个列表来与当前列表进行对比并显示。也就是说您无需再重写 getItemCount()，因为 ListAdapter 会负责管理列表
		- 在 Activity 类中，调用 Adapter 的 submitList() 方法并传入数据列表。
		- 在 Adapter 类中，onBindViewHolder() 现在可以使用 getItem() 从数据列表中获取指定位置的元素了
- RecyclerView中添加头部或尾部
	- 为头部view创建adapter
	- 在Activity类中使用ConcatAdapter（构成一个新的adapter），传递给RecyclerView



## 多媒体篇
##### 相机
- 拍照前，先创建空白相片提供存储空间
	- 创建文件
	- 获取文件uri
	- 在manifest中声明provider
		- file_path.xml文件中，标签external_path下填入共享路径。name可以随便填，path为`.`时代表全部
```kotlin
// lateinit var outputImage: File
outputImage = File(externalCacheDir, "img.jpg")
if(outputImage.exists())outputImage.delete()
outputImage.createNewFile()
imageUri = 
	if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.N)
		FileProvider.getUriForFile(
			this,
			"包名.fileprovider", 
			outputImage)
	else Uri.fromFile(outputImage)
```

```xml
<!-- Manifest-->
<provider
	android:name="androidx.core.content.FileProvider"
	android:authorities="包名.fileprovider"
	android:exported="false"
	android:grantUriPermissions="true">
	<meta-data
		android:name="android.support.FILE_PROVIDER_PATHS"
		android:resource="@xml/file_paths"/>
</provider>

<!-- file_opaths.xml-->
<paths xmlns:android = "http://schemas.android.com/apk/res/android">
	<external-path name="my_images" path="/">
</paths>
```
- 启动摄像头
```java
Intent intent = new Intent("android.media.action.IMAGE_CAPTURE");
intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
startActivityForResult(intent, TAKE_PHOTO);
```
- 在onActivityResult中，获取照片并展示
```java
Bitmap bitmap = BitmapFactory.decodeStream(
	getContentResolver().openInputStream(imageUri)
);
picture.setImageBitmap(bitmap);
```

##### 相册
- 需要运行时权限WRITE_EXTERNAL_STORAGE
- 启动相册进行选择照片
```java
Intent intent = new Intent("android.intent.action.GET_CONTENT");
intent.setType("image/*");
startActivityForResult(intent,CHOOSE_PHOTO);
```
- 在onActivityResult中处理图片：从intent获得并解析uri、展示
```Kotlin
data.data?.let{ uri ->
	val bitmap = contentResolver
		.openFileDescriptor(uri, "r")?
		.use{
			BitmapFactory
				.decodeFileDescriptor(it.fileDescriptor)
		}
	imageView.setImageBitmap(bitmap)
}
```


##### MediaPlayer播放多媒体
- 指定文件、初始化
```java
File file = new File(Environment.getExternalStorageDirectory(), "music.mp3");
mediaPlayer.setDataSource(file.getPath());
mediaPlayer.prepare();
```
- 按钮点击事件：播放、暂停、停止
```java
switch (v.getId()){
  case R.id.play:
	  if (!mediaPlayer.isPlaying()){
		  mediaPlayer.start();
	  }
	  break;
  case R.id.pause:
	  if(mediaPlayer.isPlaying()){
		  mediaPlayer.pause();
	  }
	  break;
  case R.id.stop:
	  if(mediaPlayer.isPlaying()){
		  mediaPlayer.reset();
		  initMediaPlayer();
	  }
}
```
- activity被销毁时应该释放
  ```java
  @Override
  protected void onDestroy() {
      super.onDestroy();
      if(mediaPlayer!=null){
          mediaPlayer.stop();
          mediaPlayer.release();
      }
  }
  ```
- 播放视频：使用VideoView，代码与音频文件相似
  ```java
  // 播放初始化
  videoView.setVideoPath(file.getPath());
  
  // 播放释放
  videoView.suspend();
  ```



## 任务篇
##### Service
- Service概览
	- 即使程序被切换到后台或者用户打开了其他app，服务仍能够正常运行
	- 服务并不是运行在一个独立的进程当中，而是依赖创建服务所在的应用程序进程。当app的进程被杀掉时，服务也将结束
	- 服务不会自动开启线程，默认在主线程当中，有可能出现主线程被阻塞的情况
- 基本操作
	- 构建service：创建Service的子类
	- 启动service：使用Intent指向Service子类，通过调用`startService()`或`stopService()`
	- 内部结束service：在MyService的任意位置调用`stopSelf()`
    ```java
    Intent intent = new Intent(this, MyService.class);
    // 服务的启动
    startService(intent);
    // 服务的关闭
    stopService(intent);
    ```
	- 与Activity建立关联
		- 注意，远程service不可直接绑定（跨进程）
		- 创建Binder子类，并在其内部添加自己想要实现的方法
		- 在service内部的`onBind()`中返回Binder实例
		- 在activity中，通过`bindService()`或`unbindService()`进行绑定，并使用`ServiceConnection`与binder进行关联
	    - service可以和多个activity建立关联
      ```java
      // 返回binder
      @Override
      public IBinder onBind(Intent intent) {
        return new Binder(){
			public void startDownload(){
	            Log.d(TAG, "startDownload: ");
	        }
	        public int getProgress(){
	            Log.d(TAG, "getProgress: ");
	            return 0;
	        }
        }
      }
      ```
- Service生命周期
	- `onCreate()`只会在Service第一次被创建时调用
	- `onStartCommand()`：每次调用（启动）`startService()`时都会调用
	- `onBind()`：返回Binder实例
	- `onDestroy()`：只有在service未绑定且已停止的情况下才会被销毁
- 后台
	- Android中的后台是指完全不依赖UI。因此注意，Service默认运行在主线程中
	- 基于Service与Activity的生命周期不同，使用Service执行后台任务不需要担心Activity被摧毁
- 安卓优先级
	- 等级：
		- 关键：活跃应用，位于前台并与用户交互，包括前台应用和前台服务
		- 高：可见的Activity
		- 中：服务，系统会尝试使服务保持活动，甚至尝试重启粘性服务
		- 低：后台进程，空进程
	- 优先执行重要级高的应用
	- 三个原则
		1. 安卓将令与用户交互的所有应用平稳运行
		2. 安卓将令带有服务的活动运行，除非这么做违法了原则1
		3. 安卓会让所有app在后台运行，除非违反了上述两条原则
- 前台服务
	- 使用前台服务可以提高服务的系统优先级
	- 需要在service的`onCreate()`中创建一个表示正在运行的通知
	- 使用`startForeground()`启动
  ```java
  NotificationChannel channel = new NotificationChannel(
	  CHANNEL_ID, 
	  "Channel", 
	  NotificationManager.IMPORTANCE_HIGH);
  channel.setDescription("description");
  getSystemService(NotificationManager.class).createNotificationChannel(channel);
  
  Notification notification = new NotificationCompat.Builder(this, CHANNEL_ID)
          .setContentTitle("title")
          .setContentText("text")
          .setShowWhen(false)
          .setSmallIcon(R.drawable.ic_launcher)
          .build();
  startForeground(1,notification);
  ```
- 使用IntentService可以简单创建一个异步的、会自动停止的服务
	- IntentService同样需要在Manifest中声明
  ```java
  Intent intent= new Intent(this, MyIntentService.class);
  startService(intent);
  ```


##### 广播
- 广播分为标准广播和有序广播。仅有序广播可被截断
- 广播接收器分为静态注册（使用Manifest）和动态注册（代码中注册）
- 使用动态广播获取网络权限
	- 创建BroadcastReceiver的子类
	- 重写onReceive方法。这个方法会在接收到广播后被调用
	- 获取像网络信息这些较敏感数据需要在Manifest中声明权限
  ```xml
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
  ```
	- 在activity中
		- 设置IntentFilter使得能够选择并接收对应的广播
		- 动态注册、注销BroadcastReceiver。分别在onCreate和onDestroy中完成
  ```java
  IntentFilter filter = new IntentFilter();
  filter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
  NetworkChangeReceiver receiver = new NetworkChangeReceiver();
  registerReceiver(receiver, filter);
  ```
- 静态广播
	- 静态注册时，可以选择直接创建一个BroadcastReceiver子类，studio会自动添加模板代码，包括：
		- 在Manifest中自动注册。enabled表示启用，exported表示可接受本程序以外的广播
	- 直接创建BroadcastReceiver的子类
	- 在Manifest中设置intent-filter、声明权限
  ```xml
  <receiver
      android:name=".BootCompleteReceiver"
      android:enabled="true"
      android:exported="true"/>
  ```

- 发送标准广播：使用`sendBroadcast(intent)`发送
  ```java
  Intent intent = new Intent("my_action");
  
  // 安卓8.0+： 所有广播发送须带发送类的包名
  intent.setPackage("com.example.broadcastbestpractice");
  
  sendBroadcast(intent);
  ```  
  ```java
  sendOrderedBroadcast(intent, null);
  ```
	- 收者有序
  ```xml
  <intent-filter android:priority="100">
  </intent-filter>
  ```
	- 截断广播
  ```java
  abortBroadcast();
  ```
- 本地广播的发送与接收
	- 本地广播不能使用静态注册
  ```java
  private LocalBroadcastManager localBroadcastManager;
  
  // 获取实例
  localBroadcastManager = LocalBroadcastManager.getInstance(this);
  
  // 动态注册(也需要注销)
  intentFilter = new IntentFilter();
  intentFilter.addAction("my_action");
  localReceiver = new LocalReceiver();
  localBroadcastManager.registerReceiver(localReceiver,intentFilter);
  
  // 发送广播(8.0+需设置包名)
  Intent intent = new Intent("my_action");
  localBroadcastManager.sendBroadcast(intent);
  ```



## 应用数据篇
##### 文件
- 存储文件
	- 所有的文件都默认存储到`/data/data/<packeage name>/files`目录下
	- 两种文件的操作模式
	    - MODE_PRIVATE：当存在同名文件时，覆盖
	    - MODE_APPEND：当存在同名文件时，追加内容
  ```java
  public void save(String data){
        // 获取路径
        // 配置输送
        // 输送数据
      FileOutputStream out;
      BufferedWriter writer = null;
      try {
          out = openFileOutput("data", Context.MODE_PRIVATE);
          writer = new BufferedWriter(new OutputStreamWriter(out));
          writer.write(data);
      }catch (IOException e){
          e.printStackTrace();
      }finally {
          try {
              if(writer!=null)writer.close();
          }catch (IOException e){
              e.printStackTrace();
          }
      }
  }
  ```
- 读取数据
  ```java
  public String load(){
      FileInputStream in = null;
      BufferedReader reader = null;
      StringBuilder content = new StringBuilder();
      try{
          in = openFileInput("data");
          reader = new BufferedReader(new InputStreamReader(in));
          String line = "";
          while((line = reader.readLine())!=null){
              content.append(line);
          }
      }catch (IOException e){
          e.printStackTrace();
      }finally {
          if (reader!=null){
              try {
                  reader.close();
              }catch (IOException e){
                  e.printStackTrace();
              }
          }
      }
      return content.toString();
  }
  ```

##### SharedPreferences
- 使用键值对保存数据，存放在`/data/data/<packeage name>/shared_prefs/`目录下
- 三种获取SharedPreference对象的方法
	- Context类中的`getSharedPreference()`，第一个参数是文件名（仅该方法需要自己命名），第二个参数目前只能填MODE_PRIVATE
	- Activity类中的`getPreferences()`方法。会将当前Activity的类名作为文件名
	- PreferenceManager类中的`getDefaultSharedPreferences()`方法。会将包名作为文件名
- 存放SharedPreferences数据
  ```java
  SharedPreferences.Editor editor = getSharedPreferences("data", MODE_PRIVATE).edit();
  editor.putString("name", "Tom");
  editor.putInt("age",28);
  editor.apply();
  ```
- 获取SharedPreferences数据
  ```java
  SharedPreferences pref = getSharedPreferences("data", MODE_PRIVATE);
  String name = pref.getString("name", "");
  int age = pref.getInt("age", 0);
  ```

##### SQLite
- 数据库文件存放在`data/data/<package name>/databases`目录下
- 直接使用SQL操作数据库数据
	- 除了查询数据使用rawQuery() ，其余使用execSQL()


##### 设置页
- 配置设置页
	- 设置标签: `android:label="@string/action_settings"`
	- 定义父活动: `android:parentActivityName=".VisualizerActivity"`
	- 打开方式: `android:launchMode = "singleTop"`
- 创建Preference Fragment
	1. 在gradle中添加依赖`implementation 'com.android.support:preference-v7:25.1.0'`
	2. 创建一个XML偏好文件以定义这个fragment内包含什么
		- 在res中创建一个xml文件夹，在其中新建`(pref_visualizer.xml)`
	3. `(pref_visualizer.xml)`中的属性
		- PreferenceScreen设置页面
		- CheckBoxPreference选项框
		- defaultValue默认值
			- key键
			- summaryOn描述-开
			- summaryOff描述-关
			- title 标题
		- ListPreference多选框
		- EditTextPreference文本框
	4. 创建`PreferenceFragmentCompat`子类并重写`onCreatePreferences()`
		- 在这个方法中，使用`addPreferencesFromResource(R.xml.pref_visualizer)`插入xml文件
	5. 在`styles.xml`中添加一个主题
	   `<item name="preferenceTheme">@style/PreferenceThemeOverlay<item>`
- 响应设置
	1. 令activity实现`OnSharedPreferenceChangeListener`方法，并在重写的方法中调用修改函数或者单独修改
		- 第二个参数s是修改处的value，可以使用string类的`.equals()`方法来比对
	2. 将该监听器与sharedPreference对象连接（注册）：`.registerOnSharedPreferenceChangeListener(this)`
	3. 在应用关闭时`onDestroy()`取消侦听器注册：`.unregisterOnSharedPreferenceChangeListener(this)`


##### 运行时权限
- 普通权限和危险权限（安卓6.0+）
	- 普通权限只需在Manifest中申请
	- 危险权限还需要用户手动点击授权。这些权限被分了权限组，其中一个权限得到授权后，其他权限也得到了授权
	- 只要targetSdkVersion小于6.0，就不会存在运行时权限的概念
- 调出权限授权页面
```java
if(ContextCompat.checkSelfPermission(this, Manifest.permission.CALL_PHONE)!= PackageManager.PERMISSION_GRANTED){
	ActivityCompat.requestPermissions(
		this, 
		new String[]{Manifest.permission.CALL_PHONE},
		1);
}
```
- 获知是否授权成功
  ```java
  @Override
  public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
      super.onRequestPermissionsResult(requestCode, permissions, grantResults);
      switch (requestCode){
          case 1:
              if(grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED){
                  call();
              }else{
                  Toast.makeText(this, "no permission", Toast.LENGTH_SHORT).show();
              }
              break;
      }
  }
  ```


##### Content Provider
- Content Provider主要用于在不同的应用程序之间实现数据共享的功能，同时能够保证被访数据的安全性
- ContentResolver对应CRUD操作
	- 使用Uri表示表名参数
		- Uri由authority和path组成，authority用于区分不同程序，path用于区分不同的表
		- 在URI的后面加一个id表示访问对应id的数据
		- 通配符：\*，#都表示全部
  ```java
  content://com.example.app.provider/table/1
  content://com.example.app.provider/*
  content://com.example.app.provider/table/#
  ```
	- 解析URI
  ```java
  Uri uri = uri.parse("content://com.ex../table2");
  Cursor cursor = getContentResolver().query(
        uri,
        projection,
        // ...
  )
  ```
- 创建ContentProvider
	- 创建ContentProvider子类，并实现方法
    - `getType()`：根据传入的内容URI来返回相应的MIME类型
    - MIME类型：以路径结尾填dir，以id结尾填item
      ```java
      vnd.android.cursor.dir/vnd.com.example.databasetest.provider.book
      ```
	- 使用UriMatcher匹配内容，以获知是要调用什么数据
  ```java
  // 预设
  static {
      uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
      uriMatcher.addURI(AUTHORITY, "book", BOOK_DIR);
        // ...
  }
  
  // 检查
  switch (uriMatcher.match(uri)){
          case BOOK_DIR:
              return "vnd.android.cursor.dir/vnd.com.example.databasetest.provider.book";
      // ...
  }
  ```
	- 需要在Manifest中注册
		- 安卓10+引入了分区存储的概念（沙盒存储机制），以防止应用读取其他应用数据。为了使得provider能提供数据，还需要Manifest的application标签中添加：`android:requestLegacyExternalStorage="true"`
		- 安卓11+还需要在Manifest中声名要访问的软件包
  ```xml
  <provider
    android:name=".DatabaseProvider"
    android:authorities="com.example.databasetest.provider"
    android:enabled="true"
    android:exported="true"></provider>
  ```


## 位置信息篇

### #ShushMe：根据位置决定是否静音的app

- 使用GPS（Global Position System）获取的位置信息准确度高，但是耗电量大；而使用网络（Wifi和基地台）确定位置准确度不高，但是耗电量低，还能更快确定设备所在国家或地区
- 这里将使用Google Play Services（也可以叫GPS）获取地理位置信息。通过使用其API，可以获取最后已知信息、接收位置更新、纬度转地址等
- 某些Google Play Services API需要先创建一个与Google Play Services连接的客户端，并使用该连接与API通信。需要先获取密匙、创建客户端
- Google Maps API禁止保存地点信息（小于30天），地点id除外
- 安卓会跟踪设备相对于任何注册的地理围栏（Geofence）的位置。每台设备的地理围栏上限为100个，每个围栏都可以设置时限
- 使用Geofence的优势在于它被优化过（更多地使用了网络，有时使用GPS），减少了不断使用GPS查询的机会

##### *1 连接Google API Client

> 以调用API

- 在Manifest.xml中
  
  - 使用meta-data标签放入密匙
  
  ```xml
  <meta-data                    
  android:name="com.google.android.geo.API_KEY"
  android:value="insert-your-api-key-here" />
  ```
  
  - 添加用户权限（uses-permission）
    - android.permission.INTERNET
    - android.permission.ACCESS_FINE_LOCATION
  
  ```xml
  <uses-permission android:name="..." />
  ```

- 在build.gradle(app)中
  
  - 添加依赖
    - com.google.android.gms:play-services-places:9.8.0
    - com.google.android.gms:play-services-location:9.8.0

- 在布局中
  
  > 放置一个checkbox用于告知用户是否已经给了定位权限，若已给，则不可按
  
  - 放置checkbox

- 在MainActivity.java中
  
  - 实现ConnectionCallbacks、OnConnectionFailedListener
  - 创建一个GoogleApiClient
  
  ```java
  GoogleApiClient client = new GoogleApiClient.Builder(this)
          .addConnectionCallbacks(this)
          .addOnConnectionFailedListener(this)
          .addApi(LocationServices.API)
          .addApi(Places.GEO_DATA_API)
          .enableAutoManage(this, this)
          .build();
  ```
  
  - 重写以下三种方法，仅用于在console中输出现在的状态（logi）
    - onConnected()
    - onConnectionSuspended()
    - onConnectionFailed()
  - 重写onResume()。检查是否得到了定位权限，并据此设置checkbox状态
  
  ```java
  if (ActivityCompat.checkSelfPermission( MainActivity.this,
  android.Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
      locationPermissions.setChecked(false);
  } else {
      locationPermissions.setChecked(true);
      locationPermissions.setEnabled(false);
  }
  ```
  
  - checkbox响应点击事件：申请权限
  
  ```java
  public void onClicked(View view) {
    ActivityCompat.requestPermissions(MainActivity.this, new String[]{android.Manifest.permission.ACCESS_FINE_LOCATION}, PERMISSIONS_REQUEST_FINE_LOCATION);
  }
  ```

##### *2 打开地点选择器

- 在布局文件中
  
  - 添加一个按钮，用于选择位置

- 在MainActivity中
  
  - 添加位置button响应点击事件
    
    - 先检查是否拥有权限，若无，发出toast并返回
    - 构造PlacePicker的intent并打开activity。因为用户手机可能没有GooglePlayServices，所以这部分应该使用try-catch块。
    
    ```java
    try {
        PlacePicker.IntentBuilder builder = new PlacePicker.IntentBuilder();
        Intent i = builder.build(this);
        startActivityForResult(i, PLACE_PICKER_REQUEST);
    }catch(Exception e){
      // ...
    }
    ```
    
    - 使用`startActivityForResult()`以获取添加结果
  
  - 重写onActivityResult()以处理获得的结果。若添加成功，则应该记录该地点，提取数据并放到数据库中
  
  ```java
  Place place = PlacePicker.getPlace(this, data);
  if (place == null) {
      Log.i(TAG, "No place selected");
      return;
  }
  
  // Extract the place information from the API
  String placeName = place.getName().toString();
  String placeAddress = place.getAddress().toString();
  String placeID = place.getId();
  ```

[TOC]

##### *3 根据id获取地点

> 一是因为Google地图服务条款要求不得保存地点信息，只能保存id，二是因为id可能会变。所以能联网时就需要更新，使用id获取地点信息。除此之外，Google地图服务条款还要求添加"powered by google"

- 在MainActivity中
  - 每当连接上服务器时，在onConnected()中更新/获取位置信息。获取数据后，刷新adapter
- 在布局中
  - 添加一个名为"powered_by_google"的drawable，并使用src添加到widget中

##### *4 使用Geofence》触发静音事件

- 构造一个新类，用于管理geofence的存储
  
  - 写一个获取GeofenceingRequest、GeofencePendingIntent的方法
  - 写一个更新/添加方法
  - 写一个注册、注销所有geofence的方法
    - 通过使用`LocationServices.GeofencingApi.*`实现

- 构造一个新类，继承自BroadcastReceiver
  
  - 在重写方法onReceive中：设置静音、发送消息

- 在MainActivity中
  
  - 在连接上服务器并查询结果后(onResult())，更新一个Geofence列表
  - 请求响铃模式（安卓N及之后）权限（由button点击事件引起），在`onResume`中初始化这个button状态
  
  ```java
  Intent i = new Intent(android.provider.Settings.ACTION_NOTIFICATION_POLICY_ACCESS_SETTINGS);
  startActivity(intent);
  ```

- 在Manifest中
  
  - 添加一个receiver

```
测试位置应用》》
可以在模拟器中设置位置甚至是线路，但是因为是仿真的，缺乏网络提供的地理位置，所以地理位置未必会及时。打开地图类app可以使得手机强制实时更新地理位置
```

## 网络篇
##### WebView
- 在app内展示网页
	- 在布局文件中添加标签WebView
	- 在代码中配置
		- getSettings()：设置一些浏览器属性
		- setWebViewClient()：让网页在当前页面展示
		- loarUrl()：网址
  ```java
  WebView webView = findViewById(R.id.web_view);
  webView.getSettings().setJavaScriptEnabled(true);
  webView.setWebViewClient(new WebViewClient());
  webView.loadUrl("http://www.baidu.com");
  ```
- 声明权限INTERNET
	- application标签下添加属性设置
  ```xml
  android:usesCleartextTraffic="true"
  ```


##### HTTP协议访问网络
- HttpUrlConnection
	- 配置HttpUrlConnection
		- 设置HTTP请求所使用的方法：GET和POST
		- 设置连接超时：setConnectionTimeout()
		- 设置读写超时：setReadTimeout()
  ```java
  HttpURLConnection connection = null;
  BufferedReader reader = null;
  try {
      URL url = new URL("http://www.baidu.com");
      connection = (HttpURLConnection) url.openConnection();
      connection.setRequestMethod("GET");
      connection.setConnectTimeout(8000);
      connection.setReadTimeout(8000);
      InputStream in = connection.getInputStream();
  
      reader = new BufferedReader(new InputStreamReader(in));
      StringBuilder response = new StringBuilder();
      String line;
      while((line=reader.readLine())!=null){
          response.append(line);
      }
        // 自己写的展示方法
      showResponse(response.toString()); 
  }catch (Exception e){
      e.printStackTrace();
  }finally {
      if(reader!=null){
          try {
              reader.close();
          }catch (IOException e){
              e.printStackTrace();
          }
      }
      if (connection!=null){
          connection.disconnect();
      }
  }
  ```
- 使用OkHttp
	- 需添加依赖`implementation 'com.squareup.okhttp3:okhttp:3.4.1'`
  ```java
  try {
      OkHttpClient client = new OkHttpClient();
      Request request = new Request.Builder()
              .url("http://www.baidu.com")
              .build();
      Response response = client.newCall(request).execute();
      String responseData = response.body().string();
  
        // 自己写的展示方法
      showResponse(responseData);
  }catch (Exception e){
      e.printStackTrace();
  }
  ```


##### XML
- 使用Pull解析XML
  ```java
  try {
      XmlPullParserFactory factory = XmlPullParserFactory.newInstance();
      XmlPullParser xmlPullParser = factory.newPullParser();
      xmlPullParser.setInput(new StringReader(xmlData));
      int eventType = xmlPullParser.getEventType();
      String id = "";
      String name = "";
      String version = "";
      while(eventType != XmlPullParser.END_DOCUMENT){
          String nodeName = xmlPullParser.getName();
          switch (eventType){
              case XmlPullParser.START_TAG:
                  if ("id".equals(nodeName)){
                      id = xmlPullParser.nextText();
                  }else if("name".equals(nodeName)){
                      name = xmlPullParser.nextText();
                  }else if("version".equals(nodeName)){
                      version = xmlPullParser.nextText();
                  }
                  break;
              case XmlPullParser.END_TAG:
                  if("app".equals(nodeName)){
                      // ...
                  }
              default:
                  break;
          }
          eventType = xmlPullParser.next();
      }
  }
  ```
- 使用SAX解析xml
	- 初始化SAX
  ```java
  try {
      SAXParserFactory factory = SAXParserFactory.newInstance();
      XMLReader xmlReader = factory.newSAXParser().getXMLReader();
      ContentHandler handler = new ContentHandler();
      xmlReader.setContentHandler(handler);
      xmlReader.parse(new InputSource(new StringReader(xmlData)));
  }
  ```
	- 创建DefaultHandler的子类
  ```java
  public class ContentHandler extends DefaultHandler {
      private String nodeName;
      private StringBuilder id;
      private StringBuilder name;
      private StringBuilder version;
  
      @Override
      public void startDocument() throws SAXException {
          id = new StringBuilder();
          name = new StringBuilder();
          version = new StringBuilder();
      }
  
      @Override
      public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
          nodeName = localName;
      }
  
      @Override
      public void characters(char[] ch, int start, int length) throws SAXException {
          if ("id".equals(nodeName)){
              id.append(ch, start, length);
          }else if("name".equals(nodeName)){
              name.append(ch, start, length);
          }else if("version".equals(nodeName)){
              version.append(ch, start, length);
          }
      }
  
      @Override
      public void endElement(String uri, String localName, String qName) throws SAXException {
          if("app".equals(localName)){
              Log.d(TAG, "id: "+id.toString().trim());
              Log.d(TAG, "name: "+name.toString().trim());
              Log.d(TAG, "version: "+version.toString().trim());
              // clear
              id.setLength(0);
              name.setLength(0);
              version.setLength(0);
          }
      }
  
      @Override
      public void endDocument() throws SAXException {
          super.endDocument();
      }
  }
  ```

##### json
- json格式
	- 所有值皆被双引号`""`包
	- 键值一对，用冒号`：`分隔
	- 花括号用于保存对象`{}` 
	- 方括号用于保存数组`[]`
- json格式数据
  ```json
  [{"id":"5", "version":"5.5", "name":"Clash of Clans"},
  {"id":"6", "version":"7.0", "name":"Boom Beach"},
  {"id":"7", "version":"3.5", "name":"Clash Royale"}]
  ```
- 使用JSONObject解析json
  ```java
  try {
      JSONArray jsonArray = new JSONArray(jsonData);
      for (int i = 0; i < jsonArray.length(); i++) {
          JSONObject jsonObject = jsonArray.getJSONObject(i);
          String id = jsonObject.getString("id");
          String name = jsonObject.getString("name");
          String version = jsonObject.getString("version");
          Log.d(TAG, "id: "+id);
          Log.d(TAG, "name: "+name);
          Log.d(TAG, "version: "+version);
      }
  }catch (Exception e){
      e.printStackTrace();
  }
  ```
- 使用GSON解析json
  ```groovy
  implementation 'com.google.code.gson:gson:2.8.6'
  ```
	- 先指定类以放置数据（这里是app类）。变量名需与json字段同名
	- 配置GSON
  ```java
  Gson gson = new Gson();
  List<App> appList = gson.fromJson(jsonData, new TypeToken<List<App>>(){}.getType());
  for(App app: appList){
      Log.d(TAG, "id: "+app.getId());
      Log.d(TAG, "name: "+app.getName());
      Log.d(TAG, "version: "+app.getVersion());
  }
  ```



## 测试篇

### #Espresso框架：测试ui

- UI测试是一种设备化测试，需要在真机或模拟器上运行。设备化测试始终位于src/androidTest/java/...下

##### *1 测试点击事件

> 测试一次点击事件后（这里是修改咖啡订单的数量），view中产生的数据是否符合预期

- 在build.gradle中添加依赖性
  
  ```groovy
    androidTestImplementation 'androidx.test:runner:1.1.0'
    androidTestImplementation 'androidx.test:rules:1.1.0'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.1.0'
  ```

- 在src/androidTest/java/...下创建类用于测试(OrderActivityBasicTest)
  
  - 类的命名应该能让别人知道是测试类、测试的是什么
  - 在类上面添加注解`@RunWith(AndroidJUnit4.class)`
  - 在类里面创建一个rule，用于提供测试工具
  
  ```java
  @Rule
  public ActivityTestRule<OrderActivity> mActivityTestRule = new ActivityTestRule<>(OrderActivity.class);
  ```
  
  - 在类中创建测试方法
    - 使用onCheck()指定view
    - 测试中，点击按钮(perform)
    - 使用matches断言是否相等
  
  ```java
  @Test
  public void clickDecrementButton_ChangesQuantityAndCost() {
      // Click on decrement button
      onView((withId(R.id.decrement_button)))
              .perform(click());
  
      // Verify that the decrement button decreases the quantity by 1
      onView(withId(R.id.quantity_text_view)).check(matches(withText("0")));
  
      // Verify that the increment button also increases the total cost to $5.00
      onView(withId(R.id.cost_text_view)).check(matches(withText("$0.00")));
  
  }
  ```

- 最后，运行该测试：右键该测试类，选择Run

##### *2 测试adapterView

> 虽然 `onView()` 可以处理我们的 UI 中的大部分视图，但是 Espresso 在调用 AdapterView 小部件时，需要不同的方法调用。因为 AdapterView（例如 *ListView* 和 *GridView*）会从 Adapter 中动态加载数据，所以每次可能只有一部分内容可能会加载到当前的视图层级中。这意味着 `onView()` 可能无法找到所需的视图。 
> 
> 要解决这一问题，我们需要使用 `onData()`，它会将我们感兴趣的适配器项加载到屏幕上，然后再在上面进行操作

- 同样的位置创建一个新的类
  
  - 同样的JUnit、Rule、Test注解
  
  ```java
  @Test
  public void clickGridViewItem_OpensOrderActivity() {
      onData(anything()).inAdapterView(withId(R.id.tea_grid_view)).atPosition(1).perform( click());
      onView(withId(R.id.tea_name_text_view)).check(matches(withText(TEA_NAME)));
  ```

  }

```
- 测试RecyclerView

- 不同adapter，需使用onView指定RecyclerView的id
- 在perform中使用RecyclerViewActions
  - scroll类方法：滚动到该位置
  - action类方法：对该位置进行操作，如长按

- 需保证withText不会匹配到多项相同的

```java
onView(withId(R.id.fragment_now_list)).perform(RecyclerViewActions.actionOnItem(hasDescendant(withText("eat dinner")),longClick()));
```

- 区别
  
  ```java
  // finding data in views
  onView(ViewMatcher)
      .perform(ViewAction)
      .check(ViewAssertion)
  
  // finding data in adapterViews
  onData(ObjectMatcher)
      .DataOptions
      .perform(ViewAction)
      .check(ViewAssertion)
  
  // finding data in recyclerView
  onView(ViewMatcher)
    .perform(ViewAction)
    .check(ViewAssertion)
  ```

##### *3 测试Intent

> - Intent stub：当你的app使用intent打开某个列表并期待用户选择并返回数据时，可以使用Intent Stub替代这个打开列表并选择数据的过程，以测试app。（测试intent响应）
> - Intent Vertification：验证发送的intent包括了需要的数据（如：跳转到电话页面，需要传入电话号码等）

- Intent stub》同样的位置创建一个新的类
  
  - 同样的RunWith注解，但是使用不同的Rule注解
  
  ```java
  @Rule
  public IntentsTestRule<OrderSummaryActivity> mActivityRule = new IntentsTestRule<>(
          OrderSummaryActivity.class);
  ```
  
  - 挡住所有外部intent（使用Before注解）
  
  ```java
  @Before
  public void stubAllExternalIntents() {
      // By default Espresso Intents does not stub any Intents. Stubbing needs to be setup before
      // every test run. In this case all external Intents will be blocked.
      intending(not(isInternal())).respondWith(new ActivityResult(Activity.RESULT_OK, null));
  }
  ```
  
  - 需要先授予打电话的权限（安卓M+，同样使用Before注解）
  
  ```java
  @Before
  public void grantPhonePermission(){
    if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.M){
      getInstrumentation().getUiAutomation().executeShellCommand("pm grant "+getTargetContext().getPackageName()+" android.permission.CALL_PHONE");
    }
  }
  ```
  
  - 测试
  
  ```java
  @Test
  public void pickContactButton_click_SelectsPhoneNumber() {
      // Stub all Intents to ContactsActivity to return VALID_PHONE_NUMBER. Note that the Activity
      // is never launched and result is stubbed.
      intending(hasComponent(hasShortClassName(".ContactsActivity")))
              .respondWith(new ActivityResult(Activity.RESULT_OK,
                      ContactsActivity.createResultData(VALID_PHONE_NUMBER)));
  
      // Click the pick contact button.
      onView(withId(R.id.button_pick_contact)).perform(click());
  
      // Check that the number is displayed in the UI.
      onView(withId(R.id.edit_text_caller_number))
              .check(matches(withText(VALID_PHONE_NUMBER)));
  }
  ```

- Intent Vertification》同样的位置创建一个新的类
  
  - 仅测试不同：模拟了输入数据并确认、验证了intent
  
  ```java
  @Test
  public void typeNumber_ValidInput_InitiatesCall() {
      // Types a phone number into the dialer edit text field and presses the call button.
      onView(withId(R.id.edit_text_caller_number))
              .perform(typeText(VALID_PHONE_NUMBER), closeSoftKeyboard());
      onView(withId(R.id.button_call_number)).perform(click());
  
      // Verify that an intent to the dialer was sent with the correct action, phone
      // number and package. Think of Intents intended API as the equivalent to Mockito's verify.
      intended(allOf(
              hasAction(Intent.ACTION_CALL),
              hasData(INTENT_DATA_PHONE_NUMBER)));
  }
  ```

##### *4 测试使用闲置资源(Idling Resources)

- 闲置：消息队列中没有ui事件、默认的AsyncTask线程池中没有更多任务时

- 设置使用闲置资源：Espresso会等到app闲置时（后台线程没事做了）才执行下一步操作。假若app使用线程从网络上加载数据，则应该等到加载完成后才进行测试

- 初始代码中，需要设置对Idling Resources是否空闲的判断：实现接口IdlingResource。然后在Handler中，调用前将其设为false（非空），调用后设为true（空）

- 在build.gradle中添加注解
  
  ```groovy
  compile 'com.android.support.test.espresso:espresso-idling-resource:2.2.2'
  ```

- 同样的位置创建一个新的类
  
  - 同样的RunWith注解，但是使用不同的Rule注解
  
  ```java
  @Rule
      public ActivityTestRule<MenuActivity> mActivityTestRule =
              new ActivityTestRule<>(MenuActivity.class);
  ```
  
  - 因为需要在测试之前初始化IdlingResource，所以使用了`@Before`注解。Activity创建之后才会执行该注解中内容
  - 在`@After`中注销注册

```java
@Before
public void registerIdlingResource() {
    mIdlingResource = mActivityTestRule.getActivity().getIdlingResource();
    // To prove that the test fails, omit this call:
    Espresso.registerIdlingResources(mIdlingResource);
}

@After
public void unregisterIdlingResource() {
    if (mIdlingResource != null) {
        Espresso.unregisterIdlingResources(mIdlingResource);
    }
}
```

##### *5 拓展

- 还可以测试：Web、RecyclerView（与上述不同，不使用onData）
- 使用Espresso Teset Recoder
