### #1 基本语法
- 表达式和语句 
	- 表达式：具有返回值，如kotlin中的条件语句
	- 语句：无返回值
- 条件语句：when表达式 
```kotlin
val b = when (num) {     
	in 0..9 -> {true}     
    else -> { false }
}
```
- in关键字 
```kotlin
//判断是否在区间内 
if (num in 1..9) {     
    print("ok") 
}  
//不在区间内 
if (num !in 1..9) {     
    print("no") 
}
//判断name是否在数组内
if (name in names) {     
    print("ok") 
}
```
-  for循环 
```kotlin
// 从1打印到100
for(x in 1..100)println(x)

// 从1打印到99
for(x in 1 until 100)println(x)

// 使用step来跳过：step必须是正数
for(x in 1..100 step 2)

// 使用downTo进行逆序
for(x in 15 downTo 1)println(x)
```
- 三种访问数组的形式 
```kotlin
// 使用foreach
for(item in array){}

// 使用访问数组下标的方式
for(index in array.indices){}

// 同时访问下标和元素值
for((index, item) in array.withIndex()){}
```
 -  导入：不区分导入的是类还是函数
```kotlin
import strings.lastChar
val c = "Kotlin".lastChar()
```
   

### #2 函数
-  局部函数 
```kotlin
fun outer(str: String) {
      fun inner(index: Int) {
          str.substring(0, index)
      }
      inner(2)
}
```
-  作用域函数Scope Functions
	- 域函数的作用仅仅是为了让代码更加简洁，只有5个：let、run、apply、also、with
	- 域函数之间的区别不大，顺手就行
| Function           | 指向对象 | 返回值         | 是扩展函数吗           | 使用场景                     |
| ------------------ | -------- | -------------- | ---------------------- | ---------------------------- |
| let                | it       | \\             | 是                     | 处理非空值、将表达式作为变量 |
| run                | this     | \\             | 如果用不到对象，则不是 | 配置对象属性并计算           |
| with(需要传入对象) | this     | \\             | 不是                   | 链式方法调用                 |
| apply              | this     | Context Object | 是                     | 配置对象属性                 |
| also               | it       | Context Object | 是                     | 在链式方法中调用             |
```Kotlin
// let, run
val text = "hello"
text.let{
	println("len:${it.length}")
}
text.run{
	println("len:$length")
}

// with
val list = mutableListOf("one", "two", "three")  
with(list) {  
    val firstItem = get(0)  
    val lastItem = last()  
    println("First item: $firstItem, last item: $lastItem")  
}

// also
fun getRandomInt(): Int {
    return Random.nextInt(100).also {
        println("value: $it")
    }
}

// apply
val adam = Person("Adam").apply { 
    age = 20
    city = "London"
}
```
- Java代码使用带有默认参数值的kotlin方法 
```kotlin
// 方法
@JvmOverloads fun myFun(str: String = ""){}

// 构造器
class Foo @JvmOverloads constructor(i:Int = 0){}
```
 

### #3 类和对象
-  定义一个类、创建对象 
	- 系统会为新建的对象分配内存，而构造函数负责将对象初始化
```kotlin
// 定义
class Dog(val name:String, var weight:Int, val breed:String){
    fun bark(){
        println(if(weight<20)"Yip!" else "Woof!")
    }
}

// 创建
var myDog = Dog("Fido", 70, "Mixed")
println(myDog.name)
myDog.bark()
```
-  构造器 
	-  使用空构造函数：类名后可以省略括号 
	-  使用初始语句块实现更复杂的操作 
	-  使用多个构造器 
		- 使用this委托给当前类的构造方法
		- 使用super委托给父类的构造方法
```kotlin
class MyButton: View{
    constructor(context: Context): this(context, null){
        // ...
    }
    constructor(c: Context, attr:AttributeSet): super(c, a){
        // ...
    }
}
```
- 私有的构造器 
```kotlin
class Secretive private constructor(){}
```
-  kotlin的对象默认必须在构造阶段初始化每一个变量 
	- 如果无法在类初始化时为属性分配初始值，则可以使用lateinit作为前缀
	- lateinit只能使用在var定义的属性上，且不可以用在以下类的属性上：Byte、Short、Int、Long、Double、Float、Char、Boolean。具体原因可以追溯到底层JVM运行代码的机制
