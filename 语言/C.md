## #1 入门
#### 运行
- main函数
	- 程序中所有代码的起点
	- 除了在ANSI C标准中，main函数必须是int类型，且在运行成功的情况下返回0
	- 此处的return语句可以省略，编译器会自动插入
	- 如果自定义了要被调用的函数，那么放在main函数之前可以省略额外的声明
- 运行程序
	- 使用编译器编译代码生成可执行文件，如gcc（GNU Compiler Collection）
	- 执行
```shell
# Windows，生成的文件是cards.exe
gcc cards.c -o cards
cards

# Unix
gcc cards.c -o cards
./cards

# 编译并运行
gcc cards.c -o cards && ./cards
```

#### 数据类型
- 整型
	- char
	- int
		- 为了适应硬件，不同平台的大小可能不一样
		- 在我的surface上，int是32位的
	- short
		- 通常为int的一半大小
	- long
		- 通常为int的两倍大小
- 浮点型
	- float
	- double
- 类型转型
	- 大变量可容下小变量
	- 大容量必须使用类型转换变为小容量
	- 编译器发现运算中出现不同变量时，变成大变量
- 修饰类型的关键字
	- unsigned
		- 被修饰的值只能是非负数
	- long
		- 对数据类型加长，比如`long double`
- 借助`limits.h`，`float.h`库获取数据类型的大小
	- 这些库提供了诸如`INT_MAX`等宏
	- 在库中数据类型的名字会变为大写并得到缩写，比如FLT表示float
- 布尔值
	- ANSI C标准使用0当作false，其他值当作true
	- C99允许在程序中使用true和false关键字，但底层仍是0和1

#### 细节
- 运算
	- 支持三目运算符、位运算
	- 循环中可以使用逗号分隔表达式
- static关键字
	- 修饰局部变量时
		- 允许创建一个只能在局部作用域中访问的全局变量
		- 不可以在其他地方访问
	- 修饰全局变量、函数时
		- 表示只有这个.c文件中的代码能使用这个变量/函数
```C
// 修饰局部变量
int counter(){
	static int count = 0;
	return ++count;
}
```
- GNU工具
	- gcc编译器
	- gdb调试器
	- gprof分析器
		- 检查代码性能、生成性能报告
	- gcov覆盖率测试
		- 检查代码的哪部分运行了
		- 用于写自动化测试时知道测试是否覆盖了所有代码


## #2 指针与字符串
#### 存储器
- 栈
	- 存放局部变量
	- 从高位地址开始占用
- 堆
	- 用于动态存储
- 全局变量
	- 存放函数以外声明的变量
- 常量段
	- 只读
- 代码段
	- 只读

#### 指针
- 找到存储器地址
	- `printf("变量x保存在%p", &x)`
		- `%p`为地址格式
		- 地址使用16进制表示
		- 地址长度取决于软件是32位还是64位，长度等于该位数
- 函数传递指针
	- `*x`
		- 读取存储器的内容
		- 也表示指针变量
```C
void go(int *pos){ // pos值是地址， *pos值才是变量值
	*pos = *pos + 1; // 不可以直接用++
}

int main(){
	int position = 100;
	go(&position);
}
```
- sizeof运算符
	- 运算符和函数的区别
		- 运算符会被编译为一串指令
		- 函数会跳到一段独立代码中执行
		- 程序会在编译期间计算sizeof
	- `sizeof(int)`
		- 告知数据类型在存储器中占用字节数
	- `sizeof("msg")`
		- 返回字符数组长度
		- 注意不是字符串长度
- 数组与指针的区别
	- sizeof
		- 参数是数组时，返回数组长度
	- 指针变量
		- 存储地址的变量，通常被分配4-8byte的存储空间
	- 数组变量
		- 指向数组中的第一个元素
		- 不会被分配空间，编译器会在其出现的地方替换为数组的起始地址，因此不能指向其他地方
		- 数组变量包含了数组长度的信息
	- 退化
		- 数组包括了数组长度的信息，而指针只包含地址信息。数组赋给指针变量时的信息丢失称为退化
		- 将数组传递给函数时，数组发生退化。参数列表只能存在指针变量
