## #1 Hi，JavaScript
- ECMAScript
	- 为Javascript提供标准
- 变量类型
	- 数字
		- 小数和整数都被看作数字
		- 所以执行计算的时候，可能经常用到`Math.floor()`向下取整
	- 字符串
	- 布尔值
	- undefined
- 声明变量
	- var
	- let
	- const
	- 直接声明：无论在何处声明，该变量都会被视为全局变量
- 全局变量
	- 当加载多个js文件时，所有文件的全局变量都会被加载。此时如果发生重名就会产生匪夷所思的现象
	- 当js文件被重新加载，该文件内声明的全局变量全都会被重新初始化
- 命名要求
	- 只能包含字母、下划线、美元符和数字
	- 不能以数字开头
	- 不能与关键字同名
	- 区分字母大小写
- 条件
```js
if(status == 'hungry'){
	eat();
}
```
- 循环
```js
while(status == 'hungry'){
	eat();
}

for(let i = 0; i<10; i++){
	console.log(i)
}

let nums = [0,1,2];
for(let i in nums){
	console.log(nums[i])
}
```


## #2 js与浏览器
- 常用函数
	- alert()
	- console.log()
	- prompt()
		- 弹出提供输入框的窗口以获取用户输入
		- 参数是告知用户输入的字符串
- debugger关键字
	- 用于在代码中设置断点
	- 在JavaScript中断点需要写入代码中，也可以在浏览器中设置
	- `debugger;`
- 在html页面中声明使用js脚本
	- `<script src="my.js"></script>`
	- 可以放在head元素中，也可以放在body元素的最后
		- 放在body元素最后是最佳方案。浏览器通常按照html顺序加载内容，而html的内容应该被优先加载以减少用户的等待时间
		- 如果放在head元素中，则DOM未生成，无法访问。或者通过如下方式告诉浏览器在页面加载后再执行脚本
	- 可以使用多个script元素
	- script元素既可以引用外部脚本文件，也可以在元素内放置代码，但不能在引用外部脚本文件的script元素中，在元素内添加代码
```js
function init(){
	var planet = document.getElementById('planet')
	planet.innerHTML = "Welcome to here!"
} 

window.onload = init;
```


## #3 类型与变量
- typeof关键字
	- 返回变量的类型，包括方法
```js
var a = "Just a string";
var probe = typeof a;
console.log(probe);
```
- undefined值
	- 未被赋值的变量
	- undefined值属于undefined类型
- null值
	- 表示变量没有指向对象
	- null值属于object类型
- NaN值（Not a Number）
	- 表示不应该存在的数字，比如0/0、虚数等
	- NaN值属于number类型
- Infinity值
	- 表示越过javascript上限的值，比如10/0
- 等于
	- 使用`==`或`!=`
		- 如果比较的数据之间类型不同，JavaScript会尝试暂时将类型转换
			- 自动将boolean、string转换为number
			- 空字符串也会被转为数字，其值为0
			- null和undefined比较结果为true
		- 字符串与字符串之间的比较基于其字母表顺序
		- 对于对象
	- 使用`===`或`!==`
		- 对比过程不会发生自动类型转换
		- 对于对象：判断是否指向同一个对象
	- 使用小于号或大于号也会发生暂时的自动类型转换，且没有不发生自动转换类型的比较
	- 不能直接判断的数据类型
		- 因为这些数据类型不能进行比较（比较结果一般为：自身不等于自身）
			- 判断是否NaN：`isNaN()`
			- 判断是否Ifinity：`isFinite()`
- 类型转换为布尔值
	- 被转换为false：空字符串、0、undefined、null、NaN
	- 其他都被视为true
- 显示类型转换
	- Number()
	- parseFloat()
		- 从字符串开头开始遇到的所有数字部分转换为浮点型
	- parseInt()
		- 可选参数二表示进制，默认为10
- 字符串的运算
	- 遇到减号、乘号、除号时，字符串直接转换为number类型
	- 直到遇到字符串，加号都只用于表示数字相加
