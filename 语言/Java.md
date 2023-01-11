## #1 控制程序流
- 注释文档Javadoc
	- Javadoc是用于提取注释（`/** ... */`），并输出一份HTML文件的工具
	- Javadoc只为public和protected成员进行文档注释
	- 能够在注释中嵌入html
	- 可以添加一些标签，如`@see`链接到其他文档、`@version`版本信息、`@param`参数

#### 操作符
- 几乎所有操作符都只能操作基本类型，但`=`、`==`、`!=`能够操作所有对象，表示所引用的对象是否相同。此外，String类还支持`+`、`+=`
- 正则表达式
	- 使用`?`表示可能有也可能没有的东西
	- 使用`+`表示有一个或多个要表达的东西
  ```java
  -?\\d+
  // \d表示一个数位,前面加上\表示转义
  // 表示前面可能有或没有负号，后面有一个或多个数位
  ```
- 位的运算：与或非、移位
- 类型转换：扩展转换会自动完成，窄化转换则需要强制
  ```java
  float f1 = 0.13;
  // 上述表达会报错，应该改为下述表达
  
  float f1 = 0.13F;
  float f2 = (float)0.13;
  ```
- 数值表达
  ```java
  1.39e-47f
  // 这表示 1.39 * 10^(-47)
  ```


#### 流程控制
- goto语句：continue和break后面加上标签，类似汇编语言的跳转
  ```java
  label1: 
  outer-iteration {
    inner-iteration {
      //... 
      break; // 1
      //... 
      continue; // 2 
      //...
      continue label1; 
      // 3 中断所有循环，回到label1，准备重新开始循环
      //...
      break label1; 
      // 4 中断所有循环，回到label1，不重新进入循环
    }
  }
  ```


#### 方法
- java中传递引用数据类型方式是值传递，无论是基本数据类型还是对象（而不是引用传递）
	- 值传递
		- 指在调用函数时将实际函数复制一份到函数中
		- 如果在函数中对参数进行修改，不会影响到实际参数
		- 对象的传递，复制的是其引用（地址值），并不是引用指向的存在于堆内存中的实际对象
			- 方法中可以改变对象的成员，但无法改变对象（如将对象置null，或更换对象、值）
	- 引用传递：将实际参数的地址直接传递到函数中
```java
// 值传递测试
class MyString{
	String s;
}

public static void main(String[] args) throws InterruptedException {
	Temp temp = new Temp();
	temp.s.s = "good";
	temp.setNull1(temp.s);
	System.out.println(temp.s.s);
	temp.setNull2(temp.s);
	System.out.println(temp.s.s);
}

void setNull1(MyString val){
  val = null;
}

void setNull2(MyString val){
  val.s = null;
}
/* output:
good

*/
```

#### Lambda
- lambda表达式不是匿名内部类
	- 两者有相似的地方，比如不能允许方法变量的更改
	- 由于lambda表达式通常很短，为表达式生成类会显得低效，因此lambda表达式不是匿名内部类的语法糖表达，而是函数式接口
- 函数式接口（Java8+）
	- 只有一个未实现的方法的接口称为函数式接口（SAM/单抽象方法接口）
	- 使用注解`@FunctionalInterface`进行标记
		- 不是必须的
		- Java编译器会检查该接口是否只有一个抽象方法
	- 常用函数式接口
		- Predicate<\T>：过滤（类型检查判断），方法为`test(T t)`返回boolean
		- Function<\T, R>：进行某种操作将T转R，方法为`apply(T t)`返回R
		- Consumer<\T>：对T进行操作，方法为`accept(T t)`，不返回
		- Supplier<\T>：工厂方法，方法为`get()`返回T
- 方法引用：lambda简写语法
	```java
	// 两种写法等价
	Supplier<String> s = Student::getCollegeName;
	Supplier<String> s = ()-> Student.getCollegeName();
	```

#### 函数式编程
- 函数式数据处理
	- （Java8+）通过Stream API对集合数据进行处理。注意与流类型无关系
	- `stream()`返回顺序流
	- `parallelStream()`返回并行流，背后可能有多个线程并行执行
- 以顺序流stream()为例
	- 传统的方法是命令式的，使用stream()的方法是声明式的
	- 链式结构，可以自由组合
	- 下面例子中的`filter`和`map()`都不会执行任何实际的操作，只是在构建操作的流水线，调用`collect()`才会触发实际的遍历执行
		- 因此如果在多个操作的自由组合中，也只会触发一次遍历
		- 将`filter`和`map()`称为中间操作(intermediate operation)
		- 将`collect()`称为终端操作(terminal operation)
	- 常见的中间操作
		- distinct方法：通过`eqauals()`过滤重复元素
			- 这是个有状态的处理
			- 有状态的处理：被过滤掉的元素不会传递给流水线的下一个操作
		- sorted方法：排序，有状态，需要排序后才将元素递给下一个操作
		- skip方法：跳过流中的n个元素，有状态
		- limit方法：限制流的长度为n，有状态
		- peek方法：遍历元素。参数列表提供了Consumer接口
		- mapToDouble/mapToInt等方法：将数据流转换为特定的流
			- 使用特定的流可能会提供更高效的方法
		- flatMap方法：将元素转换为流
	- 常见的终端操作
```java
// 过滤（筛选）
List<Student> perfects = students
	.stream()
	.filter(t->t.getScore()>90)
	.collect(Collectors.toList());

// 转换
List<String> names = students
	.stream()
	.map(Student::getName)
	.collect(Collectors.toList());
```



## #2 类与对象
#### 封装、继承、多态
- 封装：隐藏实现细节，提供简化接口（封装就是黑盒子的意思）
	- 访问权限
		- 未给定访问权限：仅包内的该类对象可访问（包访问权限、friendly）
		- public：皆可访问
		- private：确保访问不了（比如阻止别人直接访问某个特定的构造器）
		- protected：提供包访问权限，以及为有继承关系的类提供权限
		- 类不可以是private或者protected的，但内部类可以
- Effective建议
	- @13：使类和成员的可访问性最小化
	- @14：在公有域中使用访问方法而非共有域（比如getter）
- 继承：表示分类关系
	- 属性或者静态方法
		- 无论父类的属性或者静态方法被private修饰还是public修饰，同名都不冲突
		- 同名时，子类需要显式使用super访问父类同名属性或者静态方法
	- 方法
		- 重写
			- 方法被外部调用时，被调用的方法是成员所属类的方法
			- 重写的方法不能够降低父类方法的可见性
			- 如果有多个重名函数，调用会选择最匹配的函数
		- 重写原理：虚方法表
			- 在类加载时为每一个类创建一个表，记录该表的对象所有动态绑定的方法机器地址
			- 一个方法中只有一条记录，子类重写了父类的方法后只会保留子类的
			- 虚方法表提高了调用效率
	- 接口
		- 父类实现的接口会被子类继承
	- 构造器方法
		- 构造器参数
			- 如果构造器无参数，则其子类构造器不需要显式声明调用。顺序依旧是先执行父类的构造器
			- 如果构造器有参数，则子类构造器需要显式调用
			- 所有子类构造器都必须调用父类构造器
		- 子构造器允许降低可见性
- 多态：同一个行为，具有不同的表现形式
	- 向上转型：子类对象转换类型为父类对象
	- 动态绑定
		- 当调用被向上转型对象的方法时，能够调用对应子类的方法
		- 静态绑定在程序编译阶段即可决定，而动态绑定发生在程序运行时
```java
// 多态
class OP{
	private final void f(){}
	private void g(){}
}

class OP2 extends OP{
	public final void f(){}
	public void g(){}
	public static void main(String[] args){
		OP2 op2 = new OP2();
		op2.f();
		op2.g();
		OP op = op2;
		// op.f();  //可以上转型，但不能调用方法
		// op.g();
	}
}
```
- Effective建议
	- @16：复合优先于继承
		- 继承打破了封装性
		- “is-a”关系不容易找到
	- @17：要么为继承而设计，并提供文档说明，要么禁止继承
		- 文档需要描述实现的是给定的方法做了什么，而不是如何做到的
		- 确保子类能够符合预期
	- @18：接口优于抽象类
		- 现有的类可以很容易被更新，以实现新的接口
	- @19：接口只用于定义类型

#### 类与对象
- 为了方便理解和操作，高级语言使用数据类型这个概念。Java定义了8种数据类型，其他的数据类型都用类来表示
- 判断对象的类型：instanceof
	- 对于继承关系同样适用
``` java
System.out.println(Integer.valueOf(3) instanceof Integer);
```

#### 接口与抽象类
- 接口声明了一组能力的协议。自己并没有实现这些能力
- 接口的
	- 变量：默认被public static final修饰
	- 继承：同样使用extend表示
	- 静态方法（Java8）：使用static修饰
	- 默认方法（Java8）：使用default修饰。可以重写
- 抽象类可以有实例变量

#### 内部类（嵌套类）
- 使用内部类的理由：只与外部类有密切关系
	- 可以实现对外部的完全隐藏，有更好的封装性
	- 不是公用的工具类
- 内部类只是Java编译器的概念
	- 对于JVM并没有内部类这个概念
	- 内部类的内部实现皆是构建了一个新的类
	- 为了便于标记构建的类，命名方式是`外部类名@内部类名`。对于匿名内部类的内部类名则是数字1开始增加 
- 4种内部类
| 类         | 从内部类可访问                                                 | 从外部类可访问 | 从其他类可访问                      | 补充                                                                                                                                              |
| ---------- | -------------------------------------------------------------- | -------------- | ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| 静态内部类 | 仅静态变量和方法                                               | 直接点访问     | 需要new一个内部类。</br>可以直接new | 1. 静态指的是这个内部类是对于外部类的</br>2. 只有该外部类可以拥有静态变量和方法。是因为Java认为其他内部类用不上                                   |
| 成员内部类 | 直接点访问                                                     | 直接点访问     | 需要先有外部类实例，才可以new内部类 | 1. 与静态内部类相比，该内部类是对于外部类对象的。代码实现中，有指向外部类实例的变量</br>2. 外部类被加载时，该内部类不会被加载，所以不该有静态成员 | 
| 方法内部类 | 与方法可以访问的变量一样。</br>但是访问的方法变量只能是声明了final的变量 | 仅方法中       | 不可以                              |                                                                                                                                                   |
| 匿名内部类 | 与方法内部类一样                                               | 同上           | 同上                                | 可以直接new一个接口或者类，然后在大括号中实现方法或者重写方法。</br>原理是该内部类实现接口或者继承该类                                            |


#### final关键字
- final参数
	- 值或者引用所指向的对象不能被更改
	- 必须在定义处声明或是初始子句中声明
- final方法
	- 使用的原因
		- 禁止重写：避免修饰的类或方法被广泛使用导致后期无法进行更改
		- 效率
- final类：该类不能被继承
	- 在final类中标记final方法是多余的
- effecitive建议
	- @15：多使用final修饰，使可变性最小化
- final和private的关系
	- 类中所有private方法都隐含是final的
	- 如果某方法为private，那么该方法和继承无关，子类使用该方法名不能称之为重载

#### 枚举
- 使用枚举让代码更为简洁
	- 枚举变量的`toString()`或`name()`会返回其字面值
		- 枚举变量的`ordinal()`或枚举类型的`valueOf()`会返回枚举值在声明式的顺序
		- 枚举类型也实现了Java API中的Comparable接口，可以通过compareTo与其他枚举值进行比较，比较的是`ordinal()`的大小
	- 可以用于switch语句，switch语句内不需要前缀
	- 枚举变量可以使用equals或\==进行比较，结果一致
	- 枚举类型的`values()`返回枚举值的数组
```java
public enum Size{
	SMALL, MEDIUM, LARGE
}

Size size = Size.MEDIUM
```
- 枚举类原理
	- 枚举类是final的，其父类式为Enum\<T>
	- 枚举类的构造方法是私有的
- 拓展枚举的用法
	- 枚举值的定义要放在最上面，并于枚举值和代码之间使用分号隔开
	- 枚举很适合作为单例模式
```java
public enum Size{
	SMALL("S","小号"), 
	MEDIUM("M","中号"),
	LARGE("L","大号");
	
	private String abbr;
	private String title;
	private Size(String abbr, String title){
		this.abbr = abbr;
		this.title = title;
	}
	
	public String getAbbr(){
		return abbr;
	}
	
	public String getTitle(){
		return title;
	}
}
  ```


## #3 对象
#### 对象创建过程
- 在Java中，初始化和创建无法分离
- 对象创建过程
	- 分配内存
	- 对所有实例变量赋予默认值
	- 执行实例初始化