```C
// 数组变量
char s[] = "Hello World";
// 指针变量
char *t = s;
char *s1 = "Hello World";

/* sizeof*/
sizeof(s) // 12
sizeof(t) // 4/8

/* 地址*/
&s == s // 数组地址
&t != t // 变量t的地址

/* 不可更改*/
// s = t

/* 函数传递*/
void read(char s[]){ } //s是指针变量
int main(){
    read(s); // s是数组变量
}
```
- 数组第n个元素
	- 可以看作地址指向
	- 每个元素地址之间相差的位数由类型决定，但指针知道类型占用位数，也可以使用和数组一样的下标访问对应元素
		- 编译器会根据变量类型计算实际跳转值
```C
/* 第三个元素*/
arr[2] == *(arr+2)

/* 因此而产生的，奇怪但正确的语法*/
3[arr] == *(3+arr) == arr[3]
```

#### 字符串
- C语言不支持现成的字符串，而是以字符数组代替
	- 因此字符串不是值，不能用switch或`==`进行对比
- 字符串以`\0`结尾，字符数据需要包含该元素
	- `\0`是ASCII字符，其值为0
	- C语言不知道数组有多长，因此此项是必要的
- 不能被修改的字符串
	- 程序中的字符串在程序载入内存时，被放置在常量存储区中
	- 指针指向了常量的地址，但不允许对常量进行修改，所以发生了错误
		- 编译器无法发现这个错误，因为编译器不知道指针指向
		- 可以通过用`const`修饰这个参数，令编译器明白这是个常量，在出现修改时编译器会提醒
	- 使用数组创建字符串即可避免该问题，因为是该常量赋值给了字符串
```C
int main()
{
    char *cards = "JQK";
    char a_card = cards[2];
    cards[2] = cards[1]; // 段错误
}
```
- `char**`
	- 用来指向字符串数组，或者字符串的指针
- scanf()
	- 遇到空格就暂停
	- 允许输入多个字段构成的结构化数据
- scanf()、gets()可能导致溢出
	- 用户输入过多数据时，在没限制输入字符时可能会导致段错误（segment fault）
	- 运气好的情况下，数据能保存且不会报错
	- 运气不好的情况下，发生缓冲区溢出
- fgets()
	- 可以指定最大接收长度，包括`\0`
```C
// scanf
char food[5];
scanf("%s", food);

// fgets
char food[5];
fgets(food, 5, stdin);
```
- 导入`string.h`可以使用更多函数


## #3 IO
#### 数据流基础
- 操作系统由无数个小工具组成
- 默认数据流
	- 标准输入stdin
	- 标准输出stdout
	- 标准错误stderr
- 重定向标准输出
	- 将软件所需参数通过重定向，直接重文件导入输入（导出文件），从而代替了用户在shell手动输入（将结果显示在shell中）
	- 例如：`./ex < file > output.json` 
- 重定向标准错误
	- 将错误消息重定向到对应文件
	- 在C的main函数中返回非0即可表示
	- fprintf函数
		- 输出函数，可以指定是标准错误和标准输出
		- `fprinf(stderr, "出错了！")`
		- `fprintf(stdout, "...")`等价于`printf("...")`
	- 例如：`./ex < file > msg.txt 2> err.txt`
- 管道
	- 一个任务对应一个工具，当需要使用到多个工具时，使用管道
	- 管道能够将一个进程的标准输出连接到另一个程序的标准输入
	- 例如：`(./ex1 | ./ex2) < data > out.txt`
		- `|`表示管道
		- 表示ex1的输出会变为ex2的输入