-  自定义getter和setter 
	- 可以将getter称为acessor，setter称为mutator
	- 编译器会为所有未显式拥有getter和setter的属性定义getter和setter（setter不是必须的，对val而言不会添加）
	- 拥有getter的属性不强制需要初始化
	```kotlin
	class Dog(val name:String, var weight:Int, val breed:String){
	val weightInKgs: Double
		get() = weight/2.2
	}
	
	// 调用如下代码时，对象属性的getter会被自动调用
	myDog.weightInKgs
	```
	- setter方法中，value（常用的变量名）是准备赋值给属性的值，field是对属性的底层值得引用
	```kotlin
	class Dog(val name:String, weight_param:Int, breed_param:String){
	val weight = weight_param
		set(value){
			if(value>0)field = value
			/* 以下写法会陷入无限循环
			if(value>0)weight = value */
		}
	}
	
	// 调用如下代码时，对象属性的etter会被自动调用
	myDog.weight = 75
	```
- 顶层函数 
	- 用于替代java工具类中的函数（只作为方法提供的Util类）
	- 直接把函数放到代码文件的顶层，不用属于任何的类。这些函数依然是包内的成员
	- 原理
		- kotlin帮你把这些函数编译为静态类 
		- 使用`@file:JvmName("MyUtil")`并放在文件开头（package之前）可以为编译的静态类取代默认名字
-  顶层属性 
	- 默认仍会通过访问器暴露给Java使用
	- 使用const修饰一个常量（以static final的属性暴露给java）
-  扩展函数：强行将代码变成类的成员函数 
	- 不允许打破封装性，即扩展函数不能访问私有或受保护的成员
	- 原理：将这个类作为第一个参数，使得和静态函数一样（可从java的调用方式看出端倪） 
		- 因为不是真正类的一部分，所以扩展函数无法继承
	- 扩展函数是静态函数的一个高级语法糖
```kotlin
fun String.lastChar():Char = get(length-1)

println("Kotlin".lastChar())	// n
```
```java
// java
char c = MyStringUtil.lastChar("Java");
```
-  扩展属性 
   - 因为扩展属性没有支持字段和存储初始值的地方，所以必须设置getter且不能初始化
```kotlin
val String.lastChar: Char
	get() = get(length-1)
```
-  封装/kotlin中
	- 省略修饰符就是public属性
	- 没有“包私有”这种可见性
	- protected修饰的类只能被其子类访问
	- 提供了新的修饰符internal，表示只在模块内可见
- 使用open关键字声明父类及其能被覆盖的属性和方法 
	- 可以使用var覆盖val属性（只需加一个setter即可），但不能使用val覆盖var属性
	- 重写的成员（使用override关键字的成员）默认是开放的
    - 继承，但是不希望被重写：使用final声明
	- 抽象方法包含了open声明的属性
-  调用父类构造器 
    - 即使父类构造方法可能是无参的，但也需要显式地调用
```kotlin
open class Car(val make:String, val model:String){
    // ...
}

class ConvertibleCar(make_param:String, model_param:String):Car(make_param, model_param){
    // ...
}
```
- 被覆盖的方法和属性是open的，可以通过使用final关键字来阻止这一行为
```kotlin
open class Vehicle{
	open fun lowerTemperature(){
        // ...
    }
}

open class Car:Vehicle(){
    final override fun lowerTemperature(){
        // ...
    }
}
```


### #4 类的拓展
- 接口
	- 默认实现的方法 
		- 在Java8中，可以通过使用default关键字来声明
		- 在kotlin中，可以直接实现
	- 允许提供抽象属性
		- 可以理解为getter方法和setter方法
		- 数据可以在实现时获得，但是接口不保存数据（保存到实现类中了）
- 处理多个接口的同名方法
```kotlin
interface Clickable{
    fun showOff() = println("clickable")
}

interface Focusable{
    fun showOff() = println("focusable")
}

class Button : Clickable, Focusable{
    override fun showOff(){
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}
```
- 内部类
	- 静态内部类（嵌套类）：直接使用class声明
	- 成员内部类：使用`inner class`声明
	- 访问外部类：使用`this@外部类`的方式访问
