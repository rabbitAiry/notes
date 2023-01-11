# 蓝牙
#### 配置蓝牙
- 在你的app中添加蓝牙权限
```xml
<uses-permission android:name="android.permission.BLUETOOTH" />  
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
```
- Android12引入了新的蓝牙权限
	- 当应用以Android12为目标平台才需要声明新权限
		- BLUETOOTH_SCAN权限：查找蓝牙设备
		- BLUETOOTH_ADVERTISE权限：当前设备可被检测
		- BLUETOOTH_CONNECT权限：与已配对的蓝牙设备通信
		- 以上权限是运行时权限，在应用中需要先得到用户批准
	```java
	List<String> list = new ArrayList<>();
	if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.S){
		list.add(Manifest.permission.BLUETOOTH_SCAN);
		list.add(Manifest.permission.BLUETOOTH_ADVERTISE);
		list.add(Manifest.permission.BLUETOOTH_CONNECT);
	}
	ActivityCompat.requestPermissions(
		this, list.toArray(new String[0]), 1);
	```
	- 对于旧版蓝牙相关权限，需令android:maxSdkVersion设为 30
	```xml
	<!-- 仅Android12+ -->
	<uses-permission android:name="android.permission.BLUETOOTH"                     android:maxSdkVersion="30" />
	<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"                     android:maxSdkVersion="30" />
	
	<uses-permission android:name="android.permission.BLUETOOTH_SCAN" />
	<uses-permission android:name="android.permission.BLUETOOTH_ADVERTISE" />   
	<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
	```
- 使用BluetoothAdapter进行设置
	- 需要两步检查：有蓝牙否，开启蓝牙否
	- BluetoothAdapter表示的是设备的蓝牙适配器
		- 如果无法获取（得到null），则表示该设备不支持蓝牙
	- 确保启动蓝牙
		- 在`onActivityResult()`中检查是否得到蓝牙开启
		- 也可以使用广播监听蓝牙变化
			- intent为ACTION_STATE_CHANGED
			- 该intent包含两个extra：EXTRA_STATE, EXTRA_PREVIOUS_STATE
			- extra可能的值为STATE_TURNING_ON, STATE_ON, STATE_TURNING_OFF, STATE_OFF
```kotlin
const val REQUEST_ENABLE_BT = 1

val bluetoothManager: BluetoothManager = getSystemService(
	BluetoothManager::class.java)  
val bluetoothAdapter: BluetoothAdapter? = bluetoothManager.getAdapter()  
if (bluetoothAdapter == null) {  
	// Device doesn't support Bluetooth  
}
if (bluetoothAdapter?.isEnabled == false) { 
	val enableBtIntent = Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE)  
	startActivityForResult(enableBtIntent, REQUEST_ENABLE_BT)  
}
```

#### 寻找蓝牙设备
- 概述
	- 使用BluetoothAdapter，可以找到附近的蓝牙设备，包括已配对设备或者发现的设备
	- 设备查找是一个对附近蓝牙可连接设备的搜索并询问其相关信息的过程
		- 附近的蓝牙设备只有在它愿意接收请求时才会响应
		- 如果一个设备时是能被发现的，他会通过分享设备名称、类以及MAC地址来回应请求。通过这些信息，可以进行设备之间初始化连接
		- 因为能被发现的设备会透露用户的位置信息，因此蓝牙搜索过程需要访问位置信息。Android8.0以上的设备应考虑使用CompanionDeviceManager API，该API可以在搜索蓝牙设备时不要求得到位置权限
	- 当首次完成与远程设备连接时，配对请求就会自动呈现给用户
		- 当一个设备配对成功后，设备的基本信息，包括设备名称、MAC地址就会被保存并且能通过蓝牙API得到
		- 对于已知MAC地址的远程设备，只要设备仍在范围内，无需再通过搜索，连接也会在任何时间得到初始化
		- 配对与连接有所不同
			- 配对意味着两台设备都知道对方的存在，有着共享的能力得到认证、互相建立加密连接
			- 连接意味着设备之间共享着能够互相传输数据的RFCOMM频道。当前的蓝牙API要求设备要在建立RFCOMM连接前完成配对。配对过程会在初始化加密连接时自动完成
- 查询配对的设备
	- 在执行设备搜索前，应该先查询已配对的设备中是否包含了目标设备
		- 使用`getBondedDevices()`查询已配对设备列表
		- 该方法会返回BluetoothDevice的Set