- 文件（自定义数据流）
	- 操作系统为程序提供了三条数据流：标准输入、标准输出、标准错误。而程序可以创建自己的数据流
	- 创建数据流：使用`fopen()`
		- `FILE *my_in_file = fopen("input.txt", "r")`
			- 文件名
			- 模式
				- w：些
				- r：读
				- a：追加，用于在文末追加数据
			- 如果函数返回0，则表示数据流打开失败
		- FILE是大写的，因为它是用宏定义的
	- 输入到输入流/输出到输出流
		- `fscanf(in_file, "%79[^\n]\n", sentence)`
		- `fprintf(out_file, "Hello %s", "World")`
	- 需要手动关闭数据流
		- 数据流的数量是有限的，用完后应该关闭
		- `fclose(in_file)`

#### main函数与选项
- main函数的输入
	- argc
		- 记录argv数组元素的个数
	- argv
		- 用户输入的命令行按空格分割后，作为数组元素呈现
		- 如输入命令`./ex1 hi`，则有
			- argv\[0] = "./ex1"
			- argv\[1] = "hi"
	- 命令的选项（诸如`ls`命令提供的`-l`选项）
		- 选项参数会作为main函数的输入
- 使用getopt()获取参数
	- 需要导入头文件unistd.h。该文件属于POSIX库
	- getOpt()的第三个参数
		- 表示对有效参数的识别
		- 如“ae:”，表示选项a和选项e有效，其中选项e需要一个参数
	- optarg表示选项需要的参数
	- optind表示已经读取了的选项
	- 当输入的参数包含`-`前，使用`--`进行反义
	- 在循环结束后，argv\[0]表示非程序名和非选项的第一个参数
		- 如输入命令`./ex1 -a hi`，则有argv\[0]="hi"
```C
/**
 * 选项
 *
 * 有效的输入：
 * e3_2 -d now -t Pineapple
 * e3_2 -t
 *
 * 报错的输入：
 * e3_2 -d
 *
*/
#include <stdio.h>
#include <unistd.h>

int main(int argc, char const *argv[])
{
    char *delivery = "";
    int thick = 0;
    int count = 0;
    char ch;
    while((ch = getopt(argc, argv, "d:t"))!=EOF){
        switch(ch){
            case 'd':
                delivery = optarg;
                break;
            case 't':
                thick = 1;
                break;
            default:
                fprintf(stderr, "Unknown option: '%s'\n", optarg);
                return 1;
        }
    }
    argc -= optind;
    argv += optind;
	// ...
}

```


## #4 源文件
#### 源文件
- 函数
	- 不允许函数重名
	- 所有的函数默认看作返回int
	- 函数内能够调用的函数应该在其之前被定义
- 使用函数声明
	- 在代码开头声明`float do_something();`
- 头文件
	- 将所有库导入声明和函数声明都存放在头文件中
	- 引用头文件
		- 在代码中包含头文件`#include "file.h"`
		- 编译器读到`#include`时，将读取头文件内容
- 预处理
	- 诸如`#include`、`#define`、`#ifdef`等
	- 预处理是把C源代码转化为可执行文件的第一个阶段。编译器会在预处理时将头文件内容使用管道插入其中
	- `#include`导入文件
	- `#define`定义宏
		- 宏既可以是一个值，也可以像函数一样
			- 像函数时，括号不可以省略
		- 预处理器会将宏替换为对应值
	-  `#ifdef`、`#else`、`#endif`
		- 实现条件编译，决定哪些代码会被包含
```C
#define DAYS_OF_WEEK 7
#define ADD_ONE(x) ((x)+1)  // 括号不可省略

#ifdef SPANISH            // 如果SPANISH这个宏存在
char *greeting = "Hola";  // 就包含这段代码
#else
char *greeting = "Hello"; // 否则包含这段代码
#endif
```
- 使用其他C源码文件
	- 创建源码文件的头文件
	- 使用include引入该头文件
	- 源码文件一起编译
		- 如：`gcc ex1.c ex2.c -o test`
- 共享变量
	- 使用extern关键字