- JavaScript中的primitive类型也可被视为object类型
- instanceof
	- 判断对象是否是某类，返回true或false
```js
function Car(color){
	this.color = color
}
var car1 = new Car("red");
console.log(car1 instanceof Car);
```


## #4 函数
- 格式
	- 参数列表无需声明类型
```js
function addNumber(num1, num2){ /*...*/ }
let addNumber2 = function(num1, num2){ /*...*/ }
```
- 参数
	- JavaScript函数传递的是复制的值
	- 参数数量
		- 当参数不够时，未能被赋值的参数被视为undefined
		- 当参数过多时，不影响运作
- 函数在文件中的顺序不重要，但以变量方式说明的函数在调用前出现
- 回调（callback、event handler）
	- 将函数当作参数，提供给其他函数运行
- 箭头函数（Arrow Function）
```javascript
let sum = (a, b) => {
  return a + b;
}

sum(2, 3);
```
- 嵌入函数（nested function）
	- 函数内部的函数
- 闭包（closure）
	- 自由变量（free variables）
		- 不是全局变量，但也不是域所在函数中定义的变量
		- 常出现在嵌入函数中
	- 环境
		- 函数中所有可访问的变量都存储于某个环境中，包括嵌入函数
		- 假如函数作为返回值时，函数的环境也会被附加其中
		- 环境存储的不是变量值的副本，而是变量引用
	- 闭包就是存在自由变量的函数域
	- 闭包的用处
		- 保护变量
```js
function makeCounter(){
	var count = 0;
	function counter(){
		count = count+1;
		return count;
	}
	return counter;
}

let doCount = makeCounter();
doCount(); // 1
doCount(); // 2
doCount(); // 3
// 相比于将count作为全局变量，使用闭包保护了count变量
```
- js中没有函数的重载，但可以根据参数数量的不同表现类似效果
	- 函数内部都存在arguments对象，像数组一样操控
```js
function printArgs(){ 
	for(let i = 0; i<arguments.length; i++){
		console.log(arguments[i]);
	}
}

printArgs("one", 2, 3, "four");
// 即使参数列表为空也不影响
```


## #5 数组
- 声明数组
	- 数组的大小是无穷的
	- 数组可以放置任意类型的数据
	- 未被赋予值得下标位置，其值为undefined。因此对下标数值无限制，负数也可以
	- 数组中存在的undefined值同样占据内存
```js
var scores = [60, "hi", true];
var scores = new Array(60, "hi", true)
scores[10] = 60;

// 仅该方法下只有一个参数时，表示创建的数组大小
var nums = new Array(3);
```
- JavaScript中
	- 数组也是对象
	- 链表等也会呈现数组形式
- 数组方法
	- push()
		- 将元素添加到数组末尾
	- sort()
		- 需要传入对比方法
	- reverse()
	- join()
	- every()
		- 遍历每个元素并根据条件返回true或false
		- 只有所有结果都为true，函数才会返回true


## #6 对象
- 直接创建对象
	- 如果属性名中间存在空格，那么应该使用双引号包裹属性名
	- 不要声明两次属性值
	- 对象的变量不持有对象，持有的是对象的引用
```js
var car = {
	year: 1957,
	color: "red",
	passengers: 2,
	drive: function(){
		alert("Zoom zoom!");
	}
}
```
- 对象的属性
```js
/**
添加、删除、遍历
*/
car.name = "Pony"
delete car.year
for(let prop in car){
	console.log(prop+":"+car[prop])
}
```
- 对象的方法
	- 对象内的属性不能直接用，必须显式使用`this`表示当前对象，使用`this.属性`得到对象内属性
	- 方法method属于函数function，是对象内的函数
	- this总是在使用点访问法访问对象成员时，指向了该对象
- 使用构造器创建对象
	- 构造器名首字母大写不是必须的，只是为了更容易与方法区分