- 密封类：定义受限的类继承结构
	- 就像是父类一样
```kotlin
sealed class Zoo{
	class Elephant():Zoo()
	class Giraffe():Zoo()
}
```
-  数据类 
	-  覆盖父类的equals方法、hashCode方法、toString方法、copy方法 
	-  equals方法中
		- 会自动比较构造器中包括的所有参数
		- 在类主体中定义的参数不会被比较 
	-  数据类定义了componentN方法，其中N表示被访问属性的编号，按照声明排序 
```kotlin
val r = Recipe("Chicken Banana", false)

// 1.
// 等价于 val title = r.title
val title = r.component1()

// 2.
// 解构数据对象：将r的属性值依次赋给这两个变量
val(title, vegetarian) = r
```
- object类（对象声明）：快速实现单例模式
	- 不允许提供构造方法
	- 可以直接通过`类名.xx`的方式访问单例的属性或者方法
	- kotlin的单例编译后的字段名为INSTANCE
- 伴生对象：为类提供静态成员
	- 使用`companion object`声明，从而提供通过容器类名称来访问对象方法和属性的能力
	- 可以访问容器类的静态构造方法
	- 伴生对象是一个声明在类中的普通对象，可以有名字、接口或者扩展函数
		- 编译后默认名字为Companion
- 委托
	- 将接口的实现委托到另一个对象
	- 原理不是继承，而是装饰器模式，这样即使委托的类方法不可以被继承也能实现效果
```kotlin
interface Base {
    fun printMessage()
    fun printMessageLine()
}

class BaseImpl(val x: Int) : Base {
    override fun printMessage() { print(x) }
    override fun printMessageLine() { println(x) }
}

class Derived(b: Base) : Base by b {
    override fun printMessage() { print("abc") }
}

fun main() {
    val b = BaseImpl(10)
    Derived(b).printMessage()
    Derived(b).printMessageLine()
}
```
- 委托属性
	- 属性的getter和setter得到委托
	- lazy属性
		- 默认：使用synchronize
		- LazyThreadSafetyMode.NONE：不保证线程安全
	```Kotlin
	val lazyValue: String by lazy {
		// 初始化过程
	    println("computed!")
	    "Hello"
	}
	
	fun main() {
	    println(lazyValue) // 第一次调用会进入初始化过程
	    println(lazyValue)
	}
	```

### #5 类型
- 基本数据类型
	- Kotlin中不区分基本数据类型和包装类型，除了泛型
		- 因为JVM实现泛型的方式，JVM不支持使用基本数据类型作为类型参数。所以Kotlin的泛型类必须使用类型的包装表示
	- 数字类型
		- Kotlin的Int类型会被编译成int类型，除了泛型
		- Kotlin不会自动将数字从一种类型转换成另一种，必须显式地转换
		- 在kotlin中使用java的基本数据类型数据时，数据不会变成平台类型，而是非空类型
- Any
	-  Any是所有类的父类
		- Any与Any?
		- 在底层，Any类型对应Object类型。Object类型会被看作Any的平台类型参数
		- Any只能使用toString、equals、hashCode方法，不能使用wait、hashCode方法（可以通过将值转为java.lang.Object）
	-  `==`操作符会调用equals方法 
		- 在Java中，\==操作是用于判断基本数据类型值是否相等以及引用地址是否相等，不能被重写
	-  `===`操作符不依赖于equals方法，与java\==相同 
		- 在尝试中发现kotlin的基本类型使用`===`比较的结果相等的，可能是因为版本更新而导致的
```kotlin
var num1 = 10
var num2 = 10
println(num1==num2)
println(num1===num2)
// z
var num3 = Integer(10)
var num4 = Integer(10)
println(num3==num4)
println(num3===num4)
// true true true false
```
- Unit类型
	- 相对于Java中的void
	- 可以作为泛型类型（不像Java需要使用Void）
	```kotlin
	interface Processor<T>{
		fun process():T
	}
	class NoResultProcessor: Processor<Unit>{
		override fun process(){
			// do stuff
		}
	}
	```