#### 编译与make
- 编译器的工作
	- 预处理：修改代码
	- 编译：转换为汇编
	- 汇编：生成目标代码
	- 链接：将目标代码放在一起
- gcc的其他能力
	- 优化代码
		- gcc为了提高性能会对代码进行部分优化
		- 大多数优化选项默认是关闭的，因为优化需要花费较长时间。通常只有最后发布阶段进行
		- 有4个级别的优化（级别越高，速度越慢）
			- -O：初级优化
			- -O2：中阶优化
			- -O3：高阶优化
			- -Ofast：最高级别的优化。生成的代码可能与标准有出入，慎用
	- 警告
		- -Wall
			- 提高警告检查的门槛，比如类型错误的警告
		- -Wextra
			- 显示所有警告
		- -Werror
			- 把警告当做错误处理。只要有警告，编译就会失败
			- 通常在多人项目中使用，以维持代码的质量
- 编译器分步骤生成可执行代码
	- 生成目标文件（编译，包含预处理、编译和汇编）
		- `gcc -c *.c`
		- 操作系统会将`*.c`替换为所有的c源文件名
	- 链接目标文件
		- `gcc *.o -o test`
	- 分步骤的原因：按需生成目标文件以节省时间
		- 在项目比较大的时候，编译耗时较长
		- 如果只是修改了部分源码文件，那么只为受到改动的文件编译可以节省许多时间
		- 通过对比源码最后修改时间和目标文件最后修改时间即可判断是否需要重新编译、链接
- make工具
	- 基于可以判断哪些文件需要操作而产生了make工具
	- makefile会调用操作系统的命令，因此有时不能在其他操作系统中使用
	- 目标
		- make编译的文件
		- 每个目标需要使用依赖项和生成方法进行描述。两者构成一条规则
- makefile文件
	- 目标和依赖项共占一行，使用逗号分隔
	- 生成方法独占一行，必须以tab开头
```makefile
launch.o: launch.c launch.h thruster.h
	gcc -c launch.c
thruster.o: thruster.c thruster.h
	gcc -c thruster.c
launch: launch.o thruster.o
	gcc launch.o thruster.o -o launch
```
- make的其他特性
	- 可以使用变量
	- 可以使用通配符
	- 可以使用隐式规则
- cmake
	- 解决了写makefile太过麻烦的问题


## #5 结构、联合与位字段
#### 结构
- 使用结构创建结构化数据类型
	- 结构中可以嵌套地使用结构
	- 获取结构的字段
		- 使用点访问法
	- 每个字段所占据的空间通常以字为单位
```C
struct fish{
	const char *name;
	int age;
};

struct fish nemo = {"Nemo", 3};
struct fish coolfish = {.name="Cool"}
```
- 结构的声明很像数组，因此使用花括号赋值必须在结构声明时进行，否则会被视为数组
```C
struct firefish;
// firefish = {"fire", 10};
```
- 别名
	- 默认情况下结构类型的struct关键字显得啰嗦
	- 使用typedef为结构命名，这个名字称为类型名
		- 结构名与类型名一般只有一个就行
```C
typedef struct{
	const char *name;
	int age;
} fish;

fish nemo = {"Nemo", 3};
```
- 结构的复制
	- 为结构变量赋值相当于计算机复制数据。和java中的类不一样
	- 在作为参数传递给函数时，同样是对结构的复制。因此应该将结构的地址传递给函数
```C
struct fish nemo2 = nemo;
nemo2.age = 1;
printf("%d", nemo1.age);  // 3
```
- 结构与指针
	- 注意括号的添加
		- 区分`(*f).age`和`*f.age`的区别
	- 简易的指针使用表示`->`
		- `(*f).age`等价于`f->age`
```C
void happy_birthday(fish *f){
	（*f）.age = (*f).age+1;
	printf("Happy Birthday %s!", (*f).name);
}

happy_birthday(&nemo);
```