- 初始化的内容包括对象的初始化
    - 静态变量会在该类的任何对象创建之前、任何静态方法执行之前就完成初始化
    - 静态子句是多个静态初始化动作的集合
    - 如果用不到包含静态变量的类，则不会被初始化
    - 成员变量会在调用构造器前初始化
```java
class Cups { 
	static Cup c1; 
	static Cup c2; 
	
	// 静态子句
	static { 
		c1 = new Cup(1); 
		c2 = new Cup(2);
	}
	Cups() { 
		System.out.println("Cups()");
	}
}
/*
初始化顺序：①调用当前类的父类构造器 
	  ②当前类的成员变量初始化过程 
	  ③当前类的构造器
*/
```
- 构造器
	- 仅类中没有构造器时，编译器会自动创建一个无参构造器
	- 构造器中使用构造器：
	    - 方法名用this代替，而不是构造器名
	    - 只能调用一次，且调用必须在最起始处
```java
Flower() {
this("hi", 47); 
// 调用其他构造器

System.out.println("default constructor");
}
```
- Effective建议
	- @1：用静态工厂方法代替构造器（简单工厂模式）
		- 静态工厂方法的名称可以将对象特点描述得更清晰
		- 使得不必每次调用时都创建一个新对象变成可能
		- 使得返回其子类型的对象变成可能，提高了灵活性
		- 可以省略构造器中的泛型声明
	- @2：遇到多个构造器参数时要考虑用构建器builder模式
		- 可以使用setter替代，但不能保证线程安全
		- 使用builder可读性佳，可把调用串起来
	```java
	public class Flower{
	    private final int size;
	    private final int color;
	    private final boolean eatable;
	
	    public static class Builder{
	        // 必选参数
	        private final int size;
	        private final int color;
	        // 可选参数
	        private boolean eatable = false;
	
	        public Builder(int size, int color){
	            this.size = size;
	            this.color = color;
	        }
	        public Builder eatable(boolean eatable){
	            this.eatable = eatable;
	            return this;
	        }
	        public Flower build(){
	            return new Flower(this);
	        }
	    }
	
	    private Flower(Builder builder){
	        this.size = size;
	        this.color = color;
	        this.eatable = eatable;
	    }
	}
	
	// User part
	Flower flower = new Flower.Builder(24, RED)
	    .eatable(false).build();
	```
	- @3：使用单例
		- 单例作为public成员、使用静态工厂方法获取单例、使用枚举类声明
	- @4：工具类应该设置private构造器

#### 引用
- 引用和对象相当于遥控器和电视机，使用引用来操控对象
- 对象与内存
	- `Object obj`的语义会反映到栈内存中，作为一个reference类型数据出现
	- `new Object()`的语义会反映到堆内存中，形成一个存储Object类型的所有实例数据值的结构化内存
	- 此对象的类型数据（如对象类型、父类）的地址信息则存储在方法区中
  ```java
  // e.g
  Object obj = new Object()
  ```
- 四种引用Reference（JDK1.2+）
	- 强引用Strong
	    - GC不会回收（如果引用还在的话）
	- 软引用Soft
	    - 系统将要发生OOM异常之前，会先回收这些对象
	- 弱引用Weak 
		- GC工作时就会回收掉，无论当前内存是否足够
	- 虚引用Phantom
	    - 无法通过虚引用来取得一个对象实例
- 引用队列：用于保存被回收后的对象引用，由ReferenceQueue类表示。
```java
String stRef = "hi";
SoftReference<String> soRef = new SoftReference<String>("hi");
WeakReference<String> wRef = new WeakReference<String>("hi");
PhantomReference<String> pRef = new PhantomReference<String>("hi", null);

System.out.println(stRef+soRef.get()+wRef.get()+pRef.get());
/** 输出
hihihinull
*/
```

#### 运行时数据区
- 方法区Method Area/ HotSpot中的永久代(Perm Gen)
	- 线程共享。存放被JVM加载的类信息、常量、静态变量、即时编译器编译后的代码等数据
	- 异常状况
		- OutOfMemoryError：如果在堆中没有内存完成实例分配，并且堆也无法再扩展时
	- 内存回收的目标主要针对常量池的回收和对类型的卸载
- 堆Heap
	- 线程共享。存放对象和数组
	- JVM所管理的内存中最大的一块
		- 可以再被细分为新生代、老年代(Young Gen, Tenured Gen)，或者被分为Eden空间、From Survivor空间、To Survivor空间
		- 可以处于物理上不连续的内存空间中
	- 生命周期：在虚拟机启动时创建
	- 异常状况
		- OutOfMemoryError：如果在堆中没有内存完成实例分配，并且堆也无法再扩展时
- 本地方法栈Native Method Stack
	- 与虚拟机栈发挥作用相似，但是为Native方法服务
	- 甚至在有的虚拟机中与虚拟机栈合并了（HotSpot）
- Java虚拟机栈JVM Stack
	- 线程私有。描述的是Java方法执行的内存模型
	- 生命周期与线程相同
    - 每个方法被执行的时候都会同时创建一个栈帧
	    - 栈帧用于存储局部变量表（栈内存）、操作栈、动态链接、方法出口等信息
	    - 每个方法执行完成后，栈帧从虚拟机栈中出栈
	- 该区域的两种异常状况
	    - StackOverflowError：如果线程请求的栈深度大于虚拟机所允许的深度
	    - OutOfMemoryError：如果虚拟机栈需要扩展但无法申请到足够的内存时
- 程序计数器Program Counter Register（PCR）
	- 线程私有，指向当前线程所执行的字节码的行号
	- PCR是为了线程切换后能够恢复到正确的执行位置而设的，各条线程之间的PCR互不影响
	- 只能为Java方法计数，不能为native方法计数
- 直接内存
	- 不是虚拟机运行时数据区的一部分，但是同样会导致OutOfMemoryError异常出现
	- NIO（new Input/Output）类可以使用Native函数库直接分配堆内存，然后通过一个存储在Java堆里面的DirectByteBuffer对象作为这块内存的引用进行操作

#### 溢出
- Java堆溢出
	- 发生异常时分析思路
	    - 通过内存映像分析工具堆dump出来的堆存储快照进行内存分析
	    - 确认内存中对象是否是必须的
	    - 确认是内存泄漏还是内存溢出
  ```java
  static class OOMObject{}
  public static void main(String[] args) {
      List<OOMObject> list = new ArrayList<>();
      while(true){
          list.add(new OOMObject());
      }
  }
  ```

- 虚拟机栈和本地方法栈溢出（StackOverflowError）
  ```java
  private int stackLength = 1;
  public void stackLeak(){
      stackLength++;
      stackLeak();
  }
  
  public static void main(String[] args) {
      TestJVM test = new TestJVM();
      try{
          test.stackLeak();
      }catch(Throwable e){
          System.out.println("stack length:"+test.stackLength);
      }
  }
  ```

- 运行时常量池溢出
  ```java
  public static void main(String[] args) {
      List<String> list = new ArrayList<>();
      int i = 0;
      while(true){
          list.add(String.valueOf(i++).intern());
      }
  }
  ```
- 方法区溢出
- 本机直接内存溢出

#### 垃圾收集|@6、7
- 垃圾收集器(GC)
	- 如果一个对象失去了对其的引用，则有可能被回收。主要的数据变量（int）也类似
	- GC会在程序内存不足的情况下回收
	- 释放对象的引用的3种方式
  ```java
  // 1 方法结束时释放
  void go(){
          Life life = new Life();
  }

  // 2 引用被赋值到其他对象
  Life life = new Life();
  life = new Life();

  // 3 直接将引用设定为null
  Life life = new Life();
  life = null;
  ```
- GC如何判断对象是否存活
	- 引用计数法
	    - 给对象添加一个引用计数器，每当有一个地方引用它时，计数器值加1；引用失效时减1
	    - 计数器为0时，对象不可能再被使用
	    - 缺点：难以解决相互引用的对象的回收（该对象组合了另一个对象的同时，自己也被另一个对象所组合）
	- 根搜索法：Java使用该GC算法
	    - 以“GC Roots”对象作为起点，当一个对象无法到达GC Roots时视其为不可用
	    - 可作为GC Roots的对象：栈中引用的对象、方法区中的类静态属性，常量引用的对象
```java
// 根搜索算法的优势
class TestJVM{
    public Object instance = null;
    private static final int _1MB = 1024*1024;

    // 用于占内存
    private byte[] bigSize = new byte[2*_1MB];
    public static void main(String[] args) {
        TestJVM objA = new TestJVM();
        TestJVM objB = new TestJVM();
        objA.instance = objB;
        objB.instance = objA;
        objA = null;
        objB = null;
        System.gc();
    }
}
```

#### GC算法
- 标记
	- 要清除一个对象，至少需要两次标记过程
		- 第一次标记
		    - 标记没有与GC Roots相连的引用链的对象
		    - 标记后进行对象筛选以判断是该对象是否有必要执行`finalize()`方法
			    - 被筛选中的对象会加入F-Queue中
			    - 未被筛选中的对象会被直接回收
			- 不被筛选中的情况有如下
				- 没有覆盖`finalize()`方法
				- `finalize()`是否已被虚拟机调用。每个对象的`finalize()`方法都只会被调用一次
		- `finalize()`
		    - 对象由虚拟机自动建立的Finalizer线程去触发`finalize()`
		    - 虚拟机不会等待方法运行结束而仅仅选择触发这个方法是为了避免卡顿
		    - 如果在`finalize()`执行后能够重新与GC Roots相连，则不会被标记
- 标记-清除（Mark-Sweep）算法
	- 首先标记出需要回收的对象，在标记完成后统一回收所有被标记的对象
	- 缺点
	    - 标记和清除的效率都不高
	    - 标记清除之后会产生大量不连续的内存碎片
		    - 空间碎片太多会导致在之后的运行过程中遇到需分配较大对象时，因为无法找到足够的连续内存而不得不提前触发另一次的垃圾收集工作
- 复制算法：更适合新生代
	- 将可用内存按容量大小分为相等的两块，每次只使用其中一块
	- 当内存用完时，将活着的对象复制并整理到另一块内存空间上，然后清理掉当前空间所有内存
	- 在新生代中，每次GC后都有大批对象死去，只有少量存活。因此这种算法有优势，只是1：1分配空间有点浪费了
	- 改进
	    - 不按照1:1的比例来划分空间，而是划分为一块较大的Eden空间和两块较小的Survivor空间。每次使用Eden和其中一块Survivor（HotSpot中，Eden和Survivor之比为8:1） 
	    - 触发复制时，将所有对象（包括Eden空间中）移到另一块Survivor空间上
	    - 当Survivor空间不足时，直接使用老年代内存（担保）。
- 标记-整理算法：更适合老年代
	- 标记后，将所有存活对象整理好，然后直接清理剩余所有内存
- GC在方法区的回收
	- 在方法区中回收性价比较低
	- 回收废弃常量：没有被任何对象引用的常量会被回收
	- 回收无用的类判断条件有3个
	    - 该类的所有实例已被回收
	    - 该类的ClassLoader已被回收
	    - 该类对应的Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法
	- 在大量使用反射、动态代理、CGLib等bytecode框架的场景，以及动态生成JSP和OSGi这类频繁自定义ClassLoader的场景都需要虚拟机具备类的卸载功能，以保证永久代不会溢出
- effective建议
	- @6：清除过期的对象引用
		- 需要警惕内存泄漏的情形
		    - 类自己管理的内存（如，类的成员是数组，持有多个引用）。一旦元素被释放掉，则元素中包含的任意对象也应该被清空掉
		    - 缓存。使用弱引用（weak）来解决
		    - 监听器和回调
	- @7：避免使用终结方法finalize

#### 内存分配与回收策略
- 分代收集
	- 新生代使用复制算法，因为清理后对象少，复制的工作量不大
	- 老年代因为对象存活率高、没有额外空间对其进行分配担保，故使用标记-清理或标记-整理算法
- 对象优先在Eden分配
	- 当Eden区没有足够的空间时，虚拟机将发起一次Minor GC
	- 空间仍然不够时且Survivor区也容不下时，已经存在的对象会被迁移至老年代
	- 参数
		- PrintGCDetails：是否打印日志参数
		- SurvivorRatio：Eden区和Survivor区的比例
- 大对象直接进入老年代
	- 避免在新生代中发生大量的内存拷贝
	- 参数
		- PretenureSizeThreshold：设置大对象的阈值。只对Serial和ParNew收集器有效
