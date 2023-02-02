> [compose codelab](https://developer.android.google.cn/courses/pathways/compose) 
> [compose文档](https://developer.android.google.cn/jetpack/compose/layouts/basics)

## 基础使用
#### 配置
- 使用 `setContent` 来定义布局
```Kotlin
override fun onCreate(savedInstanceState: Bundle?) {
	super.onCreate(savedInstanceState)
	setContent {
		BasicsCodelabTheme {
			Surface(
			  modifier = Modifier.fillMaxSize(),
			  color = MaterialTheme.colors.background
			) {
				Greeting("Android")
			}
		}
	}
}
```
- 预览
	- 配置预览效果
		- widthDp：小屏幕手机常见宽度320dp
		- uiMode：设置深色模式
```Kotlin
@Preview(showBackground = true,
		 uiMode = UI_MODE_NIGHT_YES,
		 name = "Text preview", 
		 widthDp = 320)
@Composable
fun DefaultPreview() {
    BasicsCodelabTheme {
        Greeting(name = "Android")
    }
}
```


## 常用组件
#### 文本和空间
- Modifier属性
	- [修饰符列表](https://developer.android.google.cn/jetpack/compose/modifiers-list) 
	- 常见
		- heightIn：描述高度，包括最大最小值
- Surface
	- shape
		- 使得该Composable内所有元素都绘制在该形状之内
- Divider：分割线
- Spacer：留白
- Text
```Kotlin
Text(
	text = name,
	style = MaterialTheme.typography.h4.copy(
		fontWeight = FontWeight.ExtraBold
	)
)
```
- TextField

#### 处理图片
- 使用painterResourse得到drawable中的图片
- clip：图片容器，比如圆形
- contentScale：图片缩放程度
	- fit：能够看到整张图片
	- fillBounds：拉伸图片使得占满容器
	- Crop：放大图片使得占满容器
```Kotlin
Image(
	painter = painterResource(R.drawable.ab1_inversions),
	contentDescription = null,
	contentScale = ContentScale.Crop,
	modifier = Modifier
		.size(88.dp)
		.clip(CircleShape)
)
```


## 状态
#### 重组
- 概念
	- Compose通过调用可组合函数将数据转换为界面，如果您的数据发生变化（状态发生更改），Compose 会使用新数据重新执行这些函数
	- Compose还会查看各个可组合项需要哪些数据，以便只需重组数据发生了变化的组件，而避免重组未受影响的组件
- Compose需要知道要跟踪的状态，以便在收到更新时安排重组
	- Compose 采用特殊的状态跟踪系统，可以为读取特定状态的任何可组合项安排重组
	- 使用 Compose 的 `State` 和 `MutableState` 类型让 Compose 能够观察到状态
- 使用remember记住可变状态
- 不同版本的同一可组合项：每个可组合项会有自己的状态版本
```Kotlin
val expanded = remember { mutableStateOf(false) }
// 或者 val expanded by remember {mutableStateOf(false)}
OutlinedButton(
	onClick = { expanded.value = !expanded.value },
) {
	Text(if (expanded.value) "Show less" else "Show more")
}
```
#### 状态提升
- 将事件让父级Composable管理，将状态向下传递

#### 保留状态
- 当activity发生配置变更时，所有状态都会丢失
- 使用rememberSaveable保存状态

#### 使用协程
- 在Composable内部使用`LaunchedEffect()`从而安全地使用协程
	- 协程会在`LaunchedEffect()`退出后取消
	- `LaunchedEffect()`的参数为键，当键发生改变时，协程会取消并重建
```Kotlin
LaunchedEffect(playerOne, playerTwo) {  
    coroutineScope {  
        launch { playerOne.run() }  // 在新协程中执行
        launch { playerTwo.run() }
    }
    raceInProgress = false  // 等待coroutineScope执行结束才能执行
}
```


## 布局
#### 布局基础
- 将状态转为界面元素
	- 组合Composition
	- 布局Layout
	- 绘图Drawing
- 标准布局元素
	- 提供排列界面元素的方式，包括Row、Column和Box
	- 设置布局中子项的位置
```Kotlin
@Composable
fun ArtistCard(artist: Artist) {
    Row(
        verticalAlignment = Alignment.CenterVertically,
        horizontalArrangement = Arrangement.End
    ) {
	    // 以下内容居中
        Image(/*...*/)
        Column { /*...*/ }
    }
}
```
- 布局模型
	- 每个节点将尺寸约束条件向下传递给子节点从而完成对子节点的测量
	- 然后确定自己的尺寸
	- 最后，确认子节点的放置位置
- 性能
	- Compose通过只测量一次子项来实现高性能
	- 如果布局需要多次测量，则需要用到Compose的固有特性测量（Intrinsic）
- 修饰符Modifier
	- 可以进行链式操作
	- 可以设置点击事件、padding、大小方式
```Kotlin
Column(
	Modifier
		.clickable(onClick = onClick)
		.padding(padding)
		.fillMaxWidth()
) {
	// ...
}
```
- 内容槽
	- 在某个UI组件中放入composable

#### Material布局
- Scaffold
	- 提供便捷的布局
	- 应用栏
		- 包括顶部栏topBar和底部栏bottomBar
	```Kotlin
	Scaffold(
	    bottomBar = {
	        BottomAppBar { /* Bottom app bar content */ }
	    }
	) {
	    // Screen content
	}	
	```
	- FAB
		- 使用 `floatingActionButtonPosition` 参数来调整水平位置
		- 使用 `isFloatingActionButtonDocked` 参数将 FAB 与底部应用栏重叠
		- 支持使用`cutoutShape` 参数设置FAB位置风格
	- SnackBar
		- 由ScaffoldState提供
	```Kotlin
	val scaffoldState = rememberScaffoldState()
	val scope = rememberCoroutineScope()
	Scaffold(
	    scaffoldState = scaffoldState,
	    floatingActionButton = {
	        ExtendedFloatingActionButton(
	            text = { Text("Show snackbar") },
	            onClick = {
	                scope.launch {
	                    scaffoldState.snackbarHostState
	                        .showSnackbar("Snackbar")
	                }
	            }
	        )
	    }
	) {
	    // Screen content
	}
	```
	- 抽屉式导航栏drawerContent


#### 自定义布局
- layout修饰符
```Kotlin
@Preview
@Composable
fun TextWithPaddingToBaselinePreview() {
// baseline到顶部为32dp
    MyApplicationTheme {
        Text("Hi there!", Modifier.firstBaselineToTop(32.dp))
    }
}

@Preview
@Composable
fun TextWithNormalPaddingPreview() {
// 文字顶部到顶部为32dp
    MyApplicationTheme {
        Text("Hi there!", Modifier.padding(top = 32.dp))
    }
}

// 自定义的方法
fun Modifier.firstBaselineToTop(
    firstBaselineToTop: Dp
) = layout { measurable, constraints ->
    // Measure the composable
    val placeable = measurable.measure(constraints)

    // Check the composable has a first baseline
    check(placeable[FirstBaseline] != AlignmentLine.Unspecified)
    val firstBaseline = placeable[FirstBaseline]

    // Height of the composable with padding - first baseline
    val placeableY = firstBaselineToTop.roundToPx() - firstBaseline
    val height = placeable.height + placeableY
    layout(placeable.width, height) { // 指定可组合项的尺寸
        // Where the composable gets placed
        placeable.placeRelative(0, placeableY)
    }
}
```
- 创建自定义布局
	- 使用Layou Composable，`Column` 和 `Row` 等所有较高级别的布局都使用 `Layout` 可组合项构建而成。
		- `measurables` 与 `layout` 修饰符类似，是需要测量的子项的列表
		- `constraints` 是来自父项的约束条件
	```Kotlin
	@Composable
	fun MyBasicColumn(
	    modifier: Modifier = Modifier,
	    content: @Composable () -> Unit
	) {
	    Layout(
	        modifier = modifier,
	        content = content
	    ) { measurables, constraints ->
	        // Don't constrain child views further, measure them with given constraints
	        // 测量每个子部件
	        val placeables = measurables.map{measurable ->
	            measurable.measure(constraints)
	        }
	
	        layout(constraints.maxWidth, constraints.maxHeight) {
	            var yPosition = 0
	            // 放置每个子部件
	            placeables.forEach { placeable ->
	                placeable.placeRelative(0, yPosition)
	                yPosition += placeable.height
	            }
	        }
	    }
	}
	```
- 布局方向
	- 通过更改LocalLayoutDirection来更改


#### 自适应布局
- 根据分配给可组合项用于执行渲染的实际宽度从而进行自适应布局
```Kotlin
@Composable
fun Card(/* ... */) {
    BoxWithConstraints {
        if (maxWidth < 400.dp) {
            Column {
                Image(/* ... */)
                Title(/* ... */)
            }
        } else {
            Row {
                Column {
                    Title(/* ... */)
                    Description(/* ... */)
                }
                Image(/* ... */)
            }
        }
    }
}

```

#### 滚动列表
- 为Column或Row添加滚动行为
```kotlin
val scrollState = rememberScrollState()
Column(Modifier.verticalScroll(scrollState)) {
	repeat(100) {
		Text("Item #$it")
	}
}
```
- LazyColumn或LazyRow的使用
	- 作用域中通过提供items元素提供渲染逻辑
	- 使用`Arrangement.spacedBy()`方法，在每个可组合子项之间添加固定间距
	- 使用`contentPadding`添加内边距
```Kotlin
@Composable
private fun Greetings(names: List<String> = List(1000) { "$it" } ) {
    LazyColumn(
		    horizontalArrangement = Arrangement.spacedBy(8.dp),
		    contentPadding = PaddingValues(horizontal = 16.dp),
			modifier = Modifier.padding(vertical = 4.dp)
		) {
        items(items = names) { name ->
            Greeting(name = name)
        }
    }
}
```
- LazyColumn或LazyRow的注意事项
	- `LazyColumn` 不会像 `RecyclerView` 一样回收其子级。它会在您滚动它时发出新的可组合项，并保持高效运行，因为与实例化 Android `Views` 相比，发出可组合项的成本相对较低
	- 如果您改变第 1 项内容，然后滚动到第 20 项内容，再返回到第 1 项内容，会发现第 1 项内容回到原来的状态。如果需要，您可以使用 `rememberSaveable` 保存此数据
- 可变列表
	- 创建一个可由Compose观察的MutableList实例：mutableStateList
```Kotlin
Column(modifier = modifier) {
	val list = remember { getWellnessTasks().toMutableStateList()}
	WellnessTasksList(list = list, onCloseTask = { task -> 
		list.remove(task) 
	})
}
```



## 动画
#### animate\*AsState
- 使用 `animate*AsState` 委托的任何动画都是可中断的，包括
	- animateColorAsState
	- animateDpAsState
```kotlin
val bgColor by animateColorAsState(
	if (tabPage==TabPage.Home) Purple100 else Green300
)
```
- 使用`animationSpec`参数指定动画的属性，包括
	- keyFrames
		- 以设置关键帧的形式设置动画
		- 使用如下方式设置关键帧
	- tween
	- spring
```Kotlin
// keyFrames
animation = keyframes {
   durationMillis = 1000
   0.7f at 500
   0.9f at 800
}

// how to use
var expanded by remember { mutableStateOf(false) }
val extraPadding by animateDpAsState(
	if (expanded) 48.dp else 0.dp, 
	animationSpec = spring(
		dampingRatio = Spring.DampingRatioMediumBouncy,
		stiffness = Spring.StiffnessLow
	)
)
Column(
	modifier = Modifier
		.weight(1f)
		.padding(bottom=Math.max(extraPadding, 0.dp)
)
```

#### 内容动画
- 自动根据内容大小发生动画改变
```Kotlin
var expanded by remember { mutableStateOf(false) }
Row(
	modifier = Modifier
		.padding(12.dp)
		.animateContentSize(
			animationSpec = spring(
				dampingRatio = Spring.DampingRatioMediumBouncy,
				stiffness = Spring.StiffnessLow
			)
		)
) {/* ... */}
```

#### 可见性变化动画
- 当某个部件动画效果为“不可见->可见”，那么可以使用AnimatedVisibility添加动画
	- 使用enter、exit参数调整动画效果，参数值如slideInVertically等
	- 为参数值设置initialOffsetX、animatioSpec等参数进一步调整动画效果
```kotlin
AnimatedVisibility(extended) {
    Text(
        text = stringResource(R.string.edit),
        modifier = Modifier
            .padding(start = 8.dp, top = 3.dp)
    )
}
```

#### Transition API
- 使用Transition API制作更复制的动画，它可以添加多个动画效果
```kotlin
val transition = updateTransition(
    tabPage,
    label = "Tab indicator"
)

val indicatorLeft by transition.animateDp(
    transitionSpec = {
        if(TabPage.Home isTransitioningTo TabPage.Work){
            spring(stiffness = Spring.StiffnessVeryLow)
        } else {
            spring(stiffness = Spring.StiffnessMedium)
        }
    },
    label = "Indicator left"
) { page ->
    tabPositions[page.ordinal].left
}
val indicatorRight by transition.animateDp(
    transitionSpec = {
        if(TabPage.Home isTransitioningTo TabPage.Work){
            spring(stiffness = Spring.StiffnessMedium)
        } else {
			spring(stiffness = Spring.StiffnessVeryLow)
        }
    },
    label = "Indicator right"
) { page ->
    tabPositions[page.ordinal].right
}
val color by transition.animateColor(
    label = "Border color"
) { page ->
    if (page == TabPage.Home) Purple700 else Green800
}
```

#### 无限期动画
- 使用`InfiniteTransition`无限期地为值添加动画效果
	- 通常，加载效果使用一些持续变化透明度的图形来表示
```kotlin
val infiniteTransition = rememberInfiniteTransition()
val alpha by infiniteTransition.animateFloat(
    initialValue = 0f,
    targetValue = 1f,
    animationSpec = infiniteRepeatable(
        animation = keyframes {
            durationMillis = 1000
            0.7f at 500
        },
        repeatMode = RepeatMode.Reverse
    )
)
```

#### 手势动画

## Navigation
#### 创建导航图
- 使用依赖
```groovy
dependencies{
	implementation "androidx.navigation:navigation-compose:2.5.0-rc01"
	// ...
}
```
- 设置Navigation的3个主要部分
	- NavController
		- 导航的核心组件
		- 通过调用 `rememberNavController()` 函数获取，并将其放置在可组合项层次结构的顶层
	- NavHost
		- 充当容器，负责显示导航图的当前目的地
	- NavGraph
		- 标出能够在其间进行导航的可组合目的地
		- 通过自定义的、唯一的字符串来表示不同的目的地
```kotlin
val navController = rememberNavController()
Scaffold(...) { innerPadding ->
	NavHost(
	    navController = navController,
	    startDestination = Overview.route,
	    modifier = Modifier.padding(innerPadding)
	) {
	    composable(route = Overview.route) {
	        Overview.screen()
	    }
	    composable(route = Accounts.route) {
	        Accounts.screen()
	    }
	    composable(route = Bills.route) {
	        Bills.screen()
	    }
	}
}
```

#### 连接、属性设置
- 触发导航
```kotlin
navController.navigate(newScreen.route)
```
- 栈顶不重复（SingleTop）
```Kotlin
navController.navigate(route) {
	launchSingleTop = true 
}
```
- 按下返回时，总是返回到导航图的起始目的地
```Kotlin
popUpTo(this@navigateSingleTopTo.graph.findStartDestination().id) {  
    saveState = true  
}
```
- 发生导航切换时，保存状态
```Kotlin
navController.navigate(route) {
	restoreState = true
}
```
- 推荐的做法
	- 如果存在多数或所有的导航切换都使用统一配置，那么可以通过扩展函数使得代码更简洁
```Kotlin
fun NavHostController.navigateSingleTopTo(route: String) =  
    this.navigate(route) {  
        popUpTo(this@navigateSingleTopTo.graph.findStartDestination().id) {  
            saveState = true  
        }  
        launchSingleTop = true  
        restoreState = true  
    }
```
- 获取堆栈中当前目的地的实时更新
```Kotlin
val nc = rememberNavController()
val cs by nc.currentBackStackEntryAsState()
// Fetch your currentDestination:
val currentDestination = cs?.destination
```
- 深层链接
	- 令该页面能够直接被快捷方式打开
	- 在Manifest中声明
	- 在对应的composable函数中使用deepLinks参数
```xml
<activity
    android:name=".RallyActivity"
    android:windowSoftInputMode="adjustResize"
    android:label="@string/app_name"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="rally" android:host="single_account" />
    </intent-filter>
</activity>
```
```Kotlin
composable(
	route = SingleAccount.routeWithArgs,    
	// ...    
	deepLinks = listOf(navDeepLink {
		uriPattern = "rally://${SingleAccount.route}/{${SingleAccount.accountTypeArg}}"
	})
)
```
- 测试深层链接
```sh
adb shell am start -d "rally://single_account/Checking" -a android.intent.action.VIEW
```


#### 为导航提供参数
- 设置参数提供的样式
	- 在Composable函数中提供arguments变量，并指定类型
```Kotlin
composable(
    route = "${account.route}/{${account.typeArg}}",
    arguments = listOf(
        navArgument(account.typeArg) { 
	        type = NavType.StringType 
	    }
    )
) {
    accountScreen()
}
```
- 访问参数
```Kotlin
composable(
	route = ...,
	arguments = ...
) { navBackStackEntry ->
	// Retrieve the passed argument
	val accountType = navBackStackEntry
		.arguments?.getString(account.typeArg)
}
```
- 传递参数
```Kotlin
// 好像什么都没发生一样
navController       
	.navigateSingleTopTo(
		"${account.route}/theType"
	)
```


## 迁移
#### View中使用Compose
- 使用ComposeView
```xml
<androidx.compose.ui.platform.ComposeView
	android:id="@+id/compose_view"
	android:layout_width="match_parent"
	android:layout_height="match_parent"/>
```
```kotlin
composeView.apply {
	// 在view的LifecycleOwner销毁时，处置Composition
	//（仅在Fragment中使用ComposeView时，才需要这样做，从而保证其拥有正确的生命周期）
	setViewCompositionStrategy(
		ViewCompositionStrategy
			.DisposeOnViewTreeLifecycleDestroyed
	)
	setContent {
		MaterialTheme {
			PlantDetailDescription(plantDetailViewModel)
		}
	}
}
```

#### Compose中使用View
- 示例：目前Compose不支持显示 HTML 格式的文本，所以需要构建一个TextView
```Kotlin
val description = "HTML<br><br>description"
val htmlDescription = remember(description) {
	HtmlCompat.fromHtml(description, HtmlCompat.FROM_HTML_MODE_COMPACT)
}
AndroidView(
	factory = { context ->
		TextView(context).apply {
			movementMethod = LinkMovementMethod.getInstance()
		}
	},
	update = {
		it.text = htmlDescription
	}
)
```