#### 联合
- 使用联合表示逻辑上的一种类型
	- 比如描述水果的数量，既可以用个数表示，也可以用重量表示
	- 但不应该同时使用这两种方式来表示，而是只有一个值有效
	- 联合在空间中占据的大小取决于最长的字段
	- 三种赋值方式
		- C89表示法
		- 指定初始化器
		- 点表示法
```C
typedef union{
	short count;
	float weight;
} quantity;

// C89表示法
quantity q = {4};   // count=4\

// 指定初始化器
quantity q = {.weight=1.5};   //weight=1.5

// 点表示法
quantity q;
q.count = 4;   // count=4
```
- 枚举
	- C语言在创建枚举时会为其中的每个值分配一个数字，从0开始
```C
typedef enum{
	COUNT, WEIGHT
} quantity_type

quantity_type t = COUNT; // 0
```

#### 位字段
- 位字段（bitfield）
	- 表示一串数据，其中不同的位用于表示不同都的逻辑意义
	- 每个字段都必须使用unsigned int表示
	- 每个字段以`:num`结尾，表示该字段占据的位数
```C
typedef struct{
	unsigned int low: 1;
	unsigned int high: 1;
	unsigned int en: 2;
}my_bit_field;
```


## #6 数据结构
#### 链表
- 链表
	- 一种抽象的数据结构。抽象是因为可以用来保存很多不同类型的数据
	- 通过创建递归结构创建了链表
- 递归结构
	- 在结构中含有对下一个结构的链接
	- 递归结构必须包含结构名
```C
typedef struct island{  // 这里的island不可以省略
	char *name;
	char *opens;
	char *closes;
	struct island *next; // 递归时必须使用结构名
}island;
```

#### 动态存储
- 获取空间
	- 使用malloc()在堆上分配空间
	- malloc()会返回一个指向新分配空间的指针
		- 返回的是通用类型的指针，即`void*`类型的指针
- 释放空间
	- C语言没有垃圾回收机制，需要确保分配的空间得到回收
	- 使用free()释放存储器
- malloc()和free()都在stdlib.h头文件中
```C
#include <stdlib.h>
// ...
island *p = malloc(sizeof(island));
free(p);
```
- 复制字符串
	- 字符串默认只能通过复制指针传递到函数中，这不是容易接受的概念，尤其是在为多个结构赋值时，需要确保多个结构不要指向同一个字符串才能保证各自独立
	- 解决方法
		- 使用malloc()划分空间，将字符串的字符复制到这个堆，再将堆的指针提供给结构
		- 或者在结构中用数组保存字符串
	- string.h提供了完成该行为的方法：strdup()
```C
char *s = "MONA LISA";
char *copy = strdup(s);
```
- 检测内存泄露：valgrind工具（英灵殿）
	- 原理：拦截并使用了valgrind自身提供了malloc()和free()方法
	- 使用
		- 为可执行文件提供调试信息，比如行号
			- `gcc -g spies.c -o spies`
		- 下载并使用valgrind
			- `valgrind --leak-check=full ./spies`


## #7 高级函数
#### 函数指针
- 函数指针
	- 函数指针保存了函数的地址，使得函数可以像参数一样传递给方法
	- 格式
		- `返回类型(*指针变量)(参数类型)`
	- 函数指针在调用或者获取其地址时不必写`*`或`&`，因为C编译器已经能够识别
	- 高级语言利用函数指针创建了很多面向对象的特性
```C
int my_func(int p1){/**/}
char** my_func2(char *p2, int p3){/**/}

int (*warp_fn)(int) = myfunc;
warp_fn(3);

char** (*warp_fn2)(char*, int) = myfunc2;
warp_fn2("hi", 2);
```
- C标准库的排序函数qsort
	- qsort()会在原数组上进行改动
	- 需要将排序方法传递给qsort
```C
qsort(void *array, 
	  size_t length, 
	  size_t item_size, 
	  int (*comp)(const void *,const void *));

int my_compare(const void* a, const void* b){
	int num_a = *(int*)a;
	int num_b = *(int*)b;
	return num_a - num_b;
}

int scores[] = {33, 54, 26};
qsort(scores, 3, sizeof(int), my_compare);
```