- 长期存活的对象进入老年代
	- 虚拟机为了能够识别长期存活，给了每个对象定义一个年龄计数器
		- 每在一次Minor GC中存活，年龄就会增加1岁
		- 当年龄增加到一定程度时，就会晋升到老年代中
		- 如果相同年龄的所有对象大小总和大于Survivor空间的一半，大于等于该年龄的对象就可以直接进入老年代
	- 参数
		- MaxTenuringThreshold：成为老年代的年龄阈值
- 空间分配担保
	- 因为Survivor无法容纳的对象进入老年代，因此需要留意老年代的剩余空间大小是否能够容纳
	- 由于无法知道内存回收后有多少对象进入老年代，因此虚拟机只能够根据以往经验来预估（平均值）。若预估结果是无法容纳，则进行一次Full GC

#### HotSpot的垃圾收集器
- 并发与并行
	- 并行Parellel：多条GC线程并行工作，但此时用户线程仍处于等待状态
	- 并发Concurrent：用户线程与GC线程同时执行
- Serial/Serial Old收集器
	- 单线程收集器，在进行垃圾收集时必须暂停其他所有的工作线程直到收集结束（Stop The World）
	- Serial是新生代收集器，采用复制算法
	- Serial Old是老年代收集器，采用标记-整理算法
	- 特点
	    - 用户毫不知情的情况下暂停工作线程会带来恶劣体验，不过在桌面场景中，停顿时间在1秒以内
	    - 简单而高效，毕竟没有线程交互开销
- ParNew收集器
	- Serial收集器的多线程版本。新生代收集器，采用复制算法
	- 特点
		- 只有Serial和Par New能与CMS收集器并行工作，因此得到重用
- Parallel Scavenge/Parallel Old收集器
	- Parallel Scavenge是新生代收集器，采用复制算法，并行的多线程收集器
	- Parallel Old老年代收集器，使用多线程和标记-整理算法
	- 目的是达到一个可控制的吞吐量Throughput
		- 吞吐量是用户代码运行时间的占比，越高越好（用/(用+GC)）
	- 参数
		- 最大垃圾收集停顿时间MaxGCPauseMillis
	    - 吞吐量大小参数GCTimeRatio
	    - 启用GC的自适应调节策略（GC Ergonomics）
		    - 将参数UseAdaptiveSizePolicy打开后，虚拟机会动态调整停顿时间和吞吐量
- CMS收集器（Concurrent Mark Sweep）
	- 以获取最短回收停顿时间为目标的收集器
	- 老年代收集器，使用标记清除算法，整个过程分为4步
	    - 初始标记（需Stop The World）：仅标记GC Roots能直接关联的对象，速度很快
	    - 并发标记：判断对象是否不能抵达GC Roots
	    - 重新标记（需Stop The World）：防止因为用户程序导致标记产生变动而发生错误操作
	    - 并发清除
	- 特点
		- 并发收集、低停顿
	    - 对CPU资源非常敏感
	    - 无法处理浮动垃圾（并发过程中产生的垃圾）
	    - 为了提高速度，使用的是标记-清除算法而不是标记-整理算法从而带来的缺点
- G1收集器
	- 使用标记-整理算法，可以实现精确地控制停顿

#### 虚拟机性能监控工具
- Java提供的工具位于JDK/bin文件夹下
- 配置JVM
	- 可以在cmd中每次运行java文件时设置，也可以在eclipse中的Debug Configuration中设置
	- 基本参数
		- -Xms：堆的最小值
		- -Xmx：堆的最大值
		- -HeapDumpOnOutOfMemoryError：出现内存溢出时Dump出当前内存堆转储快照
		```
		-Xms100m -Xmx100m -XX:+UseSerialGC
		```
- 命令行工具
	- jps：虚拟机进程状况工具
	- jstat：虚拟机统计信息监视工具
	- jinfo：Java配置信息
	- jmap：Java内存映像工具
	- jhat：虚拟机堆转储快照分析工具
	- jstack：Java堆栈跟踪工具
- 可视化工具
	- JConsole
	- VisualVM


#### 序列化
- Java自带的序列化
	- 将内存中的Java对象持久保存到一个流中
	- 通过Serializable接口和ObjectInputStream(ObjectOutputStream)实现
	- 作用
		- 对象状态持久化
		- 跨进程中传递对象
	- 特点
		- 序列化后的形式比较大，占空间
		- 序列化的性能较低
		- 不能与其他语言交互
		- 格式是二进制的，不方便查看和修改
- 序列化操作
	1. 创建出FileOutputStream
	2. 创建ObjectOutputStream
	3. 写入对象
	4. 关闭ObjectOutputStream
  ```java
  // 1. 如果文件不存在，会自动被创建出来
  // 文件名的格式是什么都无所谓
  FileOutputStream fs = new FileOutputStream("MyGame.ser");
  // 2
  ObjectOutputStream os = new ObjectOutputStream(fs);
  // 3
  os.writeInt(3);
  os.writeObject(characterOne);
  os.writeObject(characterTwo);
  os.writeObject(characterThree);
  // 4
  os.close();
  ```
- 反序列化操作
```Java
  // 1
  FileInputStream fs = new FileInputStream("MyGame.ser");
  // 2
  ObjectInputStream os = new ObjectInputStream(fs);
  // 3 顺序与写入相同，超出次数则会抛出异常
  int size = os.readInt();
  GameCharacter[] objects = new GameCharacter[size];
  for(int i = 0; i<size; i++){
	  objects[i] = (GameCharacter)os.readObject();
  }
  // 4
  os.close();
```

- 序列化细节
	- 不会被保存的变量
		- 静态：静态变量属于类，不会被保存
		- transient变量
	- 涉及组合的序列化（假设对象a、b将要被序列化）
		- 如果a引用了对象c，那么c也会被保存
			- 特别地，如果c没有声明Serializeble会报错
		- 如果a和b都引用了对象c，那么c只会保存一份，反序列化后指向同一个对象
		- 即使a和b循环引用，反序列化后仍然能够保持引用关系
	- 自定义保存方式
		- 使用ObjectOutputStream对象的defaultWriteObject()或defaultReadObject()是自定义的基础，调用此方法后再进行自定义
	- 序列化实现原理：反射机制
- 序列化的版本控制
	- 背景：假设1.0的版本将对象序列化了，但在2.0的版本中把double成员变量改为int类型的成员变量
	- 使用serialVersionUID记录类的版本
		- java会依据结构自动生成该id。还原时，Java会自动对比
		- 这个参数不是继承的，而是靠程序员手动添加的
		```java
		private static final long serialVersionUID = 0L;
		```
- Effective的建议
	- 谨慎实现Serializable接口
		- 降低了“改变这个类实现”的灵活性
		- 增加了出现Bug和漏洞的可能性：反序列化机制没有通过构造器来创建
		- 随着类发行新的版本，相关测试负担增加了
	- 考虑使用自定义的序列化机制
	- 保护地编写readObject()
	- 对于实例控制，枚举类型优先于readResolve
	- 考虑使用序列化代理代替序列化实例



## #4 类
#### 包
- 创建CLASSPATH，存放`.class`文件
- import
	- 使用`*`可以引入该包下的所有直接类，不包括次级文件夹中的类
	- 静态导入`import static`：导入类的公开静态方法和成员
- jar包
	- 编译后的一个或多个Java class文件可以打包为一个jar文件
	- 在class文件的根目录使用jar工具打包：`jar -cvf 包名.jar 根文件夹` 

#### 类文件结构
- 类文件（字节码）
	- 虚拟机载入和执行字节码文件
	- 平台无关性的基石：Java通过为各个平台提供虚拟机使得代码能够"Write once, Run Anywhere"
	- 代码无关性的基石：其他语言也可以生成Class文件
- 类文件结构特点
	- 以8byte为基础单位的二进制流
		- 各个数据项目严格按照顺序紧凑地排列在类文件中，没有分隔符
		- 遇到占用8byte以上的数据项时，会按照高位在前的方式分割
		- 数据项的长度和顺序都被严格限定
	- 数据类型：类文件采用伪结构来存储，只有两种数据类型
		- 无符号数
			- 使用u1, u2, u3, u4表示1byte, 2byte, 3byte, 4byte的无符号数
		- 表
			- 以`_info`结尾
			- 描述有层次的复合结构的数据
- 类文件结构（按照顺序）
	- 魔数
		- 4byte，值为0xCAFEBABE（咖啡的名字）
		- 用于确定这个文件是否为一个能被JVM接收的类文件。每种文件的头部都会有魔数以标识文件类型，比如gif或jpeg
	- 版本号
		- 4byte，包括了2byte次版本号和2byte主版本号
		- 这是JDK的版本号，高版本JDK能向下兼容，但不能运行以后版本的类文件
		- 从45开始表示JDK1.0
	- 常量池
		- 长度不定
		- 常量池容量计数值
			- u2类型，表示常量池中的常量数量-1（计数从1开始）
		- 常量池包括了两大类常量
			- 字面量：Java语言层面的常量
			- 符号引用：包括了类和接口的全限定名、字段的名称和描述符、方法的名称和描述符
		- 常量池中的每一项常量都是一个表
			- 共有11种结构各不相同的表结构数据
			- 每一种表的第一位是u1的标志位表示表的类型标志
	- 访问标志
		- 2byte，用于识别类或接口层次的可访问性（被什么修饰）
	- 类索引、父类索引与接口索引集合
		- 类索引和父类索引是u2，接口索引集合是一组u2类型数据的集合
		- 类文件中这三项数据用于确定类的继承关系
	- 字段表集合
		- 描述接口或类中声明的变量（被什么修饰）
	- 方法表集合
		- 描述接口或类中声明的方法（被什么修饰）
		- 方法里面的代码存放在方法属性表集合中一个名为”Code“的属性里面
	- 属性表集合
- 分析类文件字节码的工具：javap


#### 反射
- 反射
	- 在运行时动态获取类型的信息，并根据这些信息创建对象、访问/修改成员、调用方法等
	- 反射的入口是Class类
	- 特点
		- 十分灵活
		- 编译器不能够在使用反射时帮助我们找到错误
		- 反射的性能要低一些
- Class类
	- Class类的对象表示的是java程序运行时的类和接口，或者枚举类、注解、数组和基本数据类型
	- 没有public的构造器，其对象会在使用ClassLoader的`defineClass()`方法时被JVM自动构造
	- Class类的对象提供了许多类或者接口的特征信息，信息来自于类文件（大部分）和类加载的环境。信息包括了：是否是内部类
	- 表示类信息java.lang.Class。Java中每个对象都有指向它所属类信息的引用
	- Class是一个泛型类
	- 获取Class对象的3种方法
		 ```java
		 Apple apple1 = new Apple();
		 Class<Apple> cinfo;
		 cinfo = apple1.getClass();
		 cinfo = Apple.class;
		 cinfo = Class.forName("com.test.Apple");
		 ```
	- Class类可以获取
		- 类的名称以及包信息
		- 字段信息Field（即静态变量与实例变量的信息）
		- 方法信息Method
		- 创建对象和构造方法
		- 类型检查和转换
		- 对应类的类型信息（如：枚举类、内部类等）
		- 对应类的声明信息（如：修饰符、父类、注解等）
		- 类的加载：`forName()`所做的就是类的加载
		- 对于数组类型，可以获取数组元素的类型、创建数组、返回数组长度等
		- 对于枚举类型，可以获取所有枚举常量
		- 泛型参数
	- 基本类型没有getClass方法，但是都有对应的Class对象，如`int.class` 
	- 数组被转换成Object而非Object\[]，是为了方便处理多种类型的数组 
- 反射示例
	- 强行获取实例（即使构造器是private的）
	```java
	Constructor<S1> constructor = S1.class.getDeclaredConstructor();
	constructor.setAccessible(true);
	S1 obj1 = constructor.newInstance();
	```
	- 序列化与反序列化（序列化为字符串形式）
