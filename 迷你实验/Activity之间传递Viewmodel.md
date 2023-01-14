需求是这样的：我需要在Activity之间传递一些数据，但是如果使用Bundle的方式来传递就实在太麻烦了，毕竟传递的数据中包含了我自定义的类，来来回回都要进行序列化和反序列化。百度搜到了原来可以将ViewModel作为两个Activity的公共数据区域，所以就试验一下吧。

实验中定义了两个Activity，一个ViewModel。两个Activity中都提供了按钮，能让一个数字+1，并且将数字结果展示。我希望得到的实验结果是这个数字存放在ViewModel，而无论哪个Activity通过按下按钮对数字进行+1操作后，都能在打开另一个Activity时发现数字得到变化。

首先，在每个Activity中各自通过ViewModelProvider获取一个ViewModel，然后测试一下结果
```kotlin
// MainActivity1
override fun onCreate(savedInstanceState: Bundle?) {  
    super.onCreate(savedInstanceState)  
    setContentView(R.layout.activity_main)  
  
    model = ViewModelProvider(this).get(NumViewModel::class.java)  
    textview = findViewById(R.id.text)  
    button = findViewById(R.id.button)  
    buttonSkip = findViewById(R.id.skip)  
  
    textview.text = model.count.toString()  
    button.setOnClickListener {  
        model.count++  
        textview.text = model.count.toString()  
    }  
  
    buttonSkip.setOnClickListener {  
        startActivity(Intent(this, MainActivity2::class.java))  
    }  
}

// MainActivity2
override fun onCreate(savedInstanceState: Bundle?) {  
    super.onCreate(savedInstanceState)  
    setContentView(R.layout.activity_main2)  
  
    model = ViewModelProvider(this).get(NumViewModel::class.java)  
    textview = findViewById(R.id.text)  
    button = findViewById(R.id.button)  
  
    textview.text = model.count.toString()  
    button.setOnClickListener {  
        model.count++  
        textview.text = model.count.toString()  
    }  
}

// NumViewModel
class NumViewModel: ViewModel() {  
    var count = 0  
}
```
[[]]

现在的状况来看，两个Activity持有的显然不是同一个ViewModel，毕竟count的指并不相等

重新打开一次两个Activity会怎样？方法是从当前所在的Activity2回到Activity1截图，然后重新打开Activity2截图
[[]]

会发现Activity1的ViewModel中数值未变，而Activity2的ViewModel数值回到了初始值0



参考资料
[如何在不同Activity或Fragment中共享数据](https://blog.csdn.net/huweijian5/article/details/114575986#commentBox)
[架构示例](https://github.com/android/nowinandroid/tree/main)