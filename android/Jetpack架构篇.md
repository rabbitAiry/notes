## ViewModel
#### 概述
- ViewModel的出现是为了分担MVC模式下Activity的工作，并且能配置发生变化时保存状态
- 优势
	- 持久性
		- 使得状态和操作结果得以保存，尽管在Activity重建后也不需要重新获取数据
	- 作用域
		- 使用ViewModelStoreOwner对象对ViewModel进行实例化。ViewModel的生命周期与之相关
		- ViewModelStoreOwner的实现包括了ComponentActivity、Fragment、NavBackStackEntry等
	- SavedStateHandle
		- 允许了在配置变化时保存数据
		- 也允许了跨进程期间的重建，比如用户关掉app之后又打开
	- 访问业务逻辑
		- 大部分的业务逻辑都应该在data层中，但UI层也会持有部分业务逻辑
		- ViewModel适合在UI层中负责处理业务逻辑、负责处理事件并在需要时委托给架构中的其他层
- ViewModel与Jetpack Compose
	- 在JetpackCompose中，VewModel是向Composable提供存储状态的主力
	- 不可以将ViewModel的作用域设为composable
	- 建议将 ViewModel 用于屏幕级可组合项，即靠近从导航图的 activity、fragment 或目的地调用的根可组合项
	- 绝不应将 ViewModel 传递给其他可组合项，而是应当仅向它们传递所需的数据以及以参数形式执行所需逻辑的函数
- ViewModel与协程
	- 提供了viewModelScope成员（扩展属性）
- ViewModel的生命周期：直到对应对象被销毁
- 最佳实践
	- 不要将ViewModel传递给其他类
	- 不要持有view、lifecycle或者context（生命周期短于ViewModel），极易发生内存泄漏
	- 内部支持协程
- 与onSaveInstanceState的比较
	- ViewModel
		- 将数据存储在内存中，因此读取/写入时间快，且支持保存复杂对象
		- 在系统发起的进程终止后，不会继续存在
	- onSaveInstanceState
		- 将数据序列化到磁盘中，用时稍长且仅支持基元类型和简单的小对象
		- 在系统发起的进程终止后仍会存在，但是当Activity被关闭后不会继续存在


#### 使用
- 构造后即可使用
```Kotlin
class MainViewModel : ViewModel(){
	var counter = 0
}

class MainActivity: AppCompatActivity(){
	lateinit var vm: MainViewModel
	override fun onCreate(savedInstanceState: Bundle?){
		super.onCreate(savedInstanceState)
		setContentView(R.layout.activity_main)
		vm = ViewModelProvider(this)
			.get(MainViewModel::class.java)
		button.text = vm.counter.toString()
		button.setOnClickListener{
			vm.counter++
			button.text = vm.counter.toString()
		}
	}
}
```
- 当需要传递参数给ViewModel时，则需要提供ViewModel的工厂与类
```Kotlin
class MainViewModel(countReserved:Int) : ViewModel(){
    var counter = countReserved
}

class MainViewFactory(
	private val countReserved: Int
) : ViewModelProvider.Factory{
    override fun <T:ViewModel?> create(
	    modelClass: Class<T>): T {
	        return MainViewModel(countReserved) as T
    }
}
```
- 在Compose中
```Kotlin
import androidx.lifecycle.viewmodel.compose.viewModel

@Composable
fun WellnessScreen(
    modifier: Modifier = Modifier,
    wellnessViewModel: WellnessViewModel = viewModel()
) {
	Column(modifier = modifier) {
		StatefulCounter()
		WellnessTasksList(
			list = wellnessViewModel.tasks,
			onCloseTask = { task -> wellnessViewModel.remove(task) })
	}
}
```


##### 场景
- activity与fragment共享viewmodal的数据
	- fragment获取宿主Activity作用域的ViewModel实例：在`onViewCreated()`中
	- 从代码上看，fragment无需与activity的方法进行交互
        ```kotlin
        mModel = ViewModelProvider(requireActivity())
            .get(MyViewModel::class.java)
        ```



## Lifecycles
- 用于设计需要监听生命周期的组件
- 使用
	- 实现LifecycleObserver接口
	- 在Activity中作为监听器添加。lifecycle拥有一个LifecycleOwner实例
```kotlin
class Something : LifecycleObserver{
	@OnLifecycleEvent(Lifecycle.Event.ON_START)
	fun activityStart(){
		// ...
	}
	
	@OnLifecycleEvent(Lifecycle.Event.ON_STOP)
	fun activityStop(){
		// ...
	}
}

// Activity onCreate()
lifecycle.addObserver(SomeThing())
```
- 主动获取生命周期
```Kotlin
class Something(lc: Lifecycle) : LifecycleObserver{
	@OnLifecycleEvent(Lifecycle.Event.ON_START)
	fun activityStart(){
		Log.d(lc.currentState)
	}
}
```