```java
// 反射示例
public static void main(String[] args) {
	Student zhangsan = new Student("张三", 18, 89d);
	String str = toString(zhangsan);
	System.out.println(str);
	Student zhangshan2 = (Student)fromString(str);
	zhangshan2.selfDescription();
}

public static String toString(Object obj){
	try {
		StringBuilder builder = new StringBuilder();
		Class<?> cls = obj.getClass();
		builder.append(cls.getName()+" ");
		for (Field f : cls.getDeclaredFields()) {
			if(!f.isAccessible())f.setAccessible(true);
			builder
				.append(f.getName()+"="+f.get(obj).toString()+" ");
		}
		return builder.toString();
	} catch (Exception e) {
		throw new RuntimeException(e);
	}
}

// TestReflect$Student name=张三 age=18 score=89.0 
public static Object fromString(String str) {
	try {
		String[] items = str.split(" ");
		if(items.length<1)throw new IllegalAccessException(str);
		Class<?> cls = Class.forName(items[0]);
		Object obj = cls.newInstance();
		if(items.length>1){
			for(int i = 1; i<items.length; i++){
				String[] fv = items[i].split("=");
				if(fv.length!=2)
					throw new IllegalAccessException(items[i]);
				Field f = cls.getDeclaredField(fv[0]);
				if(!f.isAccessible())f.setAccessible(true);
				Class<?> type = f.getType();
				if(type==int.class)
					f.setInt(obj, Integer.parseInt(fv[1]));
				else if(type==byte.class)
					f.setByte(obj, Byte.parseByte(fv[1]));
				// 省略其他基础数据类型
				else{
					Constructor<?> ctor = type.getConstructor(
						new Class[]{String.class}
					);
					f.set(obj, ctor.newInstance(fv[1]));
				}
			}
		}
		return obj;
	} catch (Exception e) {
		throw new RuntimeException(e);
	}
}
```


#### 注解
- 内置注解
	-   `@Override`：表示该类重写了该方法，告知编译器这是重写方法
	-   `@Deprecated`：表示对应的代码已经过时了，通知程序员不应该使用它
	-   `@SuppressWarnings`：表示抑制Java的编译器警告，告诉编译器不要警告
-   声明式编程风格
	- 诸如简单声明Serializable，java就能够自动处理很多复杂的操作，亦或是使用注解或synchronized关键字降低了编程的难度
- 注解的创建
	```java
	@Target(ElementType.METHOD)
	@Retention(RetentionPolicy.SOURCE)
	public @interface Override{}
	```
	- `@Target`表示注解目标，可以使用`{...}`表示有多个目标，可选值有
		- TYPE：类、接口或者枚举声明
		- FIELD
		- METHOD
		- PARAMETER：方法列表参数
		- CONSTRUCTOR
		- LOCAL_VARIABLE：本地变量
		- MODULE：模块（Java9+）
		- 没有声明@Target：适用于所有类型
	- `@Retention`表示注解信息保留到什么时候
		- SOURCE：只在源代码中保留（不会被编译为字节码）
		- CLASS：保留到字节码文件中，JVM不一定会在内存中保留
		- RUNTIME：运行时中一直保留
		- 没有声明@Retention：默认为CLASS
- 注解的特性
	- 注解不能继承
	- 使用`@Inherited`可以让使用该注解的类的子类自带使用该注解
	- 注解中的参数
		- default表示默认值
	```java
	@Target(ElementType.METHOD)
	@Retention(RetentionPolicy.SOURCE)
	public @interface MyPassword{
		String roomPassword() default "123456"
	}

	@MyPassword("####0")
	int room;
	```
- 反射与注解：需要通过反射才能得到了用户填入注解的参数
	- `Annotation[][] annts = method.getParameterAnnotations();`

#### 动态代理
- 代理
	- 代理对象是对象的助理，用户与代理打交道而不直接接触实际对象
	- 代理的优点
		- 节省成本较高的实际对象的创建开销，按需延迟加载
		- 执行权限检查
		- 屏蔽网络差异和复杂性
	- 静态代理：代理对象在代码中存在
- 动态代理
	- 在运行时由Java动态创建代理
	- 场景：假如每个类都需要代理，为每个代理类都实现接口的工作量太烦琐
- Java自带的动态代理：使用Proxy类
	- 通过Proxy类的静态方法newProxyInstance创建代理对象
		- loader：传入被代理类所使用的ClassLoader
		- interfaces：代理类要实现的接口列表
		- h：InvocationHandler接口，只定义了一个方法invoke。对代理接口所有方法的调用都会转给该方法
		- 返回值可以强制转换为interfaces数组中的某个类型接口，不能转换为类
	```java
		public static Object newProxyInstance(
			ClassLoader loader, 
			Class<?>[] interfaces,
			InvocationHandler h
		);
	```
	- `newProxyInstance()`内幕
		- 通过`Proxy.getProxyClass()`创建代理类定义，类定义会被缓存
		- 获取代理类的构造方法，构造方法有一个InvocationHandler类型的参数
		- 创建InvocationHandler对象，创建代理类对象
	- 需要继承InvocationHandler以指定代理行为
		- InvocationHandler是一个接口，只有一个抽象方法。所有调用真实类的方法都会经过该抽象方法
		- 其invoke方法有三个参数
			- proxy：表示代理对象本身
			- method：只在被调用的方法
			- args：方法的参数
		- 在这个方法中进行真正的方法调用。注意调用真正方法时不可以使用带proxy的`method.invoke()`，否则会出现死循环
	```kotlin
	// 被代理类的入口
	// person对象是真实被调用的类
	class OwnerInvocationHandler(private val person: PersonBean) : InvocationHandler {  
	    override fun invoke(
		    proxy: Any?, 
		    method: Method?, 
		    args: Array<out Any>?): Any? {  
		        try {  
		            method?.let {  
		                if (it.name.startsWith("get")) 
			                return it.invoke(person, args)  
		                else if (it.name.equals("setHotOrNotRating")) 
			                throw IllegalAccessException()  
		                else if (it.name.startsWith("set")) 
			                return it.invoke(person, args)  
		            }  
		        } catch (e: InvocationTargetException) {  
		            e.printStackTrace()  
		        }  
		        return null  
	    }  
	}
	```
	- 局限性
		- 只能为接口创建代理
		- 返回的代理对象只能转换到某个接口类型
- cglib动态代理
	- 解决了java自带的动态代理的局限性
- 面向切面编程AOP（Aspect Oriented Programming）


#### 类加载
- 从源码到程序
	- 编译
		- 将源代码文件变成扩展名是`.class`的字节码
		- 由javac命令完成
	- 连接
		- 在JVM中解析`.class`文件，转换为机器能识别的二进制代码，然后运行
		- 由java命令完成
- 类加载时机
	- 在java中，类是动态加载的
		- 只有在第一次使用类的时候才会加载
		- 加载前会查看是否已经加载
- 类的生命周期
	- 加载
		- JVM的任务
			- 获取定义此类的二进制字节流
			- 通过字节流所代表的静态存储结构转化为方法区的运行时数据
			- 在Java堆中生成一个代表该类的Class对象
	- 连接
		- 验证
			- 检查类文件中的字节流包含的信息是否符合当前虚拟机的要求，并且不会危害虚拟机自身的安全
				- 不安全的行为包括：访问数组以外的数据、跳转到不存在的代码
				- Java代码是安全的语言，但是未必得到的类文件都是由Java编译器生成的，所以需要对类文件进行验证
			- 四个验证阶段
				- 文件格式验证
					- 检验类文件的格式
					- 比如：魔数、版本号等
				- 元数据验证
					- 对字节码描述的信息进行语义分析，以保证其描述的信息符合规范
					- 比如：是否有父类、是否继承了不允许被继承的类等
				- 字节码验证
					- 进行数据流和控制流分析/对类的方法体进行校验
					- 比如：类型转换是否有效、跳转指令是否不会跳转到方法体以外的字节码指令
				- 符号引用验证
					- 在“解析”阶段发生，此时虚拟机将符号引用转化为直接引用
					- 校验内容比如：符号引用中的类字段和方法是否能被访问、是否能够找到引用的对应类等
		- 准备
			- 为类变量分配内存并设置类变量初始值
				- 不包括实例变量
				- 声明final的静态变量直接得到代码中的赋值
				- 没有声明final的静态变量则只会被赋予初始值
				- 代码块中，a被赋值为123，b被赋值为0
				```java
				static final int a = 123;
				static int b = 123;
				```
				- 这些内存将在方法区中进行分配
		- 解析
			- 将常量池内的符号引用替换为直接引用
				- 符号引用
					- 以一组符号来描述所引用的目标
				- 直接引用
					- 可以直接指向目标的指针、相对偏移量或是句柄
					- 与虚拟机实现的内存布局相关，所引用的目标必定在内存中已存在
			- 虚拟机对解析结果进行缓存
				- 在运行时常量池中记录直接引用，并把常量标识为已解析状态
				- 这是为了避免多次解析
			- 四种引用的解析
				- 类和接口：对应于CONSTANT_Class_info
					- 如果解析结果不是数组，则将符号引用的全限定名传递给当前类的加载器去加载类
					- 如果解析结果是数组，则加载数组元素类型，然后生成数组对象
				- 字段：对应于CONSTANT_Fieldref_info
				- 类方法：对应于CONSTANT_Methodref_info
				- 接口方法：对应于CONSTANT_InterfaceMethodref_info
	- 初始化
		- 必须立即初始化的场景
			- 遇到new、getstatic、putstatic、invokestatic这四条指令
				- 并且访问的静态变量是当前类所拥有的，而不是父类
				- 并且不是常量（static final）
			- 使用反射时
			- 子类准备要被初始化时
			- 被调用main()的类
		- 根据代码进行初始化的过程，这些内容整合成了类构造器`<clinit>()`中
		- `<clinit>()`
			- 由编译器自动收集类中所有变量的赋值动作和静态语句块中的语句合并产生
			- 与对象构造器不同，不需要显示地调用父类构造器，因为虚拟机保证在此之前父类的`<clinit>()`已经执行完毕
			- 虚拟机会保证这个方法在多线程环境中被正确地加锁和同步
	- 使用
	- 卸载
- 类加载器ClassLoader
	- 负责将字节码文件加载到内存中
	- 输入的是完全限定的类名，输出的是Class对象
	- 不同的ClassLoader
		- 可以加载相同的类但互相隔离，互不影响
		- 加载同一个类后，两者不被看做是同一个类，instanceOf等机制失效
	- 程序在运行时的四个加载器（父子顺序）
		- 启动类加载器（Bootstrap ClassLoader）
			- 负责加载Java的基础类
			- 虚拟机自身的一部分，无法被Java程序所使用
		- 扩展类加载器（Extension ClassLoader）
			- 负责加载Java的一些扩展类
			- 在Java9，该加载器被取消了，且使用平台类加载器（Platform ClassLoader）进行替代
		- 应用程序加载器（Application ClassLoader）
			- 负责加载应用程序的类
			- 系统类加载器（System ClassLoader）：默认的应用程序加载器
		- 自定义加载器（User ClassLoader）
	- 双亲委派模型
		- 当加载一个未被加载的类时，先让父加载器执行加载
		- 在父加载器没有加载成功的前提下，自己尝试加载
		- 双亲委派模型避免了Java类库被覆盖的问题，被Java所推荐而不是强制
    - 自定义ClassLoader的作用
        - 实现热部署：在不重启Java程序的情况下，动态替换类的实现
        - 应用的模块化和相互隔离
        - 从不同地方灵活加载：可以从网络中加载字节码文件
- ClassLoader内幕
	- 获取ClassLoader
		- Class对象都能够获取加载自己的加载器，通过使用`getClassLoader()`
		- 如果由BootStrap ClassLoader加载，则返回null
	- ClassLoader的方法
		- 获取父加载器：`getParent()`
		- 获取默认的系统类加载器：`getSystemClassLoader()`
		- 加载类：`loadClass(String name)`
			- 与Class的forName()的区别：前者不会执行类的初始化代码（`<clint>()`部分）
- 自定义类加载器：通过继承ClassLoader，重写findClass()实现自定义
	- `findClass()`：找到类文件字节码的字节形式后，使用defineClass()转换为Class对象
- 热部署
	- 单例模式获取实例，并令获取单例的方法作如下动作
		- 实例化一个自定义类加载器
		- 通过类加载器获取类，再通过反射获取实例
	- 使用独立线程，通过记录类文件的最后修改时间，并时刻跟踪从而得知文件是否发生了更改。若发生更改，则重新调用单例模式中的方法修改单例
	- 因为总不是同一个类加载器实例，使得可以成功修改单例



## #5 常见类型
#### Object类
- `toString()`：返回对象的文本描述
	- 默认的文本描述，如`Point@76f9aa66`。@之前是类名，之后是该对象的哈希值十六进制表示
- `clone()`：克隆对象
	- 拷贝
		- 引用拷贝：比如`Apple a2 = a1`
		- 浅拷贝
			- 使用`a2 = (Apple)a1.clone()`
			- 浅拷贝创建了新对象
			- 浅拷贝的使用需要对象实现了Cloneable接口，并重写clone()方法（重写的方法中只需要调用clone()即可）
			- 浅拷贝后的对象中，成员引用指向与原对象相同
		- 深拷贝
			- 深拷贝后的对象后，成员引用指向对象也得到了拷贝
			- 深拷贝需要在重写clone()方法时，同时对每一个成员进行拷贝