- Nothing类型
	- 没有任何值
	- 只有被当作函数返回值使用，或者当作泛型函数返回值的类型参数使用才有意义


### #6 空值、异常、类型转换
-  在类型后面加上问好表示该类型可以为空 
	- 不应该未声明类型信息就存放null，否则编译器会把该变量视为只能存放空值得变量 
	- Any类型也不包括空值 
```kotlin
var w: Wolf? = Wolf()
w = null
```
-  确保非空 
	- 可以使用if检查处理null，在if作用域内不会报错
	-  解决方法1：使用问号安全访问（链式） 
		- 只有在w不为空的情况下才调用eat方法，否则返回null
		```kotlin
		w?.eat()
		```
	-  解决方法2：使用let 
		- 只有在值不为空时执行代码块中的代码
		- 花括号不可省略
```kotlin
w?.let{
	it.eat()
}
```
- Elvis表达式
	- 表达式因为长得像Elvis猫王而被命名
	```kotlin
	// 错误示范
	if(w!=null) w.hunger else -1
	
	// 使用Elvis表达式
	w?.hunger ?:-1
	```
-  断言非空：值为空时抛出NullPointerException异常 
```kotlin
var x = w!!.hunger
```
- lateinit：延迟初始化
	- 如果属性被访问时没有被初始化，就会得到异常
- 从Java到Kotlin
	- 在Java中使用`@NotNull`和`@Nullable`注释能被转换为kotlin的非空特性
	- 平台类型：无法识别其非空性的Java类型，Kotlin会将这些类型作为平台类型
		- 平台类型的数据既可能为空，也可能为非空。编译器不会对这些类型的数据进行非空检查
		- 不可以在Kotlin中声明平台类型的数据
-  异常 
	- kotlin中throw、try、catch是一个表达式
	- 使用try-catch-finally捕获异常
	- kotlin不区分受控异常和非受控异常，不需要在方法中声明会抛出哪些异常（不需要使用throws）
-  类型判断 
	-  在类型判断之后，会进行自动转换 
```kotlin
fun foo(o: Any): Int {     
    if (o is String) return o.length
    if (o !is String) return 0     
    return 0
}
```
-  类型转换：使用`as`
	- 安全的类型转换：使用`as?`
		- 当无法进行类型转换时，返回类型null
```kotlin
val r: Roamable = Wolf()
if(r is Wolf){
    val wolf = r as Wolf
	wolf.eat()
}
```
- 类型检查
	- `is`类似于java中的`instanceof` 


### #7 集合
- 包含可空值的集合
	- 集合类型可以使用可空类型`?`
	- 使用`filterNotNull()`将可空类型集合转换为不可空类型的集合
- 只读集合与可变集合
	- 集合接口kotlin.collections.Collection仅提供了遍历、获取集合大小、查找操作，不包括添加或移除元素的方法
	- MutableCollection接口在Collection基础上，声明了修改接口元素的方法：增、删、清空
	- 只读集合并不总是线程安全的
		- 只读集合可能被不同的集合所引用
	- Kotlin集合和Java
		- 每一种Java集合接口在Kotlin中都有两种表示：只读与可变
		- 无论将哪一种集合接口传递给Java代码，在Java中都能够对集合进行修改
-  迭代 
	- 迭代与查询字符
	```kotlin
	for(c in 'A'..'F'){
	    val binary = Integer.toBinaryString(c.toInt())
	    binaryReps[c] = binary
	}
	```
	```kotlin
	var c = 'A'
	println(c in 'a'..'z')		// false
	println {"Kotlin" in "Java", "Scala")	// true
	```
	- 迭代map
	```kotlin
	val binaryReps = TreeMap<Char, String>()
	for((letter, binary) in binaryReps){
	    println("$letter = $binary")
	}
	```
	- 迭代集合：含下标
	```kotlin
	val list = arrayListOf("10","11","1001")
	for((index, element) in list.withIndex()){
	    println("$index: $element")
	}
	```