#### 可变参数
- 由stdarg.h头文件提供处理可变参数函数的代码
	- 宏(macro)
		- 可以看作是特殊类型的函数，可以修改源代码
		- 对可变参数的处理用到了宏。预处理器会为宏插入代码
	- 使用`...`表示还有更多参数
	- 使用va_list保存可变参数
		- 处理后应该使用va_end销毁va_list
	- 使用va_start说明可变参数从参数列表的什么位置开始
		- 因此函数中至少有一个普通参数，用于为可变参数提供定位
```C
#include <stdarg.h>

void print_inputs(int args, ...){
	va_list ap;
	va_start(ap, args);
	for(int i = 0; i<args; i++){
		printf("arg:%i\n", va_arg(ap, int));
	}
	va_end(ap);
}
```


## #8 静态库与动态库
#### 静态库
- 标准头文件目录
	- 对于使用`<>`包裹的头文件，编译器会在标准目录中查找
	- 在类Unix系统（Mac、Linux），标准目录在
		- `/usr/local/include`
		- `/usr/include`
	- 在MinGW中，标准目录在
		- `C:\MinGW\include`
- 共享头文件的方法 
	- 将头文件保存在标准目录中
	- 在include语句中使用完整路径名
		- `#include "/learn/test.h"`
	- 告诉编译器位置
		- `-I`用于告诉编译器还可以在哪里找头文件。编译器会先检查该目录，再查看标准目录
```C
gcc -I/learn test1.c -o test
```
- 共享目标文件
```C
gcc /learn/test1.o /learn/test2.o -o test
```
- 目标文件的存档
	- 存档文件由一批目标文件打包而成
	- 存档即静态库
	- 当需要共享的目标文件数目较多时，使用文件存档可以一次告诉编译器一批目标文件
	- 存档文件以`lib[名字].a`的形式存在
	- 查看存档：使用nm命令
		- `nm libl.a`
		- T表示文本，说明这是一个函数
	- 查看存档里的目标文件：使用`ar -t`
	- 提取存档里的目标文件：使用`ar -x`
	- 创建存档：使用ar命令
		- `ar -rcs libmycode.a t1.o t2.o`
		- 存档名必须为`lib[名字].a`的形式
	- 链接存档
		- `gcc t1.c -lmycode -o test`
		- 使用`-l`表示使用存档，不需要写lib和`.a`
		- 可以多次使用`-l`表示使用多个存档
- 静态链接的特点
	- 在编译程序时链接
	- 一旦链接以后就不能修改，静态库的代码一同并入可执行文件中

#### 动态库
- 动态链接
	- 在程序运行前进行链接操作
	- 这意味着对动态库进行修改但无需对程序的可执行文件进行修改，就能让修改效果生效
- 动态库
	- 同样由多个目标文件创建，但除此之外，其中包含的目标文件在动态库中链接成了一段目标代码
	- 不同的操作系统有不同的后缀，但创建方法相同
		- Windows（MinGW）：dll
		- Windows（cygwin）：dll.a
		- Linux：so
		- MAC：dylib
	- 动态库同样应该放入标准目录
- 创建放入动态库的目标文件
	- `gcc -fPIC -c t1.c -o t1.o`
	- `fPIC`选项用于告知编译器想要创建位置无关的代码
		- 位置无关表示代码可以加载到任意位置
		- 一些操作系统需要该标志位，大多数不需要，并且会提出警告，但不影响使用
- 创建动态库
	- `gcc -shared t1.o -o /usr/libs/libt1.so`
	- `-shared`表示将目标文件转化为动态库
	- 若想重命名库，则需要重新编译
- 如果动态库没有放在标准目录中，那么需要通过设置环境变量将该动态库所在目录添加才可以正常运行
```sh
#linux设置环境变量
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:目录

#windows minGW下设置环境变量
PATH="%PATH%:目录"
```