- effective建议
	- @8：覆盖equals时的通用约定
		- 自反性reflexivity：自己必须等于自己
		- 对称性symmetry：a=b时，也必须做到b=a。这很容易在多态的比较中出现不符合的情况
		- 传递性transitive：如果a=b，b=c，则a=c
		- 一致性consistency：不会因为多次调用equals方法，其结果会发生改变
		- 非空性Non-nullity：对比双方都不应为空
	- @9：覆盖equals时也要覆盖hashCode

#### 包装类与基础数据类型
- 8种基础数据类型
	- boolean、char、byte、short、int、long、float、double、void
	- 基础数据类型变量占用空间不大，存储在堆中不如存储在栈中高效
	- 所以java提供了不用new来创建变量的方法，设计了并非是“引用”的自动”变量“
	- 数字类型
		- 6种数据类型包装类都有一个共同的父类Number
		- 高精度数字：BigInteger和BigDecimal
- 包装类
	- 基本类型具有包装器类，如int类型和Integer类。使用包装器类使得可以在堆中创建一个非基本对象来表示基本类型
	- 装箱、拆箱
		- 将基本类型转换为包装类的过程称为装箱。每个包装类中提供`valueOf()`的装箱方法
			- 使用`valueOf()`而不是new以创建包装类：除部分包装类外，都会缓存包装类对象，以减少需要创建对象的次数，节省空间
		- 反之称为拆箱。每个包装类提供`基础类型Value()`的拆箱方法
		- Java5+引入了自动装拆箱，这是编译器提供的能力，最后还是会替换为对应的valueOf和intValue方法
	- 包装类特性
		- 重写了Object类的方法
			- `equals(Object obj)`：使得能够反映逻辑相等关系
			- `hashCode()`：逻辑相等的对象的哈希值必须一样
			- `toString() `
		- 实现了Comparable
		- 与String之间相互转换
		- 提供了常用变量
		- 不可变性：包装类的实例对象一旦创建，就没有办法修改了
			- 不可变是为了使得程序更为简单安全，不会被意外更改，可以安全地共享数据
- Effective建议
	- 优先使用基本类型，而不是包装类
    ```java
    // 耗时10s，原因是程序构造了2^31个对象
    Long sum = 0L;
    for (long i = 0; i < Integer.MAX_VALUE; i++) {
        sum+=i;
    }
    
    // 耗时1s
    long sum2 = 0L;
    for (long i = 0; i < Integer.MAX_VALUE; i++) {
        sum2+=i;
    }
    ```


#### 编码与char类型
- 非Unicode编码
	- ASCII
		- 美国的标准，只有128个字符
			- 字符32~126表示的是可打印字符
			- 字符0~31、127用于控制目的
		- 每个字符恰好一个字节，其中最高位规定为0，这是因为计算机最小存储单位是字节
		- 最高位为0使得其他编码方式能够与其得以区分
	- ISO 8859-1（Latin-1）：西欧国家早期标准
	- Windows-1252：ISO 8859-1的替代，上者的编码可以直接看作下者的编码
	- GB2312
		- 使用两个字节表示汉字，且这两个字节的最高位为1
		- 如果最低位为0，则认为是ASCII字符
	- GBK：建立在GB2312的基础上且向下兼容，增加了更多的汉字，包括繁体字
	- GB18030
		- 向下兼容GBK，增加少数民族字符以及中日韩字符
		- 两个字节已经表示不了所有字符，所以该编码使用变长编码，有的字符是两个字节，有的是四个
- Unicode编码
	- Unicode
		- 给所有字符分配了唯一数字编号，但并没有规定这个编号怎么对应到二进制表示
		- 最大编号有6byte，常用编号的最大值则为4byte
		- 使用`U+编号`表示Unicode编号，比如：“马”的Unicode是U+9A6C
	- UTF-8、UTF-16、UTF-32：负责编号到对应的二进制表示
- char
	- 用于表示一个字符，既可以是中文字符，也可以是英文字符
	- 字节流按byte读取，而字符流按char读取
		- 一个char包含的字节数目与编码有关，字符流封装了这种细节
		- 本质上固定占用两个字节，因此仍然可能存在字符需要两个char表示


#### 字符串与日期
- 输出数字的格式化
  `%[指定参数][flags][最小的字符数][.精确度][数据类型(必填)]`
```java
String.format("I have %,.2f money",476578.09876);
// I have 476,578.10 money
```
- 以日期格式输出（数据类型填入以下，传入的参数是Date类型）
	- %tc 完整的日期与时间
	- %tr 只有时间
	- %tA %tB %tC 周月日
	- %<tB 使用前面用过的参数
- 日期加减法
```java
Calendar c = Calendar.getInstance();

// 日期设置为2004年2月7日15:40
c.set(2004,1,7,15,40)
// 返回时间的总微秒数
c.getTimeInMillis();
// 加上35天
c.add(c.DATE, 35);
// 仅日期自增35
c.roll(c.DATE, 35);
// 输出时间
System.out.println(c.getTime())
```

#### 文件概览
- 文件类型
	- 文本文件
		- 每个字节都是可打印字符的一部分
		- 字符到二进制的映射（即编码）有多种方式，如GBK、UTF-8
	- 二进制文件
		- 如压缩文件、pdf、图片等
		- 每个字节不一定表示字符，可能表示颜色、字体、声音大小等
- Java中处理文件的类型
	- 字节流
		- 将I/O设备视作流
		- 字节流以二进制（字节）形式处理数据，没有编码的概念（因此也不能按行处理）
			- 基类是InputStream、OutputStream
			- 子类包括了FileInputStream、ByteArrayInputStream等
		- Java使用装饰者模式以对基本的流增加功能
			- 基类是FilterInputStream和FilterOutputStream
			- 子类包括了BufferedInputStream、PrintStream等
	- Reader和Writer基类（字符流）
		- 能够按字符处理文本数据的基类，改善了流不方便使用的情况
		- 包括
			- Reader/Writer 抽象类
			- InputStreamReader/InputStreamWriter
			- FileReader/FileWriter
			- CharArrayReader/CharArrayWriter
			- StringReader/StringWriter
			- BufferedReader/BufferedWriter
			- PrintWriter/Scanner
	- 随机读写文件
	- File
	```java
	try {
	  File file = new File("MyCode.txt");
	  FileReader fileReader = new FileReader(file);
	  BufferedReader reader = new BufferedReader(fileReader);
	
	  String line = null;
	  while((line=reader.readLine())!=null){
		  System.out.println(line);
	  }
	  reader.close();
	} catch (Exception e) {
	  e.printStackTrace();
	}
	```
	- NIO（New IO）
		- NIO代表一种不同的看待IO的方式，有缓冲区和通道的概念，也支持比较底层的功能
		- 某些操作性能更高
			- 比如复制文件到网络可以利用DMA机制，不用CPU和应用程序参与，直接将数据从硬盘复制到网卡
	- 序列化（Java自带的）
	- 其他形式的序列化：XML、JSON等

#### 字节流
- 抽象类InputStream
	- `read()`
		- 从流中读取下一个字节，返回类型为int，但取值为0~255
		- 读到流结尾时返回-1
		- 如果流中没有数据，则会阻塞直到数据到来、流关闭或异常出现
		- 出现异常时，抛出IOException
	- `read(byte b[])`  /  `read(byte b[], int off, int len)` 
		- 将读入的字节从b\[off]开始放入，最多读取len个字节，返回值为读入字节的个数
	- `close()`
		- 不管read方法是否抛出了异常，都应该调用close方法
- 抽象类OutputStream
	- `write(int b)`
		- 向流中写入一个字节
	- `write(byte b[])`  /  `write(byte[], int off, int len)` 
		- 从byte\[off]开始批量写入字节
	- `flush()`
		- 将缓冲而未实际写入的数据进行实际写入。OutputStream中，该方法为空
	- `close()`
- FileOutputStream
	- `sync()`
		- 本地方法，确保数据写到硬盘上
	```java
	String text = "早安，么么哒";
	byte[] bytes = text.getBytes(Charset.forName("UTF-8"));
	OutputStream output = new FileOutputStream("hello.txt");
	output.write(bytes);
	```
- FileInputStream
	```java
	InputStream input = new FileInputStream("hello.txt");
	byte[] buf = new byte[1024];
	int bytesRead = input.read(buf);
	String text = new String(buf, 0, bytesRead, "UTF-8");
	System.out.println(text);
	```
- BufferedInputStream
	- `public BufferedInputStream(InputStream in, int size)`
		- size是可选参数表示缓存区的大小，默认值为8192
- ByteArrayInputStream/ByteArrayOutputStream
	- 源和目的地是字节数组
- DataInputStream/DataOutputStream
	- 按基本类型和字符串读写


#### 泛型
- 泛型类
	- 类名后面多了一个`<T>`
	- 数据类型也可以为泛型T
- T表示类型参数，泛型就是类型参数化，处理的数据类型不是固定的，而是可以作为参数传入
- 类型参数可以有多个，使用逗号分隔
- Java7+可以省略后面的类型参数
```java
Pair<String, Integer> pair = new Pair<>("可乐",3)
```
- 基本原理
	- 通过类型擦除实现，类定义中的类型参数T会被替换为Object。Java编译器会插入必要的强制类型转换
	- 在程序运行过程中，不知道泛型的实际类型参数
- 好处：更好的安全性、更好的可读性
- 泛型方法
	- 与泛型类不同，调用时不需要特意指定类型参数的实际类型
  ```java
  public static <T> int indexOf(T[] arr, T elm)
  ```
- 泛型接口
- 类型参数的限定：通过extends来表示
	- 上界为某个类、某个接口
	- 限定类型后，就可以使用该类型的方法了
    ```java
    // 上界为某个类
    public class Pair<U extends Number, V extends Number>{
        U first;
        V second;
        public Pair(U first, V second){
            this.first = first;
            this.second = second;
        }
    
        // 允许使用Number的方法
        public double sum(){
            return first.doubleValue()+second.doubleValue()
        }
    }
    ```

- 类型参数限定 - 上界为其他类型
	- 虽然Integer是Number的子类，但是MyArray\<Integer>不是MyArray\<Number>的子类。所以以下方法1会提示编译出错
	- 通过类型限定来解决
    ```java
    // 以下是两个整个MyArray的元素添加进另一个MyArray的方法
    // 1. 会提示编译错误
    public void addAll(MyArray<E> c){
        // ...
    }
    
    // 2. 正确写法
    public <T extends E> void addAll(MyArray<T> c){
        // ...
    }
    
    // 3. 使用通配符——等价于方法2
    public void addAll(MyArray<? extends E> c){
        // ...
    }
    
    MyArray<Number> numbers = new MyArray<>();
    MyArray<Integer> ints = new MyArray<>();
    numbers.addAll(ints);
    ```
- 泛型的通配符
	- 通配符形式更为简洁，但上面两种通配符都有一个重要的限制：只能读，不能写


#### 异常
- 异常类型
	- Throwable，所有异常类的共同父类
		- 构造器参数
			- message：异常消息
			- cause：触发的异常
	- Error
		- 表示系统错误或者资源耗尽，由Java系统自己使用的异常
		- 未受检异常
	- Exception，应用程序错误
		- RuntimeException，未受检异常
		- Exception的其他子类，受检异常
- 异常机制
	- throw与return
		- return代表正常退出，throw代表异常退出
		- return返回的位置是确定的，throw后执行哪行代码由异常处理机制决定
	- 默认机制：输出异常栈信息并退出
	- 捕获异常：使用try/catch/finally
		- catch/finally代码块中重新抛出异常：该异常不能再被当前try-catch代码块处理
		- 如果finally没能捕获异常：在异常抛给上层前先执行finally代码块
		- try或catch中包含return语句
			- 需要先执行finally语句再执行catch语句
			- finally语句不可能改变return的值。该值已经被决定了
		- try或catch和finally中包含return语句
			- try和catch内的return会被略过
	- throws
		- 声明可能会产生异常，受检异常必须声明
		- 子类方法抛出的异常必须不能够多于父类（父无子无）