```js
// 使用new创建对象
function Dog1(name, breed, weight, bark){
	this.name = name;
	this.breed = breed; // 品种
	this.weight = weight;
	this.bark = bark;
}

let dog1 = new Dog1
	("Fido", "Mixed", 10,
		()=>{console.log("Woof!")}
);

// 参数较多的时候，使用以下方法在不容易出错
function Dog2(params){
	this.name = params.name;
	this.breed = params.breed; // 品种
	this.weight = params.weight;
	this.bark = params.bark;
}
let dog2Params = {
	name: "Brandy",
	breed: "Mixed",
	12, 
	()=>{console.log("Woof!")}
};
let dog2 = new Dog2(dog2Params);
```
- new关键字
	- 属于操作符
	- 创建对象的流程
		- 创建一个新的对象
		- 将this指向该对象
		- 通过构造器填充参数
		- 返回this指向的当前对象
- instanceof内幕
	- 判断方式是对象是否通过同一个对象构造
	- 即使对象被构造后，删除或添加了其他属性
```js
function Car(color){
	this.color = color
}
var car1 = new Car("red");
console.log(car1 instanceof Car);
```


## #7 对象原型
- Javascript的对象机制
	- 比Java或C++更灵活
	- 不是基于类，而是基于原型。因此对象继承自其他对象
	- 除了primitives，一切事物皆对象
- 原型（prototype）
	- 减少重复部分
		- 对象的方法成员就是重复的部分
		- 对象的方法大抵相似，然而使用new声明的对象在每次构造对象遇到方法成员时，都需要创建方法。此举为浏览器带来性能问题
	- 原型的方法和属性都可以动态地添加、修改
	- 原型模型下向对象调用成员时（包括属性和方法），对象会先查找自身是否存在对应成员，如果没有，则沿着原型链向上查找
	- 对象的原型可以不止一个
- 构建原型
```js
function Dog(name, breed, age){
	this.name = name;
	this.breed = breed; // 品种
	this.age = age;
}

Dog.prototype.bark = function(){
	if(this.age>25)console.log("Woof!")
	else console.log("Yip")
};

var dog = new Dog("Fido", "Mixed", 10);
dog.bark();
```
![[img/prototype.excalidraw]]

- 除非访问原型，否则对象内部不能修改原型的成员值
	- 任何没有访问原型但对原型值进行的操作都被视为覆盖操作（override），并将该值写入对象中
```js
Dog.prototype.sitting = false;
Dog.prototype.sit = function(){
	if(this.sitting){
		console.log(this.name+" is already sitting");
	}else{
		this.sitting = true;   // override!
		console.log(this.name+" is now sitting");
	}
}
dog1.sit(); // Fido is now sitting
dog1.sit(); // Fido is already sitting
dog2.sit(); // Brandy is now sitting
```
- hasOwnProperty()
	- 查看属性是否属于当前对象（而不是原型）
	- 属于Object的方法
- 原型链
	- 指一个对象具有多个原型，形成链状（继承再继承）
	- 使用call方法减少代码重复（不是必要的）
```js
// 这是用于表演的狗
function ShowDog(name, breed, age, handler){
	Dog.call(this, name, breed, age);
	this.handler = handler;
}

ShowDog.prototype = new Dog();
ShowDog.prototype.constructor = ShowDog;
// 这些方法分别表示：立正、漫步、沐浴
ShowDog.prototype.stack = ()=>{console.log("Stack")};
ShowDog.prototype.gait = ()=>{console.log("Gait")};
ShowDog.prototype.groom = ()=>{console.log("Groom")};

let dog1 = new ShowDog(
	"Scotty", "Scottish Terrier", 15, "Cookie"
);
```
![[img/prototype_chain.excalidraw]]

