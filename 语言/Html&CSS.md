## #1 HTML
- HTML语法
	- 元素由开始标记、内容、结束标记组成
	- 属性
		- 在开始标记内，如`<style type="text/css">`
		- 属性值应该使用引号包裹
- HTML结构
	- 所有HTML内容都在`<html> .. </html>`之内
	- head元素表示HTML头部
		- title元素：标题
		- style元素：样式
		- link元素：插入外部样式表
		- meta元素
			- 为搜索引擎提供关键字
			- 属性name
				- 值为description时，属性content为描述该网页的一段话
				- 值为keywords时，属性content为关键字，使用逗号分隔
				- 值为robots且属性content值为`noindex,nofollow`时，表示请求搜索引擎忽略该网页
	- body元素：表示HTML主体
		- script元素
			- `<script src="my.js"></script>`
- 基础元素
	- 段落`<p>`
		- 注意这是块元素
	- 标题`<h1>..<h6>`
	- 链接`<a>`
		- href属性
			- 指定跳转页面或者到达该页面的某个区域（hypertext reference）
			- 路径格式类似linux
		- target属性
			- 值为`"_blank"`时使用新窗口来打开网页
		- `<a href="/me/handsome.html">吾</a>帅极了`
	- 引用`<q>`
		- 浏览器可能会将样式渲染成不同效果，比如双引号
	- 块引用`<blockquote>`
	- 换行`<br>`
		- 因为其内容没有意义，所以可以省略结束标记
		- 没有文本内容的元素称为空元素。图片标签也属于空元素
	- 列表：有序列表`<ol>`、无序列表`<ul>`
		- 列表项`<li>`
	- 自定义列表`<dl>`
- 元素内的文本
	- 尽管没用元素声明，但直接被视为内联元素
- 跳转到页内区域
	- 使用a标签，并用id属性标记跳转区域，此时将a标签称为目标锚
	- 使用`#the_id`作为跳转路径
- 反义
	- 表示`<`：`&lt`
	- 表示`>`：`&gt`
	- 表示`&`：`&amp`
- 块元素和内联元素（block、inline）
	- 块元素独立显示，占用独立行
	- 内联元素在文字流中存在
- div元素
	- 块元素
	- 将页面分割为不同的逻辑部分。像一个容器一样，将同一部分的元素包含，使得整体网页清晰明了。不要滥用div元素
- span元素
	- 内联元素
	- 跟div元素一样，用于将内联内容分成不同的逻辑部分
- sub元素、sup元素
	- 类似上标和下标$2^x、i_3$
- html5推出的语义元素
	- 仅作为标识，没有特别效果
	- header元素：用于标题等介绍性内容
	- nav元素：用于导航
	- main元素：主要内容
	- section元素：分块的页面部分逻辑
	- article元素：长文本
	- aside元素：内容补充
	- footer元素：页脚


## #2 发布网页
- 选择主机代理商发布网站
	- 提供管理域名等服务
- 域名
	- 类如`baidu.com`
	- 域名由ICANN机构控制，需持续付费以保持域名的使用权
	- 区别于网站名
		- 网站名诸如`www.baidu.com`
		- `www.baidu.com`是域名`baidu.com`的一个网站
- 根目录
	- 网页的最顶层目录，位于服务器中
	- 对于远程服务器，通常使用FTP或SFTP将网页相关内容发送到远程服务器中
		- FTP提供put、get指令发送或获取文件、
	- 当直接访问网站目录而没有指定资源时，服务器会查找该目录下是否存在默认文件
		- 默认文件的名字通常为`index.html`或者`default.htm`
- URL（Uniform Resource Locators统一资源定位符）
	- 全球性地址，用于定位网络上的资源
	- 同时，URL包含了接收资源的协议名
		- `http://`
			- 传输超文本文件的协议
		- `file:///`
			- 查看本地文件


## #3 HTML的发展
- HTML历史
	- 推出于1989年
	- 在HTML3时代，网景和微软恶性竞争，导致格式不一致
	- 1998年，W3C成立并制定了统一的HTML标准，同时推出HTML4.0和CSS
	- HTML4.01存在两个版本：过渡版本及严格版本
	- 2000年推出XHTML，该语言合并了XML和HTML，并被认为截断了HTML的发展
		- 该书出于2007年，并认为HTML5不可能被推出
		- 因为XHTML过于严格，导致没能被太多开发者接受
	- 2012年，HTML5被推出
	- 不会再有更多的版本号了，即使HTML的内容年年不断更新。现有的HTML机制采用向后兼容机制