-  集合 
	- 初始化
	```kotlin
	val set = hashSetOf(1,7,53)
	val list = arrayListOf(1,7,53)
	val map = hashMapOf(1 to "one", 7 to "seven")
	val letters = Array(26){i -> ('a'+i).toString()}
	```
	-  集合类工具 
		- kotlin提供了集合功能
	```kotlin
	// 获取最后一个值
	list.last()
	// 获取最大值
	list.max()
	// 将数据转为数组
	list.toTypedArray()
	```
	- IntArray与Array\<Int>
		- 为了表示基本数据类型数组，Kotlin提供了诸如IntArray的类
		- Array\<Int>是一个包含装箱类型的数组

-  可变参数为初始化提供支持 
   - 使用vararg关键字表示
```kotlin
//java
public void selectCourse(String... strArray){}

//kotlin
fun listOf<T>(vararg values: T):List<T> {}
```

   - 使用\*可以直接展开 
      - java与kotlin的可变参数均是通过数组实现
      - 在kotlin中，可以以*的方式直接表示多个参数

```Kotli
fun main(args: Array<String>) {
    println(listOf("args:", *args))
}
```

-  中缀调用和解构声明 
   -  to函数 
```kotlin
1.to("one")		// 一般调用
1 to "one"		// 使用中缀符号调用
```
 

   -  需要允许使用中缀符号调用的函数需要使用infix来修饰 
```kotlin
infix fun Any.to(other: Any) = Pair(this, other)
```

   - 使用Pair来初始化两个变量：解构声明
```kotlin
val(number, name) = 1 to "one"
```


### #8 Lambda
- lambda语法
	- lambda与参数位置
		- 如果lambda表达式是函数调用的最后一个实参，则可以放到参数列表的括号之外
		- 如果lambda表达式是函数唯一的参数，则可以省略参数列表的括号
		- 在Java中，SAM接口可以直接作为lambda参数
		- 在kotlin中，SAM接口不被作为参数，而是只能使用`()->Unit`等的高阶函数形式
	- lambda表达式：被花括号`{}`包围
		- 参数和函数体都在lambda表达式内，使用`->`隔开
		- 参数类型如果可以被推到，则可以省略
		- 如果只有一个参数且类型可被推断，则未给参数命名的情况下，kotlin提供了it表示该参数
		- 与java不同，kotlin的lambda表达式可以访问非final变量，并且在lambda内部可以修改变量
			- 原理：为非final的变量使用包装类（或数组的其中一个元素），并将该包装类引用声明为final
	- 传递成员引用
		- 使用`类名::成员名`传递引用
		- 使用`::类名`作为构造方法的传递引用
	- 可以使用变量存储lambda：`val mf =  {p:Person -> p.age}` 
- 集合中的lambda表达
	- 例子
	```kotlin
	val people = listOf(Person("Alice",29),Person("Bob",31))
	```
	- 遍历
	``` kotlin
	people.forEach{println("$it")}
	```
	- 依据条件找到最大值
	``` kotlin
	// 以下两者都可以找到people中年龄最大值 
	println(people.maxBy{it.age})
	println(people.maxBy(Person::age))
	```

- with函数
	- 与let不同，with函数内部更像是内部类，可以通过this指向自身，通过`this@外部类`得到外部类
```kotlin
fun alphabet():String{
	val result = StringBuilder()
	for(letter in 'A' .. 'Z'){
		result.append(letter)
	}
	result.append("Now I know the alphabet!")
	return result.toString()
}
// ------ 使用with使得代码更简洁
fun alphabet() = with(StringBuilder()){
	for(letter in 'A' .. 'Z'){
		append(letter)
	}
	append("Now I know the alphabet!")
	toString()
}
```
- apply函数
```kotlin
fun alphabet() = StringBuilder().apply{
	for(letter in 'A' .. 'Z'){
		append(letter)
	}
	append("Now I know the alphabet!")
}.toString()
```

### #9 Lambda高阶函数
- 高阶函数
	- 类型：以(参数类型列表)->返回类型作为类型
		- 括号内为函数参数的类型，括号不可以被省略
		- 无返回内容时，返回类型为Unit
	- 原理
		- 通过FunctionN实现：每个函数类型的变量都是FunctionN接口的一个实现
	- 当java代码使用高阶函数参数时，可以直接当作匿名内部类使用
		- kotlin中声明返回Unit时，在Java中则应该返回`Unit.INSTANCE`