## #6 容器
#### ArrayList
- 剖析ArrayList
	- 内部维护一个数组elementData，其类型是Object\[]的
	- 初始化
		- 提供一个空数组elementData
	- 添加元素
		- 每次添加数据前都会调用`ensureCapacityInternal(size+1)`以确保数组容量是够的
		- `ensureCapacityInternal(int minCapacity)`会先判断数组是不是空的，如果是，则指定分配大小不小于DEFAULT_CAPACITY
		- 然后，如果需要的长度大于当前数组的长度，则调用`grow()`方法
		- `grow(int minCapacity)`会先计算oldCapacity的1.5倍是否能够容纳minCapacity，如果能，newCapacity就是它了；如果不能则newCapacity等于minCapacity
		- 然后使用`Arrays.copyOf()`创建新数组（通过在参数中指定新的容量大小，就可以直接得到新数组了）
	- 添加在中间
		- 使用`System.arraycopy(Object src, int srcPos, Object dest, int destPos, int length)`将index以后的元素移一位
	- 删除
		- 释放数组对应位置的引用，以便原对象被垃圾回收
	- 基本思维：封装复杂操作，提供简化接口
- foreach语法的原理
```java
// 使用foreach
for(Integr a: list){
	System.out.println(a);
}

// 编译器会将foreach代码转换为类似如下代码
Iterator<Integer> it = list.iterator();
while(it.hasNext){
	System.out.println(it.next());
}
```
- 迭代器接口Iterator
	- 只要对象实现了Iterable接口，就可以使用foreach语法
	- 可以创建Iterator对象的类也不一定需要实现Iterable
	- Iterator接口提供了三个方法：`hasNext()`、`next()`、`remove()` 
	- Itr
		- 内部成员类Itr实现了Iterator接口，并在调用`iterator()`方法时返回新的Itr实例
		- Itr没有复制数据，而是List的视图
		- 内部参数
			- cursor指向下一次调用next方法时返回的对象
			- lastRet指向上一次调用next方法时返回的对象
			- expectedModCount表示修改次数
				- 用于防止在遍历期间数据结构发生修改
				- Iterator内部操作都会检查该数值是否与ArrayList.modCount一致
		- 内部方法
			- remove
				- 删除上一个遍历的元素（lastRet的位置）
				- 删除后，cursor指向删除位置，lastRet=-1
				- 必须通过再次调用next方法才能继续删除
	- ListIterator接口
		- Iterator的子接口，声明了双向遍历的能力
	```java
	// 使用迭代器的remove()
	Iterator<Integer> it = list.iterator();
	while(it.hasNext()){
		if(it.next()<=100){
			it.remove();
		}
	}
	```
	- 迭代器设计模式
		- 将数据的实际组织方式与数据的迭代遍历相分离，是一种关注点分离的思想
		- 需要访问容器元素的代码只需要一个Iterator接口的引用，而不需要关注数据的实际组织形式，可以使用一致和统一的方式进行访问
		- 提供了简单而一致的接口
- ArrayList实现的接口
	- Collection：表示一个数据集合，数据间没有位置的概念
	- List：表示有序的数据集合，扩展了Collection
	- RandomAccess
		- 这是一个没有代码的接口，在Java中被称为标记接口。
		- RandomAccess表示可以随机访问（指根据索引值直接可以定位到具体元素）
		- 在一些通用的算法代码中，可以根据这个声明而选择效率更高的实现。比如二分查找会根据list是否实现了这个接口而采用不同的实现机制
```java
int binarySearch(List<? extends Comparable<? super T>> list, T key){
    if(list instanceOf RandomAccess || list.size()< BINARYSEARCH_THRESHOLD){
        return Collections.indexedBinarySearch(list,key);
    }else{
        return Collections.iteratorBinarySearch(list,key);
    }
}
```
- 序列化
	- 通过`writeObject()`和`readObject()`向序列化机制提供方法
	- 序列化方式是先写入size，再写入每个元素
	- ArrayList的内部实现elementData被transient声明。序列化后重新还原可以节省空间
- 控制数组容量
    - `ensureCapacity(int minCapacity)`：确保数组元素有这么大
    - `trimToSize()`：重新分配一个数组，大小刚好为实际内容的长度。可以节省数组占用的空间

#### LinkedList
- LinkedList实现的接口
	- List（继承了Collection、Iterable）、Serializable
	- Queue（Deque的父类）
		- 主要操作有：添加（add、offer），查看（element、peek），删除（remove、poll）。每种操作都有两种形式
		- 在队伍为空时，element和remove会抛出NoSuchElementException异常，而peek和poll返回null
		- 在队列为满时（显然不包括LinkedList），add会抛出IllegalStateException，而offer只是返回false
- LinkedList实现原理  
	- 内部存在静态内部类Node
		- 该类存在三个变量：item，next，prev
		- 所以LinkedList的内部实现是双向链表
	- 内部有成员变量：size、first、last、modCount
	- node方法（也是get方法的实现）
		- 通过index找到对应节点。如果index靠后，则从后面往前搜索
	- add方法
	    - 直接为last节点后新增一个节点
	    - 要注意这是个双向链表，因此添加节点需要更改周围的节点的指针
	    - 最后要维护内部三个成员变量的语义
	    - 指定位置时，指定的位置是待添加节点的位置
	- indexOf方法：从前往后遍历搜索
	- 删除元素
	    - 不需要专门让被删除节点变为null，毕竟已经没有node指向它了
- LinkedList特点
	- 按需分配空间
	- 不可以随机访问


#### HashMap
- 概述
	- HashTable与HashMap相似，但HashTable是线程安全的，且不允许放入空值
	- 使用`Collections.synchronizedMap(new HashMapp())`进行包装使得map变得同步化
	- 继承自AbstractMap，实现了Map接口
- HashMap的两个参数决定了它的性能：`initial capacity`和`load factor`
	- 容量
		- 桶的数量
		- 默认的初始容量为16
		- 该值总是2的幂
	- `load factor`负载因子
		- 决定了HashMap的容量何时应该扩容
		- 默认值0.75
	- 当HashMap中键值对的数量超过了load factor与容量的乘积时，HashMap会进行一次重新建表(rehashed)，容量扩大一倍
		- 乘积结果使用threshold变量记录
- 实现原理
	- 位运算技巧
		- 在获取hash%n时，可以通过`hash&(n-1)`得到
			- 因为容量n总为2的幂，因此n-1的二进制数中，前半部分总是0，后半部分总是1
		- tableSizeFor()获取最小2的幂次的容量
			- >>>为无符号右移
			- 或运算使得位中出现的1越来越多
			- 而右移保证了只有最高位的右边会不断变为1
			- 或运算与右移操作结合，最终保证了最高位右侧全为1
			- 一个数字只有32位，所以这是合理的
	```java
	static final int tableSizeFor(int cap) {  
		int n = cap - 1;  
		n |= n >>> 1;  
		n |= n >>> 2;  
		n |= n >>> 4;  
		n |= n >>> 8;  
		n |= n >>> 16;  
		return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;  
	}
	```
	- HashMap结构
		- Map.Entry<K,V>接口
		- HashMap.Node<K,V>：单链表
			- 表示键值对
			- Map.Entry的实现
		- HashMap.TreeNode<K,V>：树形节点
			- HashMap.Node的子类
		- Node<K,V>\[] table
			- 表示哈希桶的数组(bin/bucket)
			- 获取哈希对应位置的方式是`(n-1)&hash`
	- get方法
		- 内部实现是`getNode(int hash, Object key)`
		- 得到对应位置后，检查节点是单链表节点还是树节点
			- 对于单链表节点，循环遍历即可
			- 对于树节点，调用其成员方法getTreeNode()
	- put方法
		- 内部实现是`putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict)`
		- 找到对应node后替换值或者新插入节点。对于树节点，使用其成员方法`putTreeVal()`
		- 并确认是否需要将bin转换为树
		- 最后确认是否需要重建哈希表
		- 返回上一个占用该key的value。如果表中没有该key则返回null
	- resize方法
		- 重建或者初始化哈希表的table数组
		- 因为容量为之前的两倍，所以同一bin下的节点去向只有两个选择。通过`oldCap&hash==0`判断节点是否能够留在原来的位置
			- 原理
				- 两种情况对应的是余数最高位是0还是1
				- oldCap只有最高位为1，所以能够通过这种方式得到结果
- 哈希冲突
	- 默认情况下，bin表示某一哈希值求余结果，使用单链表记录对应该bin的所有节点
	- 当某一链表的节点数量超过TREEIFY_THRESHOLD时，将该bin对应的链表转换为树。该阈值默认为8
	- 当某一树的节点数量低于UNTREEIFY_THRESHOLD时，将该bin对应的树重新转为列表.该阈值默认为6
- Collection接口
	- 在List中，能够通过传入Collection类的参数实现导入
		- 内部实现：从Collection类的toArray()方法得到数组，使用System.arraycopy()进行复制
	- AbstractCollection的成员toArray()
		- 新建数组，大小由内部size()得到
		- 使用迭代器（hasNext与next一起操作）填充新数组，并返回
- `keySet()`, `values()`, `entrySet()`方法返回的都是视图
	- 视图不是复制的值，而是基于返回值会直接修改Map自身
	- 3个视图皆是AbstractCollection的子类
	- HashMap提供了3种迭代器对应这3个视图的成员`iterator()`返回的迭代器：KeyIterator, ValueIterator, EntryIterator
- 内部所有迭代器皆是HashIterator的子类
	- 注意。HashIterator没有继承或者实现任何方法，但它的子类都实现了Iterator接口
	- 内部所有迭代过程皆以键值对entry为单位
	- 内部结构
		- next：下一个待返回的entry
		- current：当前entry
		- expectedModCount：提供fast-fail功能（不允许贸然结构性更改）
		- index：当前的slot位置
	- next方法
		- HashIterator没有next方法，而其所有子类的next方法的内部实现都是HashIterator的nextNode()
		- nextNode方法内部是遍历哈希桶的数组及每个节点的next方法
- 红黑树
	- 二叉查找树实现了元素的快速查找
	- 红黑树是一种自平衡的二叉查找树
		- 防止了树的不平衡（比如所有节点在同一侧，此时树和链表几乎没区别）
		- 保证了从根到叶子节点的最长路径不会超过最短路径的2倍
	- 红黑色要求
		- 根节点、叶子节点为黑色
		- 每个红色节点的两个子节点都是黑色的（红色节点不能连续）
		- 任一节点到达叶子节点的路径都包含相同数目的黑色结点
	- 红黑树节点变动
		- 为了仍然保持节点颜色要求，可以采用变色或旋转
			- 旋转：为子树重新挑选根节点
	- AVL树
		- 作为另一种二叉平衡树，AVL树对平衡的要求更高
		- 所以查找效率更高，但平衡调整成本也更高
		- 需要频繁插入删除时，选用红黑树更合适

> roughly 大致
> disperse 分散
> proportional to 成正比
> ameliorate 改善
> ties 平局
> structurally 结构上
> overpopulated 过渡填充
> malicious 恶意
> skeletal 骨干的

#### HashSet
- 继承自AbstractSet，实现了Set接口
- 内部使用HashMap实现
	- `HashMap<E, Object>`，值为常量（Object PRESENT）
	- 内部有许多的操作皆为直接套用HashMap
		- add()：`map.put(e, PRESENT)`
		- iterator()：返回map的keySet的iterator
		- size()：返回map的size
- 构造方法
	- 内部操作总会涉及实例化内部的HashMap
	- 可以指定HashMap的初始容量和负载因子



## #7 线程
- 进程与线程
	- 进程
		- 程序是存放在硬盘中的可执行文件，而一个进程是一个程序的一次启动和执行
		- 现代操作系统中，进程是并发执行的
	- 线程
		- Java将线程调度工作委托给操作系统的调度进程去完成
		- 线程只有得到CPU时间片才能执行指令，并处于执行状态。否则处于就绪状态，等待系统分配的下一个CPU时间片
		- 主流操作系统对线程分配采用抢占式调度模型，优先级高的线程获取的CPU时间片会相对多一些
		- `Thread.currentThread()`来获取当前线程实例
	- 进程与线程
		- 一个进程至少有一个线程
		- 进程是操作系统分配资源的最小单位，而线程是CPU调度的最小单位
		- 线程的上下文切换速度比进程上下文切换速度快得多，所以有时称线程为轻量级进程
- 线程的创建
	- 继承Thread
	- 实现Runnable接口
	- 使用Callable和FutureTask创建线程
	- 使用线程池

#### 线程内幕
- 线程状态
	- NEW
		- 线程创建成功但没有调用start()
	- RUNNABLE
		- 执行状态
		- 包括了得到CPU时间片和等待CPU时间片（就绪）两种
	- TERMINATED
		- 线程的代码逻辑执行完成或者异常终止导致run()结束
	- TIMED_WAITING
		- 线程进入限时等待状态。该状态不会被分配CPU时间片。进入等待的情况如下
			- `Thread.sleep(time)` 
			- `Object.wait(time)` 
			- `Thread.join(time)` 
			- `LockSupport.parkNanos()` 
			- `LockSupport.parkUntil()` 
	- BLOCK
		- 不会占用CPU资源
		- 进入该状态的情况包括：线程等待获取锁、IO阻塞
	- WAITING
		- 无限期等待。进入的情况如下
			- `Object.wait()` 
			- `Thread.join()` 
			- `LockSupport.park()` 