- 规范
	- 声明自己的类型
		- 在文件第一行声明`<!DOCTYPE html 版本信息>`
		- 声明`<!doctype html>`即可表示为html5
	- 说明内容的类型
		- 使用meta标签，添加于html的head元素内，且在title元素之上
		- 说明内容包括了网页属性、字符编码格式
		- html5的声明方式：`<meta charset="UTF-8">`
	- 样式内容不应该在html中声明，而是css中声明
- 严格的HTML4.01规范
	- 整个文件必须只由DOCTYPE声明和html元素组成
	- html元素内只有head元素和body元素
	- head元素内必须存在title元素
	- body元素内只能包含块元素
		- 内联元素必须包含在块元素内
	- 块元素
		- 不允许包含在内联元素内
		- 不允许包含在p段落元素内，因为只有文本才能组成段落
	- blockquote元素内只能包含块元素
	- a链接元素不能自我嵌套
	- 空元素中不能嵌套其他内联元素
- XHTML
	- 特点
		- 向后与HTML兼容
		- 属于XML
		- 语法更严格
	- html元素需要添加额外属性
		- xmlns属性
			- 指向了标识符。标识符用于向浏览器描述标签的用法
		- lang属性和xml:lang属性
			- 描述使用的自然语言
	- 空元素必须使用诸如`<br/>`的格式
- HTML5
	- 由标记语言、CSS3、JavaScript API等组成的技术集合
	- 提供了丰富特性，包括
		- 画布api
		- 改进的表单
		- 音视频元素
		- 帮助明确页面结构的元素，如页眉
		- 支持本地存储
		- 离线Web应用
		- 支持地理定位
		- Web工作线程


## #4 表格、表单元素与媒体
- 表格
	- table元素：整个表格
	- caption元素：表格的标题
	- tr元素：表格的一行
	- th元素：表格头的一格
	- td元素：表格的一格
		- rowspan属性：占据两行
		- 被占据的行如果还有单元格可以填写，可以直接按顺序放置元素
	- 表格曾经也被用作完成版面设计，不过如今使用CSS完成版面设计才是正道
- 表单
	- 客户端用户填写的特定数据，服务器返回适当的网页
	- form元素
		- action属性
			- 指向了web服务器的URL，其中包含处理表单数据的Web应用程序。这可能是一个php程序
		- method属性
			- 指定了表单以何种方式给服务器发送数据，通常使用POST或GET
			- 使用GET时，表单数据直接添加给URL自身（诸如URL结尾的`?num=3`）
	- 子元素提供
		- 属性name
			- 表单中所有元素都必须有且唯一的name
			- 数据提供给服务器时，就会转化为`num="3"`等键值对数据
		- 属性value
			- 通常指按钮上或者选项中的文字
	- input元素
		- 空元素
		- type属性
			- 值为text，表示一个输入框
			- 值为submit，表示一个按钮，默认带有“submit”文字
			- 值为radio，表示一个radiobutton
				- 使用同一个name属性表示同一组radiogroup
				- 使用checked属性默认选择
			- 值为checkbox，表示一个复选框
			- 值为password，表示密码框，输入密码时文本被隐藏
			- 值为file，表示选择一个文件并提交
	- textarea元素
		- 可输入多行文本的文本框，注意不是空元素
		- 属性rows、cols
			- 文本区的宽高
		- 内容
			- 初始文本
	- select元素
		- 菜单选项控件（点击后下拉出许多选项的控件）
		- option元素：作为控件提供的选项
	- fieldset元素
		- 使用方框包裹内容，常用于复选框选项的外围
		- 使用multiple属性时，允许多选
		- legned元素
			- 方框上的标题
	- label元素
		- 辅助input元素
- frameset元素、frame元素或iframe元素
	- 固件一组框架，比如在其中嵌入网页
- img元素
	- src属性
		- 指定了图片位置
	- alt属性
		- 图片不能显示时，展示其内容以描述图片信息
		- 在HTML4.01中要求该属性不可缺
	- width/height属性
		- 浏览器会根据该值改变图像
- video元素
	- controls属性：是否显示视频进度控制条
	- src属性：视频位置
	- autoplay属性：设置自动播放。该属性不需要值
	- mute属性：设置静音。该属性不需要值


