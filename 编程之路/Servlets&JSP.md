 ## #1 Hello, Servlet&JSP
- HTTP GET方法
	- 能够发送简单的请求
	- 报文
		- 请求行+请求首部
		- 响应首部+数据体
	- 缺点
		- 请求时，GET能发送的数据有限
		- GET中的内容会被添加到URL中
```HTTP 请求https://www.baidu.com/s?wd=servlet
GET /s?wd=servlet HTTP/1.1
Host: www.baidu.com
Accept: text/xml
Connection: keep-alive
```
```HTTP 响应
HTTP/1.1 200 OK
Content-Type: text/html
Content-LengthL: 397
Server: Apache-Coyote/1.1
Connection: close
<html>
...
</html>
```
- HTTP POST方法
	- 用于发送复杂请求
	- 报文
		- 请求行+请求首部+消息体（负载）
- MIME类型
	- 描述数据类型
	- 出现在首部的Accept、Content-Type中
- URL统一资源定位符
	- 构成
		- {Protocol}://{Host}:{Port}/{Path}/{Resource}?{Query}
- Apache
	- 开源的Web服务器应用，用于处理HTTP请求
	- 默认以htdocs为根目录
	- 将工作交给适合的应用
		- 提供动态内容
			- Web服务器只提供静态页面
			- 辅助应用可以生成非静态的即时页面（比如客户自身的信息）
		- 保存数据
- JSP
	- 允许将java代码放入html中


## #2 Web应用体系结构
- 容器(Container)
	- servlet受控于容器，其内部没有main方法
	- 容器提供了
		- 通信支持
			- 与ServerSocket打交道
		- servlet的生命周期管理
			- 负责加载类、实例化和初始化servlet
			- 调用servlet方法
			- 使servlet实例能被回收
		- 多线程支持
			- 自动为请求创建新的线程
				- 注意，只有一个Servlet实例，但可以有多个线程
		- 提供使用XML部署描述文件的方式
		- JSP支持
			- 负责把JSP代码翻译成真正的Java
	- 当容器检测到请求对象为servlet时，会创建两个对象
		- HttpServletResponse
		- HttpServletRequest
	- 容器使得开发者不必了解底层服务，可以更加专注地处理业务逻辑
	- TomCat是常用的Web容器
- 安装TomCat
	- 解压apache-tomcat**.tar.gz
	- 运行`./startup.sh`
	- 访问`http://localhost:8080/`检查是否启动成功
- 自定义的servlet的内容
	- 继承自HttpServlet
	- 重写doGet()、doPost()方法
- servlet的多个名字
	- 公共的URL名
		- 这是客户知道的URL名
	- 部署名
		- 部署人员可以创建名字
	- 实际的文件名
		- 包含实际路径
	- 通过建立servlet名的映射（使得servlet可以拥有多个名字），有助于改善应用的灵活性和安全性
- 映射到servlet的方法
	- 创建一个XML文档，称为部署描述文件（DD）
	- servlet元素：把内部名映射到完全限定类名
	- servlet-mapping元素：把内部名映射到公共URL名
	- 无需写出servlet的实际路径名，容器会进行匹配
```XML
<web-app xmlns="http://java.sun.com/xml/ns/j2ee"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd"
version="2.4">	
	<servlet>
		<servlet-name>我的内部名1</servlet-name>
		<servlet-class>完全限定类名1</servlet-class>
	</servlet>
	<servlet-mapping>
		<servlet-name>我的内部名1</servlet-name>
		<url-pattern>/公共URL</url-pattern>
	</servlet-mapping>
</web-app>
```
- MVC
	- 不仅是业务逻辑与表示分离，并且要让业务逻辑不知道表示的存在
	- M模型
		- 实际的业务逻辑和状态放在模型中
		- 可以理解成不涉及视图及视图状态更改的代码部分，独立成模型。使得模型可以轻松搭配任何形式的UI
		- 通常负责与数据库交互
	- V视图
		- 负责与用户交互、从控制器得到模型的状态
		- JSP即视图
	- C控制器
		- 获得用户输入后，决定对模型的影响
		- 令模型更新后，让视图更新模型的状态
		- servlets应该充当控制器，要注意将业务逻辑与servlets分离