## LiveData
#### 概述
- LiveData是一种响应式编程组件，能够感知生命周期，并只有在对应部件在活跃(active)时通知更新数据
- 优势
	- 确保UI与跟得上数据变化：观察者模式
	- 没有内存泄漏
	- 不会因为activity处于后台而崩溃
	- 不需要手动处理生命周期
	- UI得到得数据总是最新
	- 配置更改不影响
	- 共享LiveData
- 使用
```kotlin
class MainViewModel(num: Int) : ViewModel(){
	val counter = MutableLiveData<Int>()
	init{
		counter.value = num
	}
	
	fun plusOne(){
		counter.value = counter.value+1
	}
}

// Activity onCreate()
vm.counter.observe(this, Observer{
	text.text = it.toString()
})
```



## Room
#### Room架构
- Entity
```Kotlin
@Entity(tableName = "word_table")
class Word(@PrimaryKey @ColumnInfo(name = "word") val word: String)
```
- DAO
	- Data Access Object必须是抽象类或接口
	- DAO会在编译时检查SQL并将其与相关方法相连
	- 所有的操作都必须在单独的线程中执行
	- 支持协程（使用suspend修饰）
	- 可以与LiveData搭配
```kotlin
@Dao
interface WordDao {
    @Query("SELECT * from word_table ORDER BY word ASC")
    fun getAllWords(): List<Word>
	
	@Query("SELECT * from word_table ORDER BY word ASC")
	fun getAllWords2(): LiveData<List<Word>>
    
    @Insert
    suspend fun insert(word: Word)
    
    @Query("DELETE FROM word_table")
    fun deleteAll()
}
```
- Room
	- SQLite之上的数据库层
	- 默认情况下，Room不允许在主线程上发出查询请求
	- 当Room查询返回LiveData时，这些查询会在后台线程上自动异步运行
```kotlin
@Database(version = 2, entities = arrayOf(User::class, Book::class))
public abstract class AppDatabase : RoomDatabase() {
	abstract fun userDao():UserDao
	companion object{
		@Volatile
		private var instance : AppDatabase? = null
		
		fun getDatabase(context: Context) : AppDatabase{
			instance?.let{
				return it
			}
			synchronized(this){
				val temp = Room.databaseBuilder(
					context.applicationContext, 
					AppDatabase::class.java,
					"app_database"
				).build().apply { instance = this }
				return temp
			}
		}
	}
}
```
- 数据库的升级（在`.addMigrations(MIGRATION_1_2)`中）
```kotlin
val MIGRATION_1_2 = object:Migration(1,2){
  override fun migrate(database: SupportSQLiteDatabase) {
	  database.execSQL(
		  "create table Book (id integer primary key autoincrement not null, name text not null, pages integer not null)"
	  )
  }
}
```

- 执行数据库操作
  
  ```kotlin
  // 获取实例
  val userDao = AppDatabase.getDatabase(this).userDao()
  ```
  
  ```kotlin
  // 增
  val user1 = User("Tom","Brady",40)
  val user2 = User("Tom", "Hanks",63)
  addDataBtn.setOnClickListener{
      thread {
          user1.id = userDao.insertUser(user1)
          user2.id = userDao.insertUser(user2)
      }
  }
  ```
  
  ```kotlin
  // 改
  thread {
      user1.age = 42
      userDao.updateUser(user1)
  }
  ```

##### *5 WorkManager

- WorkManager适合用于处理一些要求定时执行的任务
  
  - 可以根据系统的版本自动选择底层如何实现后台任务
  - WorkManager注册的周期性任务不能保证一定会准时执行。系统会将触发时间接近的几个任务放在一起执行以减少CPU被唤醒次数，从而延迟续航
  - 可能在国产手机上不稳定

- WorkManager可处理三种类型的工作
  
  - 立即执行（甚至是加急执行）
  - 需要长时间运行（可能超过10“）
  - 可延期执行/定期执行

- 添加依赖
  
  ```groovy
  implementation 'androidx.work:work-runtime:2.2.0'
  ```

- WorkerManager使用步骤
  
  1. 定义一个后台任务，并实现具体的任务逻辑
  2. 配置该后台任务的运行和约束信息，并构建后台任务请求
  3. 将该后台任务请求传入WorkManager的enqueue()方法中。系统会在合适的时间运行

- 创建Worker子类
  
  - doWork()方法
    
    - 不会运行在主线程当中
    - 要求返回一个Result对象用于表示任务的运行结果。运行结果包括了success()、failure()和retry()
    
    ```kotlin
    class SimpleWorker(context:Context, params:WorkerParameters): Worker(context, params) {
      override fun doWork(): Result {
          Log.d("SimpleWorker", "doWork: ")
          return Result.success()
      }
    }
    ```

- 配置以及传入
  
  - OneTimeWorkRequest.Builder用于构建一次性的后台任务
  
  - PeriodcWorkRequest.Builder用于构建周期性的后台任务，要求运行时间不得小于15"
    
    ```kotlin
    val request = OneTimeWorkRequest.Builder(SimpleWorker::class.java).build()
    
    WorkManager.getInstance(this).enqueue(request)
    ```