- 使用内联函数提高性能
	- lambda表达式会被编译成匿名类，其中的变量可能还需要创建一个新的对象，带来了额外的开销
	- 使用inline修饰符标记一个函数，使得编译器不会生成函数调用的代码，而是使用函数实现的真实代码替换（插入）每一次函数调用
```kotlin
inline fun <T> synchronized(lock: Lock, action:()->T):T{
	lock.lock()
	try{
		retrun action()
	}finally{
		lock.unlock()
	}
}
fun lock(l: Lock){
	println("Before sync")
	synchronized(l){
		println("Action")
	}
	println("After sync")
}
// ------ 编译结果
fun _foo_(l: Lock){
	println("Before sync")
	l.lock()
	try{
		println("Action")
	}finally{
		l.unlock()
	}
	println("After sync")
}
```

### #10 泛型
- 泛型函数和属性
	- `fun <T> List<T>.slice(indices: IntRange): List<T>`
- 声明泛型类
	```kotlin
	interface List<T>{
		fun get(index: Int):T
	}
	```
- 类型参数约束
	- 单个约束
	```kotlin
	// T是类型参数， Number是上界
	fun <T: Number> List<T>.sum():T
	```
	- 多个约束
	```kotlin
	// 要求类型必须既实现了CharSequence，也实现了Appendable
	fun <T> ensureTrailingPeriod(seq:T)
		where T:CharSequence, T:Appendable{
		if(!seq.endsWith('.')){
			seq.append('.')
		}
	}
	```
	- 未指定上界的类型形参
		- 默认使用Any?作为上界

- 运行时的泛型
	- 默认情况下，运行时中，泛型信息被擦除了
		- 不能够做到在运行时中检查List的具体类型
		```kotlin
		// 检查不合法（会报错）
		fun check(c: List){
			if(c is List<Int>){...}
		}
		// 检查合法
		fun check(c: Collection<Int>){
			if(c is List<Int>){...}
		}
		```
	- 使用内联函数可以优化这个问题
		```kotlin
		// 检查合法
		inline fun <reified T> check(c: Any){
			if(c is T){...}
		}
		```
	- 使用Class类型也可以解决这个问题
		```Kotlin
		inline fun <reified T: Activity> 
			Context.startActivity(){
				val intent = Intent(this, T::class.java)
				startActivity(intent)
		}
		```

- 泛型和子类型化
	- 协变
		- 使用out声明
		- 协变表示诸如List\<Child>是List\<Base>的子类的特性
	- 逆变：可以看作协变的镜像
		- 逆变表示诸如List\<Child>是List\<Base>的父类的特性
	- 星号投影
		- 使用\*代替类型参数，表示不知道泛型实参的任何信息


### #11 反射
- 反射概述
	- Kotlin反射API的推出是为了提供Java中不存在的概念，诸如属性和可空类型
	- Kotlin反射API并没有做到替代Java反射API
- 反射API
	- Kotlin反射API主要入口：KClass
		- 获取KClass实例
			- 使用`类::class`
			- 使用`实例.javaClass.kotlin`
	- Java反射API主要入口通过使用`类::class.java`得到
- KClass
	- 获取属性集合：`memberProperties`
	- 获取成员集合：`members`
		- 返回的是由KCallable实例的集合
		- 通过调用KCallable实例的call方法调用函数或者属性的getter
		- call方法的参数是所对应方法的参数。参数不符合时会抛出IllegalArgumentException
		- KFunction是KCallable的子类
			- KFunction末尾的数字（如果有）表示需要的参数数量
			- 可以通过invoke调用其方法


### #12 协程
- 协程是便于使用异步操作的框架
- 优势
	- 代码上简化并发步骤，将异步回调转换为顺序代码，消除了回调等带来的嵌套
	- 系统上减轻线程开销
- 创建协程：使用launch
	- launch中的代码块即为协程