- call方法、apply方法和bind方法
	- 改变this的指向
		- 可以理解为将对象的参数借由其他对象的方法写入自身成员中
		- 在这里，借由了`Dog()`将breed、age、handler写入新的ShowDog对象中
	- 三个方法功能相同但语法不同
	- 如果使用apply方法，那么对应方法的参数列表要使用数组形式
	- 如果使用bind方法，则返回一个新的方法，然后再对其调用
```js
const steven = {
	name: 'Steven',
	battery: 70,
	charge: function(chargeTo, time){
		this.battery += chargeTo
		console.log(time+"将手机充电至"+this.battery)
	}
}

const becky = {
	name: 'Becky',
	battery: 20
}

// call
steven.charge.call(becky, 30, "8:40");

// apply
steven.charge.apply(becky, [30, "8:40"]);

// bind
let becky_charge = steven.charge.bind(becky);
becky_charge(30, "8:40");
```


## #8 常见类型
- Object
	- 所有对象的构造器
	- 提供的成员包括
		- hasOwnProperty()
		- isPrototypeOf()
		- propertyIsEnumerable()
		- constructor
		- toString()、toLocaleString()
		- valueOf()
- Math
	- 对象
- 函数
	- 在JavaScript中，函数也是对象
- Date
	- 创建
		- `Date()`
			- 返回现在时间字符串
		- `new Date()`
			- 返回现在时间对象
		- `new Date(毫秒数)`
			- 返回自1970/1/1开始，加上指定毫秒后的时间
			- 这个参数也称为时间戳timestamp
		- `new Date(字符串)`
			- 指定时间对象的时间
			- 可以使用这种格式："2022-04-21"
			- 只有字符串所表示的时间合理才有意义
		- `new Date(2022, 1, 2, 3, 21, 34, 5)` 
			- 参数分别是年月日时分秒和毫秒
			- 年和月的参数是必须的
			- 注意月份从0开始记起
	- 方法
		- `getHours()`等返回年月日时分秒和毫秒


## #9 DOM
- DOM（Document Object Model）
	- 浏览器在解析和渲染HTML页面的时候，也构建了DOM
	- 当脚本修改了DOM，浏览器会依次更新页面
- 获取元素
	- 通过id
		- `document.getElementById()`
	- 通过类名获取了元素的数组
		- `document.getElementsByClassName()`
	- 通过name属性获取
		- `document.getElementsByName()`
	- 通过元素名获取元素的数组
		- `document.getElementsByTagName()`
	- 通过CSS选择符语法获取的第一个元素/元素的数组
		- `document.querySelector()`
		- `document.querySelectorAll()`
- 以字符串方式填充、插入元素
	- innerHTML属性
		- 获取元素值和子元素的html代码
		- 通过直接修改内容可以实现添加元素
	- textContent属性
		- 只能获取元素的值
		- 即使在文本串中添加html代码，也不会解析得到元素
```js
let container = document.querySelector(".container");

container.innerHTML = "<p>DOM is cool!</p>"; 
container.textContent = "Hello, World!"; 
```
- DOM元素的属性
	- 获取
		- `getAttribute()`
	- 修改
		- `setAttribute(属性, 值)`
		- 该方法可以修改/添加属性
		- 可以修改类名
- 修改元素
	- createElement()：创建元素
	- createTextNode()：创建文本，作为内联元素
	- appendChild()：添加元素作为末尾子元素
	- remove()：移除元素
```js
let newItem = document.createElement("li");
newItem.textContent = "Hello World";
let ul = document.getElementById("playlist");
ul.appendChild(newItem);
```


## #10 事件响应
- 浏览器负责收集所有的响应，比如用户点击、输入、延时地理位置变化等
- 通过设置属性的方式设置响应
```js
let button = document.getElementById('button');
button.onclick = function(){ /* ... */}
```
- 从事件响应中
	- 可以获取元素：target
	- 可以获取鼠标位置，包括
		- clientX, clientY：浏览器窗口的相对位置
		- screenX, screenY：屏幕的相对位置
		- pageX, pageY：浏览器页面的相对位置