## #5 使用CSS
- 格式
	- 每个花括号视为一个规则
	- 花括号前的内容（如：`p`）称为选择符
	- 只有一种注释样式
```CSS
/* 段落规则 */
p {
	background-color: red;
	border: 1px solid gray;
}
```
- 类
	- 一个元素可以加入多个类，类名之间使用空格隔开
	- 当类的样式发生冲突时，优先选择CSS代码中最后出现的类
	- 在html中的元素内作为class属性声明类
- id
	- 单个页面内，id和元素一一对应
	- 在html中的元素内作为id属性声明
- 继承
	- 元素从其父节点继承了部分样式。当父元素设置样式时，子元素也会应用该样式
	- 不是所有属性都默认被继承
		- 通常，字体属性可被继承，而边框等属性不被继承
		- 可以在这些属性种使用值inherit表示继承
	- 子元素可以选择覆盖该样式
	- 浏览器提供了默认的样式，并将该样式作为最顶层样式
- 伪类pseudo-class
- 选择符
	- 元素名称
		- `p{..}`
	- 多选
		- `h1, h2{..}`
	- 类
		- `.myclass{..}`
	- id
		- `#myId{..}`
	- 双重筛选元素
		- 如`p.myclass{..}`，表示既是段落又是该类的元素
	- 匹配子孙元素
		- 如`div h2{..}`，表示元素既是div元素的子孙，且是h2元素
		- 可以递归表示父子孙关系，如`div h2 #myid{..}`
	- 仅匹配子元素
		- `.myclass>h2{..}`，仅匹配既是myclass类的子元素，且是h2的元素
	- 使用伪类匹配状态
		- 状态包括（这也是推荐的声明顺序）
			- link：未被访问
			- visited
			- hover：鼠标在其上停留
			- focus：浏览器聚焦于链接上的状态。聚焦相对于被键盘/遥控器选中
			- active：用户第一次点击该链接时
		- 元素可能同时处于几个状态
		- `a:link{..}`
	- 使用伪类匹配特定子元素
		- `div:first-chilld{..}`
		- 还有
			- first-letter：第一个字母
			- first-line
	- 属性选择
		- 选择对应属性的元素
		- 如`img[height="300"]{..}`
	- 兄弟选择
		- 如`h1+p{..}`，选择了前面是h1元素的p元素
	- 当一个元素存在多个规则时，所有样式效果会叠加
- 使用css
	- 存在于html头部的style元素中
	- 独立的css文件
		- 在html头部中使用link元素链接到对应css文件
		- 不再需要style元素
		- `<link rel="stylesheet" href="my.css">`
	- 可以使用多个css样式表，只需增加link元素即可
		- 最后声明的link元素，其指向的样式表优先级最高
	- 对样式表进行限制
		- 比如不同的设备优先使用不同的样式表
		- 在link元素中添加media属性定义设备类型
- 层叠的概念
	- 代码指定了样式、浏览器也指定了样式，层叠后得到了最终的网页效果
	- 浏览器收集所有样式，并将越具体的样式赋予越高的优先级
		- 这里的具体指的是选择符
			- id优先级最高，类或伪类其次，元素名称最后
			- 指定了2种或3种，则优先度更高（比如：既指定了id，又指定了类）
			- 可以将指定了id当作100分，指定了类或伪类为10分，指定了元素名称为1分，统计这些分数得到最终优先级排序
		- 具体程度相当的情况下才会按照出现在CSS文件中的顺序排序，越后出现优先级越高
	- 


## #6 样式
- 属性值表示
	- 颜色
		- 使用关键字
			- css支持17种关键字表示颜色
		- 使用`rgb(0,0,0)`
			- 参数既可以是百分比，也可以是十进制数
		- 使用十六进制代码
			- `#cc0000`，可简写为`#c00`
			- 只有两两相等时才可以简写
	- 大小（高度）
		- 使用具体数值
			- px单位
		- 使用百分比：百分比的依据是父元素字体大小
			- %单位
			- em单位
				- 使用小数表示
				- 整数部分为0时，可以省略整数部分。如`.5em`
			- 当网页按比例缩放时，该方式有更佳的显示效果
		- 使用关键字
			- xx-small、small、medium、large等
			- 浏览器会将这些关键字替换为默认像素值
	- 基础display属性
		- 值block
			- 视为块元素
		- 值inline
			- 视为内联元素
		- 值inline-block
			- 跟内联元素相似，但是相比之下
				- 可以设置宽度
				- 上下margin和padding能生效
		- 值none
			- 元素不在页面上
			- script元素的display属性默认值为none
