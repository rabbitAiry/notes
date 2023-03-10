[TOC]

# 深入理解Java虚拟机

### #1 走近Java

##### ex1.5 自己编译JDK

[TOC]

### #2 Java内存区域与内存溢出异常

##### 2.2 运行时数据区域

> JVM会在执行Java程序中把它所管理的内存划分数据区域。这些区域有各自的用途，以及创建和销毁时间

- 程序计数器PCR：当前线程所执行的字节码的行号指示器
- JVM栈：线程私有的，生命周期与线程相同
- 本地方法栈：与JVM栈的区别是，JVM栈为JVM执行Java方法服务，而本地方法栈是为JVM使用到的Native服务
- Java堆Heap：所有线程共享的一块内存区域
- 方法区：所有线程共享的一块内存区域
- 运行时常量池：方法区的一部分
- 直接内存：并不是JVM运行时数据区的一部分

##### 2.3 对象访问

- ##### ex2.4 OOM异常

[TOC]

### #3 垃圾收集器与内存分配策略

##### 3.2 对象已死？

> GC在对堆进行回收前，第一件事情就是确定还有哪些对象活着

- 引用计数算法
  - 给对象中添加一个引用计数器，每当有一个地方引用它时，计数器值加一
- 根搜索算法
  - 通过一系列名为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链。当一个对象到GC Roots没有任何引用链相连接时，证明对象不可用

> JDK1.2+，Java对引用的概念进行了扩充

- 分代
  - 年轻代Young Gen
    - 主要存放新创建的对象
  - 年老代Tenured Gen
    - 主要存放JVM认为生命周期比较长的对象
- 引用分类
  - 强引用
    -
  - 软引用
    -
  - 弱引用
    -
  - 虚引用
- 两次标记过程
- 回收方法区

##### 3.3 GC算法

- 标记-清除算法
  - 算法分为标记和清除两个阶段
  - 特点
    - 效率不高
    - 会导致空间产生大量不连续的内存碎片。空间碎片太多可能会导致分配较大对象时因无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作
    - 下述算法都以此为基本算法
- 复制算法
  - 将可用内存按容量划分围为大小相等的两块，每次只使用其中的一块。当当前块内存用完时，就将还存活着的对象复制到另一块上，然后清理当前块
  - 特点
    - 解决了标记-清除算法的效率、碎片问题
    - 代价过高，牺牲了原来内存的一半
- 标记-整理算法
  - 清除阶段将所有存活对象向一端移动，然后直接清理掉边界以外的内存
- 分代收集算法
  - 根据对象的存活周期的不同将内存划分为几块

##### 3.4 GC

- Serial收集器
  - 单线程的收集器
  - 在垃圾收集时，必须暂停其他所有的线程直到收集结束（"Stop the World"，就像JVM在打扫房间时，用户只能坐着什么都不能干）
- ParNew收集器
  - Serial收集器的多线程版本
- Parallel Scavenge收集器
  - 与ParNew收集器相似，但其目标是达到一个可控制的吞吐量
- SerialOld收集器
  - Serial收集器的老年代版本
- Parallel Old收集器
  - Parallel Scavenge收集器的老年代版本
- CMS收集器
- G1收集器
- 并行与并发
  - 并行Parallel：很多垃圾收集线程工作，但用户线程仍处于等待状态
  - 并发Concurrent：垃圾线程能与用户线程同时执行

##### 3.5 内存分配与回收策略

- 对象优先在Eden分配
- 大对象直接进入老年代
- 长期存活的对象将进入老年代
- 动态对象年龄判定
- 空间分配担保

[TOC]

### #4 虚拟机性能监控与故障处理工具

##### 4.2 JDK的命令行工具

- jps：JVM Process Status
  - JVM进程状况工具，可以列出正在运行的虚拟机进程
- jstat : JVM Statistics Monitoring Tool
  - JVM统计信息监视工具，用于监控JVM各种运行状态的命令行工具
- jinfo : Configuration Info for Java
  - Java配置信息工具，用于实时地查看和调整虚拟机的各项参数
- jmap：Memory Map for Java
  - Java内存映像工具，用于生成堆转储快照(heapdump文件)
- jhat : JVM Heap Analysis Tool
  - JVM堆转储快照分析工具
- jstack : Stack Trace for Java
  - Java堆栈跟踪工具，用于生成JVM当前时刻的线程快照（threaddump）

##### 4.3 JDK的可视化工具

- JConsole
  - 基于JMX的可视化监视和管理的工具
- VisualVm
  - 多合一故障处理工具

[TOC]

### #5 调优案例分析与实战

##### 5.2 案例分析*

- 高性能硬件上的程序部署策略
- 集群间同步导致的内存溢出
- 堆外内存导致的溢出错误
- 外部命令导致系统缓慢

##### ex5.3 Eclipse运行速度调优

[TOC]

### #6 类文件结构

> 代码编译的结果从本地机器码转换为字节码

##### 6.2 无关性的基石

- 字节码（*.class）是构成平台性无关的基石

##### 6.3 Class类文件的结构

- Class文件是一组以8字节为基础单位的二进制流
  
  - 排列紧凑，中间没有分隔符
  - 遇到需要占用8位字节以上空间的数据项时，则会按照高位在前的方式分割成若干个8位字节进行存储

- Class文件格式
  
  - 采用一种类似于C语言结构体的伪结构来存储。只有两种数据类型：无符号数和表
  - 无符号数
    - 以u1、u2、u4、u8来代表1、2、4、8个字节的无符号数
    - 无符号数可以用来模数数字、索引引用、数量值或者按照UTF-8编码构成字符串值