```js
function showAnswer(eventObj){
	var element = eventObj.target
	element.src = element.id + ".jpg"
}
```
- 设置延时
	- setTimeOut()：对应时间后触发
	- setInterval()：每隔一段时间后触发
		- 该方法返回一个timer对象，使用该对象作为clearInterval()的参数可停止循环触发
	- 这些方法都属于window对象，因为window是全局对象，因此不必显示使用点访问法表示
```js
setTimeout(()=>{ /**/ }, 5000)
```
- 为元素设置事件响应
	- addEventListener()
	- removeEventListener()


## #11 Promise
- Promise是js中实现异步的对象
	- resolve：如果耗时操作后成功，则使用这个函数（逻辑意义上的成功）
	- reject：如果耗时操作后失败，则使用这个函数
```javascript
let promise = new Promise(function(resolve, reject) {
	// 耗时操作
});
```
- Promise的状态
	- pending：触发promise函数内容，但没出结果
	- fulfilled：结果为成功，调用resolve方法后
	- rejected：结果为失败，调用reject方法后
- 为resolve和reject域函数提供逻辑
	- resolve的回调写在then区块中
	- reject的回调既可写在作为then区块的第二个函的位置数，也可以在写在catch中
	- finally区块一定会被调用
	- 一个promise中允许有多个then区块，称之为promise链
		- 所有then区块中的代码会按顺序执行
		- promise链中，每一步都需要返回promise
```js
let promise = new Promise(function(resolve, // ...

promise
	.then(function success(response){ 
		// ...
	})
	.then(function afterSuccess(){
		// ...
	})
	.catch(function fail(error){
		// ...
	})
	.finally
```
- async/await
	- 能够用更少的代码创建Promise对象
	- async
		- 修饰函数，表示这是个异步函数
	- await
		- 简化async的使用
```js
// 只使用async
async function say(text) {
  if (/*..*/)return Promise.resolve('good!');
}

function handlePromiseResult() {
  const resultOfThePromise = say('Hi');
  resultOfThePromise
    .then(response => console.log(response)); 
}

handlePromiseResult();

// async搭配await
async function say(text) {
  return new Promise((resolve, reject)=>{
	  if(/*..*/) resolve('good!')
  })
}

async


```


## #12 其他
- jquery
	- 用于简化js代码中使用DOM等的流程，由JavaScript开发
```js
$(function(){  // 页面加载后才执行
	// 为id为buynow的元素设置点击事件
	$("#buynow").click(function(){ 
		alert("I want to buy now");
	})
})
```
- BOM（Browser Object Model）
	- 包括提供了alert()、propt()等方法的window对象，和document对象（document对象是window对象的属性）
	- 其他常用方法和属性
		- innerWidth、innerHeight：浏览器宽高
		- close()关闭窗口
		- setTimeOut()、setInterval()
- 异常处理try-catch
```js
// 获取一个不存在的元素
try{
	var message = document.getElementById("message");
	message.innerHTML = "Hello world";
}catch(error){
	console.log("Error!!!"+error.message);
}
```
- 正则表达式
	- 在js中的正则表达式应该包裹在`//`之中
	- 语法
		- `[0-9]`：表示一个0-9的数字
		- `^d`：1个数字
		- `{3}`：表示3个字符，字符要求由该符号之前声明
		- `?`：表示0到1个字符，字符要求由该符号之前声明
		- `$`：表示这里应该是字符串的结尾
	- 如
		- `/[0-9]{3}/`：匹配3个数字，如“285”
		- `/^\d{3}-?\d{4}$/`：可以匹配如“555-1222”
- JSON（JS Object Notation）
	- 使用字符串表示js的对象，其中只包括基本类型的参数
	- JSON与对象之间的切换
```js
let fidoString = '{"name":"Fido", "breed":"Mixed"}';
let fido = JSON.parse(fidoString);
let fidoString2 = JSON.stringfy(fido);
```
- js用于服务端
	- Node.js是比较流行的服务端选择，同样是单线程模型