```kotlin
val pairedDevices: Set<BluetoothDevice>? = bluetoothAdapter?.bondedDevices
pairedDevices?.forEach { device -> 
	val deviceName = device.name
	val deviceHardwareAddress = device.address
}
```
- 查找设备
	- 调用`startDiscovery()`开始查找设备
		- 这个过程是异步的，并且会返回一个boolean值表示查找过程是否成功发起了
		- 查找过程通常包括了12秒的查询扫描，然后是对每个设备进行扫描已得到蓝牙名称
		- 该过程需要获取位置权限，且需要动态申请，否则会返回false
	```XML
	<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />  
	<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
	```
	```Kotlin
	// 申请
	val list = arrayOf(
			Manifest.permission.ACCESS_COARSE_LOCATION, 
			Manifest.permission.ACCESS_FINE_LOCATION)  
	ActivityCompat.requestPermissions(this, list, REQUEST_ENABLE_LOCATION)
	
	// 反馈
	override fun onRequestPermissionsResult(  
	    requestCode: Int,  
	    permissions: Array<out String>,  
	    grantResults: IntArray  
	) {  
	    super.onRequestPermissionsResult(requestCode, permissions, grantResults)  
	    when(requestCode){  
	        REQUEST_ENABLE_LOCATION -> {  
	            if(
		            grantResults.isNotEmpty() && 
		            grantResults[0]==PackageManager.PERMISSION_GRANTED){  
	                initSearch()  
	            }else{  
	                showToast("bluetooth init need location information")  
	                finish()  
	            }  
	        }  
	    }  
	}
	```
	- 要获得查找过的设备的详细信息，你的app应该注册一个BroadcastReceiver来得到ACTION_FOUND的intent
		- 系统会对每个找到的设备进行一次intent为ACTION_FOUND的播报
		- intent包括的extra有EXTRA_DEVICE和EXTRA_CLASS，分别对应BluetoothDevice对象和BluetoothClass对象
	```Kotlin
	private val receiver = object: BroadcastReceiver(){  
    override fun onReceive(context: Context, intent: Intent) {  
        when(intent.action!!){  
            BluetoothDevice.ACTION_FOUND ->{  
                val device:BluetoothDevice = intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE)!!  
                val deviceName = device.name  
                val deviceHardwareAddress = device.address  
                adapter.addItem(deviceName)  
            }  
        }  
    }  

	registerReceiver(receiver, IntentFilter(BluetoothDevice.ACTION_FOUND))
	unregisterReceiver(receiver)
}
	```
	- 要初始化与蓝牙设备的连接，可以通过对BluetoothDevice对象调用`getAddress()`得到对应的MAC地址
	- 执行蓝牙搜索会消耗大量蓝牙适配器的资源
		- 在你找到要连接的蓝牙设备后，应该要在进行连接前通过`cancelDiscovery()`停止搜索
		- 而且，不应该在连接设备之后再执行查找，因为查找过程会大幅度削弱现有连接的带宽
- 使设备可见（能被发现）
	- 基于Android的设备默认蓝牙都不能够被发现。要令设备在短暂时间内能被发现，用户可以通过系统设置，亦或是app要求用户允许被发现的能力
	- 要使设备能被发现，则应该调用`startActivityForResult(Intent, int)`以及使用ACTION_REQUEST_DISCOVERABLE的intent。这会无需打开设置地向系统发起请求允许设备可见
		- 默认情况下设备会在2分钟内可见
		- 可以自定义一个不同的可见时间——最长1小时——通过增加EXTRA_DISCOVERABLE_DURATION的extra
		- 如果将EXTRA_DISCOVERABLE_DURATION的extra的值设为0，设备将总是不可见。这种做法是不安全的，强烈不建议
	```kotlin
	const val REQUEST = 2
	val intent = Intent(
		BluetoothAdapter.ACTION_REQUEST_DISCOVERABLE).apply {  
		    putExtra(BluetoothAdapter.EXTRA_DISCOVERABLE_DURATION, 300)  
		}  
	startActivityForResult(intent, REQUEST)
	```
	- 发起设备可见请求后，会展示一个dialog。当用户同意后，设备会在对应时间内变得可见，并且app会收到`onActivityResult()`回调，result code的值等于持续的时间
	- 如果设备上的蓝牙还没有开启，那么使设备蓝牙可见会自动开启蓝牙
	- 设备会在分配的时间内持续可见。如果需要在设备可见状态改变时得到通知，则应该注册一个BroadcastReceiver，使用ACTION_SCAN_MODE_CHANGED的intent
		- 这个intent包括的extra有EXTRA_SCAN_MODE和EXTRA_PREVIOUS_SCAN_MODE，对应于提供的新旧两种扫描模式
		- extra可能对应的值包括了SCAN_MODE_CONNECTABLE_DISCOVERABLE、SCAN_MODE_CONNECTABLE、SCAN_MODE_NONE
	- 如果你正在初始化对远程设备的连接，那么你不需要使得设备可见。只有在你的app拥有一个用于接收连接的server socket时（远程设备必须在初始化连接之前能够找到其他设备），才需要使设备可见


#### 连接蓝牙设备
- 作为服务器端
	- 作为服务器端的设备需要持有
		- BluetoothServerSocket类
			- 监听蓝牙连接请求
			- 提供已经准备好的BluetoothSocket
	- 当得到BluetooSocket后，BluetoothServerSocket应该被回收
	- 设置ServerSocket的步骤
		- 通过调用`listenUsingRfcommWithServerRecord(String, UUID)`得到ServerSocket
			- 参数1是服务名称，系统会将该名称写进新的SDP
			- UUID是128位的字符串id，用于表示app的蓝牙服务
		- 调用`accept()`方法开始监听
			- 该方法时一个阻塞方法，不要在主线程中执行该方法
		- 调用`close()`关闭ServerSocket