```kotlin
launch(Dispatchers.IO){
	timeWastingJob()
}
```
- 切换线程：使用withContext
```kotlin
launch(Dispathers.Main){
	val image = withContext(Dispathcers.IO){
		getImage()
	}
	imView.setImageBitmap(image)
}
```
- 声明为suspend函数
	- suspend函数
		- 只能被suspend函数调用或者在直接的协程里被调用
		- 声明为suspend仅起到提醒作用
	- withContext()可以切换指定线程的suspend函数
	- 挂起指当前线程不再执行当前协程。直到当前协程在其他（包括自身）线程执行完毕后，当前线程才回来执行当前协程（挂起即临时切换线程并稍后恢复）
```kotlin
launch(Dispathers.Main){
	val image = getImage()
	imView.setImageBitmap(image)
}

suspend fun getImage(){
	withContext(Dispathcers.IO){
		getImageWithTime()
	}
}
```



- 概念
	- Job作业：可取消的工作单元
	- CoroutineScope协程域：用于创建新协程的函数
	- Dispatcher调度程序：确定协程将使用的线程
		- Main将始终在主线程上运行协程
		- Default（默认）、IO、Unconfined等会使用其他线程
	```Kotlin
	launch(Dispatchers.IO){ /**/ }
	```
- 使用launch
	- GlobalScope允许其中的任何协程运行
	- `launch()`函数会在协程域内创建协程
```Kotlin
fun main() {
    repeat(3) {
        GlobalScope.launch {
            println("Hi from ${Thread.currentThread()}")
        }
    }
}
```
- 将线程切换回到主线程
```Kotlin
GlobalScope.launch(Dispatchers.Main){
	val image = withContext(Dispatchers.IO){
		getImage(imageId)
	}
	avatarIv.setImageBitmap(image)
}
```
- 使用runBlocking
	- `runBlocking()`
		- 启动新协程并在新协程完成之前阻塞当前线程
		- 在典型的 Android 代码中并不常用
	- `async()`函数会返回 `Deferred` 类型的值
		- `Deferred` 是一个可取消的 `Job`，可以存储对未来值的引用（相对于callable的返回值future）
	- `await()`函数会等待 `Deferred` 的输出
	- suspend
		- 只要一个函数调用另一个 `suspend` 函数，那它也应是 `suspend` 函数
		- `getValue()` 不是在 `main()` 本身中调用的，而是在传递给 `runBlocking()` 的函数中调用的，所以不需要声明suspend
		- suspend只是表示函数拥有被挂起的能力，应该在协程域内进行
```Kotlin
val formatter = DateTimeFormatter.ISO_LOCAL_TIME
val time = { formatter.format(LocalDateTime.now()) }

suspend fun getValue(): Int {
    println("entering getValue() at ${time()}")
    delay(3000)
    println("leaving getValue() at ${time()}")
    return 2
}

// 依次获取num1, num2
fun main() {
    runBlocking {
        val num1 = getValue()
        val num2 = getValue()
        println("result of num1 + num2 is ${num1 + num2}")
    }
}

// 异步获取
fun main() {
    runBlocking {
        val num1 = async { getValue() }
        val num2 = async { getValue() }
        println("result of num1 + num2 is ${num1.await() + num2.await()}")
    }
}
```
- 使用delay
	- 协程函数，负责将协程挂起一定时间
	- 挂起协程不会阻塞线程，而是令其他协程继续运行从而充分利用线程资源
```Kotlin
fun main() = runBlocking { 
    launch { 
        delay(1000L) 
        println("World!")
    }
    println("Hello")
}
/** 
Hello
World!
*/
```
- 结构化并发
	- 概念含义：协程内包含子协程。结构化意味着需要掌控代码运行顺序
	- 外部的协程域只有在内部协程域结束时才能结束
	- join
		- 等待协程完成
```Kotlin
LaunchedEffect(playerOne, playerTwo) {  
    coroutineScope {  
        launch { playerOne.run() }  
        launch { playerTwo.run() }  
    }
    raceInProgress = false  
}
```
```Kotlin
// 或者
val job = launch {
	delay(1000L)
	println("World!")
}
println("Hello")
job.join() 
println("Done") 
// Hello World! Done
```