## #9 系统调用
- 约定俗成，当返回-1时，表示当前函数运行时出现了错误
- `system()`
	- 接受一个字符串参数，并把它作为命令执行
	- 如`system("ls -l")`
	- 这种做法不安全，用户可以在文本中注入命令行代码
- `exec()`
	- 明确地指定操作系统运行哪一个程序
	- 位于`unistd.h`中
	- 通过运行其他程序来替换当前进程
		- 可以告诉exec()要使用哪些命令行参数和环境变量
		- 正在运行的程序被替换后直接终止，之后的代码不会再被执行，除非替换过程中出现错误
	- 参数
		- 程序（包括程序位置）
		- 命令行参数，其中第一个命令行参数是程序名
			- 所以参数1和参数2应该相同
			- 命令行参数可以有多个
		- NULL
			- 输入完所有命令行参数后，下一个参数是NULL，告诉函数没有其他参数了
		- 环境变量
			- 提供了额外传递参数的机会
			- 该参数以数组形式存在，其中数组的最后一个元素必为NULL
	- 不同版本的exec：execl()、execlp()、execle()等
		- 后缀含l
			- 表示接收参数列表
		- 后缀含v
			- 表示接收参数向量或参数数组
		- 后缀含p
			- 表示根据PATH查找程序
		- 后缀含e
			- 表示要使用环境变量字符串数组
- 传递环境变量
	- 读取环境变量getenv()
	- 传入环境变量
		- 环境变量的格式是“变量名=值”
```C
// 传入
char *arr[] = {"JUICE=peach and apple", NULL};
execle("dinner", "dinner", NULL, arr);

// 读取
printf("Juice:%s\n", getenv("JUICE"));
```
- 处理调用错误
	- 当exec函数调用出现错误，原来程序后面的代码会继续执行
	- 使用errno变量排查错误
		- 定义在`errno.h`中，系统在调用出错时设置errno变量为错误码
		- `string.h`的`stderror()`可将错误码转为错误信息
```C
/**
查看网络配置。如果ifconfig运行失败，则运行ipconfig
*/

#include<stdio.h>
#include<unistd.h>
#include<errno.h>
#include<string.h>

int main(){
	if(execl("/sbin/ifconfig", "/sbin/ifconfig", NULL) == -1){
	if(execlp("ipconfig", "ipconfig", NULL) == -1){
		fprintf(stderr, "%s", strerror(errno))
		return -1
	}
	return 0
}
```
- `fork()`
	- 克隆当前程序，原来的程序继续运行
		- 克隆后，原进程叫父进程，新进程叫子进程
	- Windows不支持fork()，除非使用Cygwin
	- 函数会返回一个pid_t类型的数值
		- 子进程的返回值为0，而父进程的返回值是子进程的进程标识符
		- 不同的操作系统使用不同数据类型的PID，所以使用pid_t对其抽象
	- 操作系统对fork()的优化
		- 操作系统不会真的复制父进程的数据，而是让父子进程共享数据
		- 但如果子进程需要对数据进行修改，操作系统会为其复制一份数据。称为“写时复制”（copy-on-write）
- exit()
	- exit(1)会立刻终止程序，并把退出状态置1
	- 该函数不会失败，因此没有返回值
- waitpid()
	- 等待子进程结束后才返回
	- 参数1为子进程的id
	- 参数2pid_status
		- 为int指针，用于保存进程的退出消息
		- 保存了好几条信息，其中前8位为退出状态
	- 参数3为选项


## #10 进程间通信
#### 数据流与管道
- 进程记录数据流的目的地
	- 进程用一个数字表示数据流，称为文件描述符
	- 文件描述符描述的文件包括了文件、键盘、屏幕、文件指针或网络
	- 进程将文件描述符和对应的数据流保存在描述符表中
		- 描述符表中，文件描述符0、1、2分别对应标准输入、标准输出和标准错误。
		- 三大默认数据流的位置是固定的，但是其指向的数据流可以改变
	- 进程内部也可以进行重定向
	- 进程内部每打开一个文件，描述符表就会新注册一项