- 盒模型
	- CSS将所有元素看作一个盒子，即一个矩形空间。元素包括块元素和内联元素
	- 边框
		- border，还有诸如border-bottom、border-top-style等属性
		- border-style
			- solid实线、double双实线
			- dotted点线、dashed虚线
			- groove凹进页面、outset凸进页面
			- inset嵌入进页面、ridges凸出
		- border-radius设置圆角
		- 提供了简洁的写法，使用空格分开
			- 三个参数：宽度、风格、颜色，顺序随意
	- 内边距、外边距
		- padding/margin，还有诸如padding-left属性
		- 可以多次设置值，以最后一次声明为最终效果值
		- 提供了简洁的写法以替代原有的padding-left等属性，使用空格分开
			- 诸如`padding: 0px 20px 20px 0px`
			- 一个参数：所有方向共用一个值
			- 两个参数：上下和左右
			- 四个参数：从上开始顺时针
		- 元素之间的margin可能会重叠
			- 两个块元素上下并列放置时
				- 假如上面元素底部margin为20px，下面元素顶部margin为10px，那么两个元素边框之间的距离为20px
			- 元素嵌套在另一个元素时
			- 内联元素的上下margin（除了img元素）
	- 背景图片
		- background-image
			- 诸如`url(images/logo.png)`
		- backgroung-repeat
			- 设置是否重复放置，否则填no-repeat
		- background-position
			- 使用像素、百分数或者关键字
			- 多个值之间使用空格分隔
		- 提供了简洁的写法，使用空格分开
			- 颜色、图片、重复、位置都可以，顺序随意
	- width、height属性
		- 不包括内外边距，仅含内容区
		- 通常内联元素无法设置宽高，但存在部分例外如img元素
		- width属性默认值为auto，占满父容器（match_parent）
		- height属性的值默认为auto，会使所有内容都可见（wrap_content）
	- box-shadow阴影
		- 值包括了offset-x、offset-y、blur-radius羽化、spread-radius阴影放大系数，默认为0、color。其中offset-x、offset-y是必填参数
		- 在box-shadow属性中使用空格将这些值分开
		- 可以设置多个阴影，使用逗号分开阴影效果
	- overflow、overflow-x/y
		- 指定子元素向下溢出时的显示效果
		- 值包括默认值visible、hidden、scroll始终显示滑动条、suto只有溢出时才显示滑动条
	- text-overflow
		- 只有在overflow属性值为hidden时有效
		- 指定子元素向右溢出时的显示效果
		- 值包括默认clip隐藏溢出内容、ellipsis隐藏并使用省略号表示隐藏
- 其他通用属性
	- visibility
		- 值包括visible、hidden、none等
	- vertical-align
		- 水平线上的对齐，尤其针对行内图片
		- 值baseline：元素底部对齐默认文本的基准线
		- 值sub、super：元素底部对齐下标、上标文本的基准线
		- 值text-top：元素顶部与文本顶部对齐
		- 值text-bottom：元素底部对齐文本底部
		- 值top、middle、bottom：行高范围内的顶部、中部、底部
		- 具体值：与文本基准线的距离
	- white-space
		- 值nowrap表示文本不会换行
	- transform
		- 坐标轴：右下为正方向
		- 值translate(x,y)
			- x、y可以使用具体值或百分比。使用百分比时，比照对象为自身宽高
		- 值如`rotate(45deg)`
			- 顺时针旋转
		- 三维变换
			- 在上述值得方法中添加指定变换的坐标轴，如`rotateY(180deg)`
		- 使用空格分隔多个值，注意值得顺序会影响效果
	- backface-visibility
		- 2D：对鼠标经过时（hover）作出响应
			- 鼠标不经过时，显示原来不包括translate属性得效果
			- 鼠标经过时的效果依值而变
		- 3D：依值显示/隐藏背面
		- 值visible
			- 鼠标经过则对translate属性生效
		- 值hidden
			- 鼠标经过则不显示
- 块元素样式
	- text-align
		- 元素内水平方向上的对齐样式
		- 不仅对文本生效，元素内所有的内联元素都生效
		- 该属性可被继承，故对所有子级内联元素都生效
		- 值包括了：left、right、center、justify、