## #3 初探Web应用
- 创建Web应用的4个建议步骤
	- 分析用户视图（浏览器显示的内容）以及高层体系架构
	- 创建开发环境
	- 创建部署环境
	- 对Web应用的各个组件完成迭代开发和测试
- 创建开发环境（这是推荐的环境）
	- etc目录
		- 存放web.xml文件（DD）
	- lib目录
		- 存放第三方Jar文件
	- src/com/example目录
		- 存放源代码
		- 建议代码按照M、C分置于不同文件夹（MVC，注意没有V）
	- classes/com/example
		- 这个目录是编译Java类得到的（使用-d）
	- web目录
		- 存放动态和静态的页面
- 创建部署环境（这是要求的环境）
	- 项目的根文件夹存放于Tomcat目录下的webapps文件夹中
	- 项目的根文件夹
		- 动态视图和静态视图
		- WEB-INF文件夹
			- web.xml
			- lib文件夹
			- classes/com/example/文件
				- 只存放类文件
				- 需要按照和开发环境系统的文件夹放置
- servlet相关的jar文件位于tomcat/lib文件下
	- 在使用import的时候要注意，包名可能不是`javax.servlet`，而是`jakarta.servlet`
	- 在vsc中，可以通过配置classpath的方式添加对应jav包(servlet-api.jar)
- 编译(不使用servlet-api.jar时)
```sh
javac -d classes src/com/example/model/BeerExpert.java 
```
- 编译(使用servlet-api.jar时)
```sh
javac -classpath /opt/apache-tomcat-10.0.27/lib/servlet-api.jar:classes:. -d classes src/com/example/web/BeerSelect.java 
```
- 启动Tomcat
```sh
cd tomcat
sudo bin/startup.sh
# 在我的ubuntu中，直接启动会导致tomcat无权更改文件
```
- 重启Tomcat
```sh
cd tomcat
bin/shutdown.sh
sudo bin/startup.sh
```


## #4 请求和响应
- servlet的生命周期
	- 加载servlet类
		- 容器启动时或者收到第一个请求时进行
	- 实例化servlet
	- init()
		- 填充内部的ServletConfig对象
			- 用于向servlet传递部署时消息
			- 用于访问ServletContext
		- ServletContext
			- 每个Web应用有一个ServletContext
			- 用于得到服务器消息，包括容器的名字和版本，以及所支持API的版本等
	- service()
		- 处理用户请求，并调用对应方法，如doGet()
		- 容器会为每个请求提供一个线程，并在线程中调用该方法
	- destroy()
- 幂等（idempotent）
	- 指可以一遍又一遍地反复做同一件事，而不会产生副作用
	- GET、HEAD、PUT都是幂等的
	- POST不是幂等的
- 获取请求报文的数据
	- `request.getParameter("")`
	- `request.getParameterValues("")`获取数组（指表单中的多选）
	- 其他报文数据
		- getHeader()、getCookies()、getSession()、getMethod()（获取HTTP方法）
- 写入响应报文
	- 常用方法
		- setContentType()
		- getWriter()
- 写入的方式
	- 通过PrintWriter：主要处理字符数据
	- 通过OutputStream：主要处理字节
```java
// PrintWriter
PrintWriter writer = response.getWriter()
writer.println("some text")

// OutputStream
ServletOutputStream out = response.getOutputSt();
out.write(myByteArr);
```
- servlet中获取目录资源
	- getResourceAsStream()，其中字符串参数是包含绝对路径和资源名称
- 请求重定向
	- servlet对response调用sendRedirect()
	- 浏览器收到响应后会自动重定向目标URL发起新请求，用户可能不会察觉重定向的发生
	- 运行URL中使用相对路径
- 请求分派
	- servlet将该请求交给其他应用处理（包括JSP）
	- 用户根本不会察觉重定向的发生
```java
RequestDispatcher view = request.getRequestDispatcher("result.jsp");
view.forward(request, response);
```