- 使用Jstack查看线程状态
	- JVM自带，可以导出JVM运行实例当前时刻的线程快照
- 线程方法
	- `sleep()`
		- 可能会抛出InterruptedException
		- 线程睡醒后不一定立即得到执行，仍需等待CPU时间片
	- `interrupt()`
	    - 将线程设为中断状态（设置标记位）
		- 中断对线程的影响与线程的状态有关
			- RUNNABLE/BLOCKED
				- 调用`interrupt()` 会设置中断标志位
				- 应在代码中通过循环内使用`isInterrupted()`查询是否有中断标志位
			- WAITING/TIME_WAITING
				- 线程调用join/wait/sleep方法会进入该状态
				- 此时调用`interrupt()` 会推出阻塞，并抛出中断异常
				- 抛出中断异常后，中断标志位会被清空。不会再触发InterruptedException
			- NEW/TERMINATE：调用后无效果
			- 在使用synchronized关键字的过程中不会响应中断请求。使用显式锁Lock以解决问题
	- `join()`
		- 调用该方法的线程会等待方法对应的对象
		- 调用线程进入TIMED_WAITING状态，直到被对方线程执行结束或超时后，重新进入RUNNABLE状态
	- `yield()`
		- 让出CPU的执行权限，使得CPU去执行其他的线程
		- 对应线程的状态仍是RUNNABLE，但是从CPU层面上，执行状态变为了就绪状态
	- daemon相关
		- Java中的线程有两类：用户线程与守护线程
			- 守护线程（daemon、后台线程）专门用于在后台提供某种通用服务的线程，如GC线程
			- 当JVM进程中已无用户线程时，JVM强行终止daemon线程
		- 守护线程的基本操作
			- 通过`setDaemon(true)`设置为守护线程，必须在线程启动前设置
			- 守护线程创建的线程默认也是守护线程
	- `setUncaughtExceptionHandler()`
		- 为线程设置异常处理机制
		```java
		t.setUncaughtExceptionHandler(new UncaughtExceptionHandler() {
            @Override
            public void uncaughtException(Thread t, Throwable e) {
                System.out.println("bug!");
                // e.printStackTrace();
            }
        });
        t.start();
		```
- Object类的成员方法
	- wait/notify方法只能在synchronized代码块中调用这连个方法
	- `wait()`
		- 将当前线程放到条件队列上并阻塞，表示当前线程执行不下去了，需要等待一个条件
		- 这个条件自身无法更改，需要其他线程更改。当其他线程更改了条件后，应该调用`notify()` 
		- 在调用`wait()`时，线程会释放对象锁
	- notify操作
		- `notify()`：从条件队列中选一个线程，将其从队列中移除并唤醒
		- `notifyAll()`：移除条件队列中所有的线程并全部唤醒
```java
public class Temp extends Thread{
    private volatile boolean fire = false;

    @Override
    public void run() {
        try{
            synchronized(this){
                while(!fire) wait();
            }
            System.out.println("fired");
        }catch(InterruptedException e){}
    }

    public synchronized void fire(){
        this.fire = true;
        notify();
    }

    public static void main(String[] args) throws InterruptedException {
        Temp temp = new Temp();
        temp.start();
        Thread.sleep(1000);
        System.out.println("fire");
        temp.fire();
    }

}

// 只有fire==true时且notify()被调用，才能让线程停止等待
/* output:

fire
fired
*/
```
- Callable与FutureTask
	- Callable接口：提供返回值、有throw抛出受检异常声明
	- Future接口
		- `get()`：阻塞性地获取异步任务的结果
		- `cancel()`：取消异步任务的执行
	- RunnableFuture接口
		- 继承了Runnable接口和Future接口
		- 使得Callable接口能被接受为Thread的构造参数
	- FutureTask类
		- 实现了RunnableFuture接口
```java
// 能够返回结果的线程执行
public static void main(String[] args) {
	ReturnableTask task = new ReturnableTask();
	FutureTask<Long> futureTask = new FutureTask<>(task);
	new Thread(futureTask).start();
}

static class ReturnableTask implements Callable<Long>{
	@Override
	public Long call() throws Exception {
		return null;
	}
}
```

#### 线程池
- 使用线程池的原因
    - Java线程的创建需要JVM和OS配合完成大量工作
        - 为线程对栈分配和初始化大量内存块，其中包含至少1MB的栈内存
        - 需要进行系统调用，以便在OS中创建和注册本地线程
    - Java高并发应用频繁创建和销毁线程的操作是非常低效的
- 线程池架构
	- Executor：接口
	- ExecutorService：接口，Executor的子类，对外提供异步任务的接收服务
		- AbstractExecutorService：实现了ExecutorService接口，为ExecutorService中的接口提供默认实现
		- ThreadPoolExecutor：线程池实现类
	- ScheduledExecutorService：接口，实现了ExecutorService接口，是一个可以完成延时和周期性任务的调度线程池接口
		- ScheduledThreadPoolExecutor：继承于ThreadPoolExecutor并实现了ScheduledExecutorService
	- Executors：静态工厂类，提供了四种快捷创建线程池的方法
		- SingleThreadExecutor：只有一个线程的线程池
			- 能够保证所有任务按照指定顺序执行
			- 唯一线程存活时间无限，阻塞队列无界
		- FixedThreadPool：固定数量的线程池
			- 线程数量不超过固定数量，阻塞队列无界
			- 使用场景：需要任务长期密集执行的场景
		- CachedThreadPool：可缓存线程池
			- 收到任务一定立即执行，线程数量无限制，依赖于操作系统能够创建的最大线程数量大小
			- 空闲线程会在60秒后被回收
			- 使用场景：需要快速处理突发性强、耗时较短的任务场景
		- ScheduledThreadPool：可调度线程池
			- `scheduledAtFixedRate(task, time1, time2, unit)`：time1表示首次执行任务的延迟时间，time2表示每次执行任务的间隔时间
			- 使用场景：周期性地执行任务
- 线程池构造参数
	- corePoolSize：线程数量小于此值时，即使有空闲线程也会创建新线程
		- 确定线程数
			- IO密集型任务
				- 由于执行IO操作的时间长，导致CPU的利用率不高。这类任务的CPU常处于空闲状态
				- IO处理线程数应该为CPU核数的两倍
			```java
			SystemPropertyUtil.getInt("io.netty.eventLoopThreads"
				, Runtime.getRuntime().availableProcessors()*2)
			```
			- CPU密集型任务
				- 执行大量计算任务，CPU的利用率很高
				- 线程数等于CPU数
			- 混合型任务
				- 比如HTTP请求操作
				- 最佳线程数=（线程等待时间与线程得到CPU时间之比+1）*CPU核数
	- maximumPoolSize
	- keepAliveTime
	- BlockingQueue\<Runnable>：任务阻塞队列
		- 在阻塞队列为空时，会阻塞当前线程的元素获取操作
		- 当队列中有元素时，被阻塞的线程会被自动被唤醒，唤醒过程不需要用户程序干预
		- 常用实现类：ArrayBlockingQueue（有界）、LinkedBlockingQueue（有界无界皆可）、PriorityBlockingQueue（无界）、DelayQueue（无界）、SynchronousQueue（不存储）
	- ThreadFactory：控制线程产生方式
	- RejectedExecutorHandler：拒绝策略
		- 被拒绝的原因：线程池已关闭、工作队列满了且线程数够了
		- 4种常用决策的父类
			- AbortPolicy拒绝策略（默认）：直接抛出RejectedExecutionException
			- DiscardPolicy抛弃策略：直接丢弃该任务，无异常抛出
			- DiscardOldestPolicy抛弃最老任务策略：将最早进入队列的任务抛弃后，再尝试加入队列
			- CallerRunsPolicy调用者执行策略：如果新任务添加到线程池时失败，则提交任务线程会自己执行该任务
- 线程池调度
	- 如果线程数量小于corePoolSize，直接创建新线程来执行任务
	- 否则，如果任务队列没满，则将任务加入到阻塞队列中
	- 否则，如果线程数量小于maximumPoolSize，则创建新线程来执行任务
	- 否则，执行拒绝策略
	- 当一个线程完成了任务时，会优先从阻塞队列中获取下一个任务
- 线程池的使用
	- 创建`ExecutorService pool = Executors.newFixedThreadPool(3);` 
	- 向线程池提交任务
		- `void execute(Runnable command)` 
		- `Future<?> submit(Runnable task)` 
		- `<T> Future<T> submit(Callable<T> task)` 
- 生命周期回调
	- beforeExecute()：任务执行前
	- afterExecute()：任务执行后
	- terminated()：线程池终止时
- 状态
	- RUNNING：线程池创建之后的初始状态
	- SHUTDOWN：该状态下线程池不再接受任务，但会将队列种的任务先完成
	- STOP：不再接受新任务，也不再处理队列中的剩余任务
	- TIDYING：该状态下所有任务都已经终止或者处理完成，将会执行terminated()钩子方法
	- TERMINATED：执行完terminated()钩子方法后的状态
- 关闭线程池
	- 相关方法
		- `shutdown()`
			- 线程池直到处理完队列中所有任务才退出
			- 状态更改为SHUTDOWN，此时不能够往线程池添加新任务
		- `shutDownNow()`
			- 线程状态更改为STOP，并试图停止所有正在执行的线程，并且不再处理阻塞队列中的任务
		- `awaitTermination()`
			- 让当前线程等待线程池完成关闭
	- 作者推荐的关闭方式
		- 执行shutdown()拒绝新任务
		- 执行`awaitTermination()`指定超时时间，判断是否已经关闭所有任务，线程池关闭完成
		- 如果`awaitTermination()`返回false，或者被中断，就调用`shutDownNow()`方法立即关闭线程池所有任务
		- 补充执行`awaitTermination()`判断线程池是否关闭完成。如果超时，就可以进入循环关闭，循环一定的次数，不断关闭线程池，直到关闭或者循环结束
		- 在JVM进程关闭前，自动将线程池优雅地关闭，以确保资源正常释放：`Runtime.getRuntime().addShutdownHook()`

#### ThreadLocal
- 使用场景
    - 线程隔离
        - 既保证了数据安全，又避免了使用Synchronized带来的性能损失
        - 经典案例：为每个线程绑定一个数据库连接，使得这个数据库连接为线程所独享
- 使用
	- 设置初始值：使用工厂方法`ThreadLocal.withInitial()`或者（Java8以下版本）在初始化时重写其`initialValue()`
	- 设置过的ThreadLocal应该要删除绑定。可以在线程池的`afterExecute()`中将线程内部的Entry得以释放
```java
static ThreadLocal<Value> LOCAL = new ThreadLocal<>();

// main()
Local.set(new Value(10));
System.out.println(LOCAL.get().num);
LOCAL.remove();
```
- 内幕
	- 早期：每个ThreadLocal拥有一个Map（ThreadLocalMap），key为线程实例，value为绑定值
	- Java8+
		- 每个线程拥有一个Map，key为ThreadLocal实例，value为绑定值
		- 优势
			- 每个ThreadLocalMap存储的键值对数量变少
			- ThreadLocalMap会随着线程的销毁一起销毁
	- Entry键值对
		- 键（ThreadLocal）为弱引用
			- 作用是为了尽量避免内存泄漏，但解决不了根本问题，需要开发人员手动调用remove()方法才能彻底解决问题
- ThreadLocal的内存泄漏问题
	- ThreadLocalMap是长生命周期对象，生命周期与线程一样，而ThreadLocal的键值对则是短生命周期对象
	- 使用static声明ThreadLocal，使其成为强引用
```Java
public static ThreadLocal<Integer> nums = new ThreadLocal<>()
```



## #8 并发
#### 并发编程的问题
- 临界区资源（代码段）
	- 临界区资源是一种可以被多个线程使用的公共资源或共享对象，但是（应该要）每一次只能有一个线程在使用它
	- 使用/进入临界区时，应该要要申请资源，退出后则要释放资源
	- 线程安全：当多个线程并发访问某个Java对象时，无论系统如何调度这些线程，也无论这些线程将如何交替操作，这些对象都能表现出一致的、正确的行为
	- 竞态条件：多个线程同时修改同一个变量（可能是由于在访问临界区代码段时没有互斥地访问而导致的特殊情况）