| #   | 数据流     |
| --- | ---------- |
| 0   | 键盘       |
| 1   | 屏幕       |
| 2   | 屏幕       |
| 3   | 数据库连接 |
| 4   | guitar.mp3 | 
- fileno()返回描述符号
```C
FILE *my_file = fopen("guitar.mp3", "r";)
int descriptor = fileno(my_file);
```
- dup2()复制数据流
	- 参数1表示被复制的数据流的描述符
	- 参数2表示复制数据流的位置描述符
| #   | 数据流     |
| --- | ---------- |
| 0   | 键盘       |
| 1   | 屏幕       |
| 2   | 屏幕       |
| 3   | guitar.mp3 |
| 4   | guitar.mp3 |
```C
dup2(4,3);
```
- pipe()建立管道
	- 创建两条相连的数据流，并加载到描述符表中
	- 使用数组作为参数，其中数组第0项表示管道读取端，第一项表示管道写入端
	- 通常创建管道并创建子进程后，双方都应该关闭一个自己不需要的管道读写端以节约资源
| #   | 数据流     |
| --- | ---------- |
| 0   | 键盘       |
| 1   | 屏幕       |
| 2   | 屏幕       |
| 3   | 管道读取端 |
| 4   | 管道写入端 |
```C
int fd[2];
if(pipe(fd) == -1){
	error("Can't create the pipe");
}
```
- 管道
	- 部分操作系统中，管道基于文件
		- 此时也称其为有名管道或FIFO
		- 通过mkfifo()使用有名管道
	- 读取空的管道时，程序会持续等待管道中出现内容
		- 子进程结束时，管道会关闭
		- 此时父进程收到EOF（End Of File），fgets()或scanf()返回false或0，并通过这样结束循环
	- 管道不能够双向通信，不过可以创建两个管道
- 打开浏览器
```sh
# Windows
cmd /c start www.baidu.com

# Linux
x-www-browser 'www.baidu.com' &

# Mac
open 'www.baidu.com'
```

#### 信号
- 信号控制程序
	- 信号到来时，进程停止任务，转而处理信号
	- 进程会查看内部的信号映射表
		- 表中每个信号对应着一个信号处理器函数
	- 按下Ctrl+C，发送SIGINT信号
		- SIGINT信号的默认处理器为调用exit()
- 替换信号处理器
	- 使用sigaction()通知操作系统替换了信号处理器
```C
int catch_signal(int sig, void (*handler)(int)){
	struct sigaction action;
	action.sa_handler = handler;
	sigemptyset(&action.sa_mask);
	action.sa_flags = 0;
	return sigaction(sig, &action, NULL);
}

void diediedie(int sig){
	// 这里只是出于演示效果
	// 这么做要小心，故障可能是因为标准输出无法使用
	puts("Goodbye cruel world...\n");
	exit(1);
}

catch_signal(SIGINT, diediedie);
```
- kill命令
	- `kill {pid}`杀死进程。原理是向进程发送信号
		- 默认：发送SIGTERM
		- -INT：发送SIGINT
		- -SEGV：发送SIGSEGV
		- -KILL：发送SIGKILL，进程不能忽略这个信号，代码也无法捕抓，一定可以杀死进程
- raise()
	- 向自己发送信号
	- 参数1为信号名
- alarm()或sleep()
	- 通过在特定时间后发送信号实现
	- 一个进程只有一个定时器
	- 重复设置定时器（调用这些函数）会重置定时器
	- alram()和sleep()不要同时使用
- 特殊的符号（作为sigaction变量使用）
	- SIG_DFL
		- 代表还原默认方式处理信号
	- SIG_IGN
		- 代表忽略这个信号


## #11 网络


## #12 线程


## #13 其他