- 字体样式
	- font-family
		- 5个字体系列
			- sans-serif
				- 无衬线
			- serif
				- 有衬线，偏向古典
			- monospace
				- 每个字母具有同样的宽度，主要用于显示代码
			- cursive
				- 手写字体，比如comic sans或书法字体
			- fantasy
				- 独特设计的字体
		- 设置多种字体
			- 字体之间使用逗号分隔，并必须在最后加上字体系列名
			- 计算机会从前往后查看当前设备是否存在对应字体
	- font-weight
		- 使用关键字
			- bold、normal、bolder、light等
		- 使用具体值
	- font-style
		- 设置斜体（italic或oblique都可以）
	- text-decoration
		- 字体效果，包括删除线等
	- line-height
		- 行高
		- 使用具体值
		- 使用百分比
			- 表示当前或父类字体的大小乘以该百分比
			- 子元素继承了父元素的line-height属性，但依旧按照父元素字体乘以百分比得到的具体大小设置行高，而不是子元素自身的字体
		- 使用纯数字
			- 表示元素自身（而不是父元素）的font-size的倍数
	- 可以直接使用font属性，因为其提供了简洁的写法
		- 使用空格分开，需按顺序
		- 字体大小、字体系列是必须有的参数
		- 行高写在字体大小后，并按如下格式
			- `12px/1.6em`
		- 字体系列名格式不变，之间用逗号分隔
		- 顺序
			- `font-style font-variant font-weight font-size/line-height font-family`
	- text-shadow
		- 值包括x-shadow、y-shadow、radius羽化范围、color，使用空格分隔，其中x-shadow、y-shadow是必填参数
		- 可使用多种阴影样式，使用逗号分隔
- 使用外部字体
	- 来自GoogleFonts
	- 来自目录中
		- 需要先声明字体文件及引用方式
```CSS
/* Google Fonts*/
@import url('https://fonts.googleapis.com/css2?family=Montserrat&display=swap');
.block{
  font-family: 'Montserrat', sans-serif;
}

/* 使用外部目录字体*/
@font-face {
  font-family: the-best-font;
  src: url('montserrat-400.OTF');
  font-stretch: normal;
  font-weight: 400;
  font-style: normal;
}

@font-face {
  font-family: the-best-font;
  src: url('montserrat-bold.OTF');
  font-stretch: normal;
  font-weight: bold;
  font-style: normal;
}

.block {
  font-family: the-best-font;
  font-weight: bold;
}
```
- 列表
	- li元素
		- list-style-type设置标记圆点类型
			- disc：默认的圆点
			- circle：空心圆
			- square
			- none
			- 自定义图形：使用`url(..)`
		- list-item-postion设置文字与标记圆点的对应位置
			- inside：文本环绕在标记下方
			- outside：标记在文本下方
- 表格
	- table元素
		- caption-side
			- 设置标题位置
		- border-spacing
			- 单元格之间的间距
			- 该属性是相对于整个表格而言的，不能对单个单元格进行设置
		- border-collapse
			- 值为collapse时，消除边框间距，并将两个相邻的边框合并为共用一个边框


## #7 布局
- 流flow
	- 浏览器用流来布置页面
	- 对于块元素
		- 按顺序解析HTML元素并自上而下依次放置在页面上
	- 对于内联元素
		- 按顺序在水平方向上依次放置，直到没有空间时则放到下一行
		- 换行时，文本元素会被切割
- 漂移元素
	- 渲染到漂移元素时，将其移出流中
	- 漂移元素之后的块元素会替代它的位置占有该行，而块元素内的内联元素会围绕着漂移元素
	- 使用
		- 使用float属性设置漂移，值为left或right
		- 漂移元素既可以是块元素，也可以是内联元素
	- 禁止与漂移元素重叠
		- 使用clear属性禁止漂移元素出现在元素的某一侧，防止了元素之间的重叠
- 水平居中
	- 将左右侧margin设置为auto
- position属性
	- 值有：static、absolute、fixed、relative
	- static
		- 默认值，表示位置由浏览器决定
	- 使用top、right等属性指定其所在位置
		- 既可以是具体值，也可以是百分比
		- 具体值可以为负数
	- 既可以是块元素，也可以是内联元素
	- absolue绝对位置
		- 渲染前，绝对位置的元素就被移出流中
		- 元素有一个对于父容器而言的位置
		- 层叠顺序
			- 较后出现的元素置于更靠近用户
			- 使用z-index来指定层叠位置
	- fixed固定位置
		- 渲染前，绝对位置的元素就被移出流中
		- 元素有一个相对于浏览器而言的位置
	- relative相对位置
		- 元素仍在流中
		- 元素有一个相对于原来位置的偏移，即使越过了父容器的边界也能显示