- 由于需要尽可能地释放CPU的能力，因此在CPU上不断增加内核和缓存。随着CPU内核和缓存的增加，导致了并发编程的可见性和有序性问题
- 并发编程的三大问题
	- 原子性：（逻辑上）不可中断的一系列操作
		- 使用javap命令解析class文件可以得到其汇编指令信息
		- 可以发现`++`操作实际上是4个操作（取、获取常数1、加、放）
		- 这4个操作之间是可以发生线程切换的，所以`++`操作不是原子操作，在并发场景中会发生原子性问题
	- 可见性：一个线程对共享变量的修改，使得另一个对象能够立刻看见
		- Java内存模型JMM（Java Memory Model）
			- 屏蔽了各种硬件和操作系统的内存访问差异，实现让Java程序在各平台下达到一致的内存访问效果，定义了JVM如何将程序中的变量在主存中读取
		- JMM规定
			- 所有的变量都存放在公共主存中
			- 每个线程都有自己独有的工作内存
			- 当线程使用变量时会将主存中的变量复制到自己的私有内存中
			- 线程对变量的读写操作，是自己工作内存中的变量副本
		- 可见性问题只出现在共享变量上
			- 局部变量、方法参数不会在线程之间共享
			- Object实例、Class实例和数组元素都存储在JVM堆内存上，堆内存在线程之间共享，所以存在可见性问题
	```java
	private static boolean shutdown = false;
	static class HelloThread extends Thread{
	@Override
	public void run() {
		while(!shutdown){
			// nothing
		}
		System.out.println("再见");
	}
	}
	
	// 讲不出“再见”
	public static void main(String[] args) throws InterruptedException {
	new HelloThread().start();
	Thread.sleep(1000);
	shutdown = true;
	System.out.println("exit main");
	}
	```
	- 有序性：程序按照代码的先后顺序执行，如果程序执行的顺序与代码的先后顺序不同，并导致了错误，即发生了有序性问题
		- 指令重排序：CPU为了提高程序运行效率，可能会对输入代码进行优化，这不保证各个语句的执行顺序同代码中的先后顺序一致，但是会保证最终的执行结果和代码顺序执行的结果是一致的
		- 重排序是在单核时代下非常优秀的优化手段。在多核时代，如果工作线程之间不共享数据或仅共享不可变数据，重排序也是性能优化利器；但会影响多线程并发执行的正确性
- volatile关键字
	- 解决了可见性和有序性问题
	- 底层原理：在汇编指令中的操作多了一个lock前缀指令lock addl。该前缀指令有三个功能
		- 将当前CPU缓存行的数据立即写回系统内存
		- 令在其他CPU中缓存了该内存地址的数据无效
		- 禁止指令重排：作为内存屏障使用

#### synchronized锁
- synchronized可以用于修饰：
	- 实例方法
		- synchronized保护的是同一个对象的方法调用
		- 不同对象的同一synchronized方法，可被多个线程同时访问
		- 不能获得锁的线程会被加入等待队列。对象释放锁后，从中选取一个线程将其唤醒获得锁。不保证公平性（竞争）
		- synchronized保护的是对象。只要线程访问的对象方法被synchronized修饰且锁的是同一个对象，该方法就会被同步执行
	- 静态方法：保护类对象
	- 代码块
		- 圆括号内是保护的对象
		- 同步的对象可以是任意对象，因为任意对象都有一个锁和等待队列
```java
// synchronized锁的是对象
class Num{
    int num = 1;
    synchronized void addNum(int n){
        try {
            Thread.sleep(5000);
            num+=n;
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    synchronized void printNum(){
        System.out.println(num);
    }
}

public static void main(String[] args) throws InterruptedException{
	Num num = new Num();
	new Thread(()->{
		System.out.println("new Thread Start!");
		num.addNum(3);
		num.printNum();
	}).start();
	Thread.sleep(1000);
	num.printNum();
}
/**
new Thread Start!
4
4
*/
```
- synchronized锁的特性
	- 可重入性
		- 指线程得到锁后可以进入多个同一锁下的方法
		- 当线程调用synchronized方法时，会先检查对象是否被锁，然后检查是否被自己线程锁定，如果是，则增加持有数量
		- 当释放锁时，减少持有数量，当数量变为0时才释整个锁
	- 内存可见性
		- 在释放锁时，所有写入都会写回内存
		- 在获得锁后，从内存中读最新数据
	- 使用synchronized来保证内存可见性的成本有点高。使用volatile是更轻量的方法
	- 死锁
		- 应该避免在持有一个锁的同时去申请另一个锁
		- 如果确实需要多个锁，则应该所有的代码都按照相同的顺序去申请锁
		- Java自带的jstack命令会报告发现的死锁。但程序中，Java不会主动处理

#### CAS
- Java并发包JUC对操作系统的底层CAS原子操作进行了封装，为上层Java程序提供了CAS操作的API
- Unsafe类
	- 主要提供一些用于执行低级别、不安全的底层操作，如直接访问系统内存资源、自主管理内存资源等。其中大量的方法都是native方法
	- 操作系统层面的CAS是一条CPU的原子指令`cmpxchg`，而该指令具备原子性
	- 提供三个CAS方法包括了`compareAndSwapObject()`、`compareAndSwapInt()`、`compareAndSwapLong()` 
	- 反射获取Unsafe实例
```java
public static Unsafe getUnsafe(){
	try {
		Field theUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
		theUnsafe.setAccessible(true);
		return (Unsafe)theUnsafe.get(null);
	} catch (Exception e) {
		throw new AssertionError();
	}
}
```
- 使用CAS进行无锁编程
	- 通过CAS将值放在字段上，如果失败就重复前两步直到成功。这种重复叫做CAS自旋
```java
public synchronized void selfPlus(){
	// num++;
	int oldValue;
	do{
		oldValue = num;
	}while(!unSafeCompareAndSet(oldValue, oldValue+1));
}
```

#### JUC
- JUC原子类
	- 与synchronized相比，JUC原子类是基于CAS轻量级原子操作的实现，使得程序运行效率更高
	- 4种原子类
		- 基本原子类：通过原子方式更新Java基础类型的值
		- 数组原子类：通过原子方式更新数组中某个元素的值
		- 引用原子类：通过原子方式更新引用类型。使用AtomicMarkableReference\\AtomicStampledReference类甚至可以解决AtomicBoolean\AtomicInteger进行原子更新时可能出现的ABA问题
		- 字段更新原子类：可以原子更新Integer、Long、Refernence中的值或引用
- AtomicInteger线程安全原理
	- 通过CAS自旋+volatile的方案实现，既保障了变量操作的线程安全性，又避免了synchronized重量级锁的高开销
- 对象操作的原子性
	- 基础的原子类型只能保证一个变量的原子操作，而引用原子类AtomicReference可以保证对象引用的原子性
	- 基本情况下只能保障引用的原子操作，成员值被修改时不能保证原子性
    ```java
    AtomicReference<User> ref = new AtomicReference<User>();
    User user = new User("1", "张三");
    ref.set(user);
    
    User next = new User("2", "李四");
    ref.compareAndSet(user, next);
    ```
	- 要使对象成员更新也具有原子性，则应该使用字段更新原子类
    ```java
    AtomicIntegerFieldUpdater<User> updater = AtomicIntegerFieldUpdater.newUpdater(User.class, "age");
    User user = new User("1", "张三", 0);
    updater.getAndAdd(user, 100); // 100
    ```
- ABA问题
	- 当前线程的活被其他线程干了，但因为其结果和没干之前一样，所以当前线程并不知道活已被干了，于是又做了一遍
	- 使用版本号解决ABA问题：JDK提供了AtomicStampledReference、AtomicMarkableReference来解决ABA问题

#### 并发工具
- CountDownLatch倒数闩
	- 使用`latch.await()`进行等待，`latch.countDown()`进行减数
	- 直到倒数闩的次数减为0，调用线程才可以继续执行
	- 实现了多并发等待的效果


## #9 并发容器
#### 保证ArrayList线程安全
- 使用Collections.synchronizedList()方式为ArrayList加锁
- 使用Vector
	- 特点：每个方法都使用synchronized修饰
- 使用CopyOnWriteArrayList
	- 特点
		- 写时复制
			- 写操作通过复制数组、原子性地修改、替换原数组实现
			- 除了多个线程同时写，其他操作都不需要锁，可以并行
		- 线程安全：可以被多个线程并发访问
		- 其迭代器不支持修改操作
		- 以原子方式支持一些复合操作
	- 内部使用ReetrantLock

#### 保证HashMap线程安全
- 使用HashTable
	- 特点
		- 每个方法都使用Synchronzied修饰
		- 线程安全但是读写效率低
		- key不允许为null
		- 底层使用数组+链表
- ConcurrentHashMap
	- 特点
		- HashMap的并发版本，直接支持一些原子复合操作
		- 读操作完全并行，写操作支持一定程度的并行
		- 解决了HashMap在并发中扩容容易出现死循环的问题
	- 原理
		- JDK1.7：分段锁
		- JDK1.8：采用CAS+Synchronized保证线程安全
			- 插入
				- 每次插入数据时判断在当前数组下标是否第一次插入，是就通过CAS方式插入
				- 然后判断是否其他线程正在进行扩容操作
			- 删除：使用synchronized修饰
- Java中没有并发版的HashSet，但可以使用`Collections.newSetFromMap`基于ConcurrentHashMap构建

#### ConcurrentSkipListMap
- 特点
	- 没有使用锁，所有操作无阻塞，皆可并行
	- 可排序
	- 主要操作复杂度为O(logN)

#### 并发队列
- Java并发包提供了丰富的队列
	- 无锁非阻塞并发队列：ConcurrentLinked~
	- 普通阻塞队列：~Blocking~
	- 优先级阻塞队列：PriorityBlockingQueue
	- 延时阻塞队列：DelayQueue
	- 其他阻塞队列：SynchronousQueue，LinkedTransferQueue



## #10 Maven与测试
#### Maven
- 使用第三方库
	- 传统做法
		1. 下载到本地
		2. 找出里面的jar文件，并添加到Java Build Path
		3. 当代码拷贝到其他电脑中时，需确保拥有同样的配置路径
	- 使用maven：能够自动下载、管理jar包、配置path的构建工具
		1. 新建一个maven project（使用默认项目），其中artifactId为项目名，Group Id以`com.`开头
		2. 在中央仓库（mvnrepository.com）中搜索第三方库
		3. 将依赖文本复制粘贴到新建maven项目的`pom.xml`文件中的`<dependencies></dependencies>`
		4. 对整个项目选择`run as maven build`，并在Goals中填入`clean package`
		5. 之后便可以直接运行`run as Java Application`
	- 将普通java项目转maven项目
	    1. 右键中的`configure`选项中选择`Convert to Maven Project`
	    2. 修改`pom.xml`文件，同上
- 新版eclipse已集成maven
- maven下载jar包后会存储在本地仓库m2
- maven的生命周期：清理-编译-测试-打包-安装-部署

#### 单元测试和JUnit
- 测试的种类
	- 单元测试：对软件中最小可测试单元进行检查和验证。通常是一个函数。属于白盒测试
	- 集成测试：将多个单元相互作用形成一个整体，对整体协调性进行测试。
		- 先单元测试，再集成测试
	- 白盒测试：全面了解程序内部逻辑结构，对所有的逻辑路径都进行测试。一般由程序员完成
	- 黑盒测试：完全不考虑内部结构，按照说明书对函数进行测序，一般由独立使用者完成
	- 自动测试：用程序来测试程序
	- 手动测试：手动输入参数
	- 回归测试：修改旧代码后，重新进行测试以确保没有引入新的错误
- 测试策略
	- 基于main函数的策略
		- 在新的main函数中逐个测试函数功能是否正确运行
	- 基于自动化测试框架的策略
- JUnit 单元测试框架
	- 示例：构成三角形，对方法`judgeEdges(int a,int b,int c)`测试
		- 新建一个测试类，其中有一个测试方法`(test())`，每一个测试方法的头部都需要加上一个注解`@Test`，这样JUnit就会自动执行这些测试方法
    - 断言：断言后者输出结果为false（1，2，3不能构成三角形），若不是，则报异常
      `Assert.assertEquals(false,new Triangle().judgeEdges(1,2,3))`
- 项目中的测试
    - 业务代码放在`src/main`中，测试代码放在`src/test`中
    - 在`pom.xml`导入JUnit的包
    - 右键测试类，选择Run as JUnit Test，每次只能执行一个Junit的类
    - 亦可以右键项目，选择Run as Maven Test，每次能执行所有的junit类