- 魔数与Class文件的版本
  
  - 每个Class文件的头4个字节称为魔数Magic Number，用于确定这个文件是否是一个能被JVM接收的Class文件
  - 许多文件的存储标准都使用魔数进行身份识别而不是扩展名，主要是基于安全考虑

- 常量池

- 访问标志

- 类索引、父类索引与接口索引集合

- 字段表集合

- 方法表集合

- 属性表集合

##### 6.4 Class文件结构的发展

[TOC]

### #7虚拟机类加载机制

> JVM把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型

##### 7.2 类加载的时机

- 与其他在编译时需要进行连接工作的语言不同，Java中类型的加载和连接过程都是在程序运行期间完成的
  - 在类加载时稍微增加一些性能开销
  - 为Java应用程序提高了灵活性
- 类的生命周期：加载》验证》准备》解析》初始化》使用》卸载
- JVM规范了仅四种必须对类进行初始化的情况（主动引用）
  - 遇到new、getstatic、putstatic、invokestatic这4条字节码指令时（实例化、get/set static成员、调用一个类的静态方法）
  - 使用java.lang.reflect包的方法对类进行发射调用的时候
  - 当初始化一个类时，其父类必须先初始化
  - 虚拟机启动时，用户指定要执行main()的主类
- 被动引用

##### 7.3 类加载的过程

- 加载
  
  - JVM完成三件事
    - 通过一个类的全限定名来获取定义此类的二进制字节流
    - 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
    - 在Java堆中生成一个代表这个类的java.lang.Class对象，作为方法区这些数据的访问入口

- 验证
  
  - 确保Class文件的字节流中包含的信息符合当前JVM的要求，并且不会危害虚拟机自身安全
  - 四个阶段的验证过程
    - 文件格式验证
    - 元数据验证
    - 字节码验证
    - 符合引用验证

- 准备
  
  - 为类变量分配内存并设置类变量初始值的阶段，这里的类变量仅包括静态变量（仅被static修饰的变量）
  
  - 同时被static和final修饰的变量初始值直接得到赋值，而不是初始值
    
    ```java
    public static final int value = 123;
    ```

- 解析
  
  - 虚拟机将常量池内的符号引用替换为直接引用的过程

- 初始化
  
  - 执行类构造器\<clinit>()的过程

##### 7.4 类加载器

- 类与类加载器
- 双亲委派模型
- 破坏双亲委派模型

[TOC]

### #8 虚拟机字节码执行引擎

- 所有JVM的执行引擎都是一致的：输入的是字节码文件，处理过程是字节码解析的等效过程，输出的是执行结果

##### 8.2 运行时栈帧结构

- 栈帧Stack Frame
  - 用于支持JVM进行方法调用和方法执行的数据结构
  - JVM运行时数据区中的虚拟机栈的栈元素
  - 存储了方法的局部变量表、操作数栈、动态连接和方法返回地址等信息
  - 每一个方法从调用开始到执行完成的过程，就对应着一个栈帧在虚拟机栈里面从入栈到出栈的过程
- 局部变量表
  - 局部变量表是一组变量值存储空间，用于存放方法参数和方法内部定义的局部变量
  - 以变量槽Variable Slot为最小单位
- 操作数栈（操作栈）
- 动态连接
- 方法返回地址
  - 两种退出方式
- 附加信息

##### 8.3 方法调用

- 方法调用阶段仅确认被调用的是哪一个方法，而不设计方法内部的具体运行过程
- 解析
- 分派
- 动态分派
- 单分派与多分派

##### 8.4 基于栈的字节码解释执行引擎

> 探讨JVM如何执行方法里面的 字节码指令的

- 解释执行
- 基于栈的指令集与基于寄存器的指令集

[TOC]

### #9 类加载及执行子系统的案例与实战

##### 9.2 案例分析

> 主流的Java Web服务器，如Tomcat等，都实现了自定义的类加载器

- Tomcat：正统的类加载架构
  - 要解决的问题
    - 同一服务器的两个Web程序所使用的Java类库可以相互隔离
    - 同一服务器的两个Web程序所使用的Java类库可以互相共享
    - 服务器需尽可能地保证自身安全不受部署的Web应用程序影响
    - 支持JSP生成类的热替换（HotSwap）
- OSGi：灵活的类加载器架构
  - 基于Java语言的动态模块规范
- 字节码生成计数与动态代理的实现
- Retrotranslator：跨越JDK版本
  - Java逆向移植工具

##### ex9.3 动手实现远程执行功能

[TOC]

### #10 编译器优化（早期优化）

> 编译器的前端：把\*.java文件转变成\*.class的过程
> 
> 运行期编译器（后端）（JIT编译器）：把字节码转变为机器码的过程
> 
> 静态提前编译器（AOT编译器）：直接把\*.java编译成本地机器代码的过程

##### 10.2 Javac编译器

> 由Java编写

- JavaC的源码与调试
- 解析与填充符号表
- 注解处理器
- 语义分析与字节码生成

##### 10.3 Java语法糖的味道

- 泛型与类型擦除
- 自动装箱、拆箱与遍历循环
- 条件编译

##### ex10.4 插入式注解处理器

[TOC]

### #11 运行期优化（晚期优化）

##### 11.2 HotSpot虚拟机内的即时编译器

##### 11.3 编译优化技术

##### 11.4 Java与C/C++的编译器对比

[TOC]

### #12 Java内存模型与线程

##### 12.2 硬件的效率与一致性

##### 12.3 Java内存模型

##### 12.4 Java与线程

[TOC]

### #13 线程安全与锁优化

##### 13.2 线程安全

##### 13.3 锁优化

[TOC]