- flexbox布局
	- 提供了革新的布局方式，轻松实现以前的布局效果
	- 为元素指定display属性为flex，则其子元素皆成为了flex item
		- 此时子元素横向布局
		- 子元素应该为块元素，否则产生奇异的效果
		- 当指定了display属性时，float、clear、vertical-align等效果都会被忽略
	- flex-direction属性
		- 默认值row，表示所有元素横向布置
		- row-reverse，表示横向布置，但是元素顺序相反
		- column
		- column-reverse
	- flex-wrap属性
		- 默认值nowrap，无论一行有多少个元素都不会换行，但是尽可能会缩减元素宽度
		- wrap，表示占满一行时自动换行
		- wrap-reverse，表示占满一行时，该行被放置在下一行，原来行用于继续放置元素
	- 提供简洁的flex-flow属性，可以直接将flex-direction和flex-wrap的值填入，使用空格分隔
	- 元素可以使用order属性指定自身在flex容器中的位置
		- 值越小，越靠前。允许负数
	- flex-basis
		- 用于flex item，指定具体宽度
	- flex-grow
		- 占据行内的空闲空间的比例
		- 默认为0，表示不占用
	- flex-shrink
		- 当元素总宽超过父容器时，按比例压缩元素宽度
	- 提供简洁的flex属性，用于将flex-basis、flex-grow、flex-shrink属性使用空格分隔填入
	- justify-content
		- 该属性不改变元素放置顺序，只改变单行元素其水平方向位置
		- 默认值flex-start，元素占据左侧
		- flex-end，元素占据右侧
		- center，元素居中分布
		- space-between，元素居中分布，且占满一行。第一个元素在行首，最后一个元素占据行尾
		- space-around，元素首尾会有间隙
	- align-items
		- 该属性指示行内元素的单行元素垂直方向位置，类似layout-gravity
		- flex-start，元素占据顶部
		- flex-end，元素占据底部
		- center
		- baseline
		- stretch
	- align-self
		- 与align-items相比，该属性针对单个元素
	- align-content
		- 该属性指flexbox中有所有元素整体水平位置，区别于单行单行元素


## #8 CSS动画
- 关键帧动画（Keyframes）
	- 在2017年以后可用
	- 创建动画
		- 百分比表示表示动画过程的阶段。其中`0%`和`100%`可以用`from`和`to`替换
	- 使用动画
		- 属性animation-iteration-count设置重复次数
			- 0表示会持续播放
			- 使用非整数表示动画会在运作期间终止
		- animation属性提供简洁操作
			- 三个值：动画名、时间、次数
```CSS
/*
在2017年以前的浏览器上展示动画需要这样声明，
分别对应chrome/safari/android、firefox、opera
@-webkit-keyframes font-size-change {}
@-moz-keyframes font-size-change {}
@-o-keyframes font-size-change {}
*/
@keyframes my-animate{
	0%{ /*...*/ }
	100%{ /*...*/ }
}

h1{
	animation-name: my-animate;
	animation-duration: 3s;
	/*
	-webkit-animation: ..
	-moz-animation:
	-o-animation:
	*/
}
```
- transitions动画
	- transition-duration
	- transition-delay
		- 延迟效果不是一触即发。比如设置了状态，但是状态持续时间小于延迟时间，那么动画就会被取消，不会看到动画效果
	- transition-property
		- 直接指定属性，并配合不同状态给该属性不同的值从而完成动画
		- 指定的属性如background-color、transform
		- 默认是all，即对全部属性生效
	- transition-timing-function
		- 设置动画效果
		- 值ease：淡入淡出
		- 值linear
		- 值ease-in、ease-out
		- 值ease-in-out：与ease相比，直到到达动画一半才加速
	- 简洁写法transition属性
		- 顺序随意，但有所限制
		- 第一个文本参数视为property，之后才是timing-function
		- 如果定义了property，那么property必须为第一个参数
		- 第一个时间参数视为duration，之后才是delay
		- 时间参数以`s`或`ms`结尾
		- 多个动画间使用逗号分隔