## #5 Web应用
#### 参数
- 放置局部常量（相对单个servlet而言）
	- 将参数作为java代码的一部分是下策，这种情况下如果参数需要更改，那么java代码需要重新编译
	- 将数据放在DD文件中，再由java代码获取
		- 使用初始化参数（init-param）
		- 容器只有在启动时读取DD
			- 参数需要修改时，仍需重新启动tomcat
		- java代码只能在init()调用时及之后获取数据
```xml
<servlet>
	<servlet-name>我的内部名1</servlet-name>
	<servlet-class>完全限定类名1</servlet-class>
	<init-param>
		<param-name>adminEmail</param-name>
		<param-value>my@qq.com</param-value>
	</init-param>
</servlet>
```
```java
getServletConfig().getInitParameter("adminEmail");
```
- 放置全局常量
	- 所有的servlet和JSP都可以访问
	- 在DD中使用上下文参数（context-param）
	- 如果应用是分布的，那么每个JVM上都有一个ServletContext
```xml
<servlet>
	<servlet-name>我的内部名1</servlet-name>
	<servlet-class>完全限定类名1</servlet-class>
</servlet>
<context-param>
	<param-name>adminEmail</param-name>
	<param-value>my@qq.com</param-value>
</context-param>
```
```java
getServletContext().getInitParameter("adminEmail");
```
- 热部署可以不必重启Tomcat地完成数据更改

#### 监听者（listener）
- 放置不是字符串的复制数据：使用监听者
	- 在监听者中获取必备的对象信息，然后生成对应对象
	- 将该对象作为属性，设置在ServletContext中
	- 注意从ServletContext获取对象时，默认都是Object类型的数据，需要进行转换
- 上下文监听者（ServletContextListener）
	- 监听ServletContext被创建和销毁的时机
	- 上下文监听者类推荐部署到WEB-INF/classes目录之下
	- 需要在DD中声明
```XML
<listener>
	<listener-class>
		com.example.MyListener
	</listener-class>
</listener>
<context-param>
	<param-name>dog</param-name>
	<param-value>snooby</param-value>
</context-param>
```
```java
class MyListener implements ServletContextListener{
	public void contextInitialized(ServletContextEvent ev){
		SerletContext c = ev.getServletContext();
		String dogName = c.getInitParameter("dog");
		Dog dog = new Dog(dogName);
		c.setAttribute("dog", dog)
	}
	
	public void contextDestroyed(ServletContextEvent ev){
		// ...
	}
}

class Dog{
	private String name;
	public Dog(String name){
		this.name = name;
	}
}

// MyServlet doGet()
Dog dog = (Dog)getServletContext().getAttribute("dog");
```
- 其他监听者
	- ServletContextAttributeListener
		- 监听属性是否增加、删除和改变
	- HttpSessionListener
		- 监听session的创建和销毁
	- ServletRequestListener
		- 监听请求的到来和销毁
	- ServletRequestAttributeListener
		- 监听请求中的属性增加、删除和变化
	- HttpSessionBindingListener
	- HttpSessionAttributeListener
	- ServletContextListener
	- HttpSessionActivationListener

#### 属性
- 属性不是参数
	- 属性可以在代码运行中进行设置和更改，而参数被设置在DD中，不可更改
	- 获取属性时，返回Object类型；获取参数时，返回String类型
- 三种属性
	- 上下文属性
		- 能被所有组件访问
	- 会话属性
		- 能访问特定HttpSession的部分才能访问
	- 请求属性
		- 能访问特定ServletRequest的部分才能访问
- 线程安全
	- 属性不是线程安全的
		- 使用synchronized，为ServletContext / HttpSession加锁
```java
synchronized(getServletContext()){
	getServletContext().setAttribute("foo", "22");
}
```
- RequestDispatcher
	- 获取
		- 从ServletRequest对象中获取
		- 从ServletContext中获取
	- 使用
		- 参数为指向资源的路径（相对路径，绝对路径都可以）
		- 发送
```java
RequestDispatcher view = request.getRequestDispatcher("result.jsp");
view.forward(request, response);
```



## #6 回话状态


## #7 JSP


## #8 无脚本JSP


## #9 JSTL


## #10 定制标记开发


## #11 部署Web应用


## #12 Web应用安全


## #13 过滤器


## #14 企业设计模式


