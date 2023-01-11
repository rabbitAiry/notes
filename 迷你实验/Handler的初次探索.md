## Handler的初次探索

> 因为使用Room数据库而开始接触多线程了，但是发现如果想要从数据库中查询结果并更新到列表中时，没有方法直接从函数中获取Runnable作用域中的列表，而且也不应该这么做。所以尝试了使用Handler，以及接触kotlin

- 最基础的版本

  - 收到信息后说一声，连文本都是自带的

  ```kotlin
  class MainActivity : AppCompatActivity() {
      val updateText = 1
      lateinit var text : TextView
      val handler = object : Handler(Looper.getMainLooper()){
          override fun handleMessage(msg: Message) {
              when(msg.what){
                  updateText -> text.text = "hello"
              }
          }
      }
  
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContentView(R.layout.activity_main)
  
          text = findViewById(R.id.text)
          thread {
              Thread.sleep(2000)
              val msg = Message()
              msg.what = updateText
              handler.sendMessage(msg)
          }
      }
  }
  ```

- java与kotlin混用

  - 先定义一个MyPlan类。这是一份购物清单的一项内容，使用java写的

  ```java
  public class MyPlan {
      public MyPlan(String thing, int number, String description) {
          this.thing = thing;
          this.number = number;
          this.description = description;
      }
  
      public String thing;
      public int number;
      public String description;
  }
  ```

  - 尝试在kotlin中使用这个类，并把文本内容传递给Handler

  ```kotlin
  Thread.sleep(3000)
  val plan = MyPlan("Egg",3,"egg for dinner")
  val msg = Message()
  msg.what = updateText
  msg.obj = plan.description
  handler.sendMessage(msg)
  ```

- 传递一个对象

  - 先是尝试了传递MyPlan对象，然后再尝试传递LinkedList对象，皆能正常运行

  ```kotlin
  class MainActivity : AppCompatActivity() {
      val updateText = 1
      lateinit var text : TextView
      val handler = object : Handler(Looper.getMainLooper()){
          @SuppressLint("SetTextI18n")
          override fun handleMessage(msg: Message) {
              when(msg.what){
                  updateText -> {
                      val list = msg.obj as LinkedList<MyPlan>
                      text.text = list[0].description + "\n" + list[1].description
                  }
              }
          }
      }
  
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContentView(R.layout.activity_main)
  
          text = findViewById(R.id.text)
          thread {
              Thread.sleep(3000)
              val plan1 = MyPlan("Egg",3,"egg for dinner")
              val plan2 = MyPlan("Milk",1,"milk for breakfast")
              val list = LinkedList<MyPlan>()
              list.add(plan1)
              list.add(plan2)
              val msg = Message()
              msg.what = updateText
              msg.obj = list
              handler.sendMessage(msg)
          }
      }
  }
  ```

img

​	kotlin代码看起来真的是赏心悦目:)

