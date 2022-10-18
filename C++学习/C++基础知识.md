### Hello World

vim ~/code/Cpp/helloWorld.cc

```c++
#include <iostream>

using namespace std;

int main()
{
    cout << "Hello World!" << endl;
    return 0;
}
```

### 单行注释和多行注释

-  单行

  //

- 多行

  /*

  */

### 变量

作用：给一段指定的内存空间起名，方便操作这段内存

语法：`数据类型 变量名 = 初始值;`

示例：

```c++
#include <iostream>
using namespace std;

int main()
{
    // 变量的定义
    int a = 10;
    cout << "a = " << a << endl;
    return 0;s
}
```

### 常量

作用：用于记录程序中不可更改的数据

C++定义常量的两种方式：

1. \#define 宏常量：`#define 常量名 常量值`
2. const修饰的变量：`const 数据类型 常量名 = 常量值`

示例：

```c++
#include <iostream>
using namespace std;

#define day 7

int main()
{
    cout << day << endl;
    
    const int pi = 3.14;
    cout << pi << endl;
    return 0;
}
```

## 数据类型

### 整形

short、int、long、long long

### 数据类型-sizeof关键字

作用：利用sizeof关键字可以统计数据类型所占内存大小

语法：`size(数据类型/变量)`

示例：

```c++
#include <iostream>
using namespace std;

int main()
{
    short num = 1;
    cout << sizeof(num) << endl;
    cout << sizeof(short) << endl;
    return 0;
}
```

### 数据类型-实型（浮点型）

作用：用于表示小数

浮点型变量分为两种：

1. 单精度 float
2. 双精度double

### 数据类型-字符型

作用：字符变量用于显示单个字符

语法：`char ch = 'a'`

> 注意1：在显示字符型变量时，用单引号括起来，不要用双引号
>
> 注意2：单引号只能有一个字符，不可以是字符串

### 字符串型

作用：用于表示一串字符

两种风格：

1. C语言风格：`char 变量名[] = "字符串";`
2. C++风格：`string 变量名 = "字符串";`

示例：

```c++
#include <iostream>
#include <string>
using namespace std;

int main()
{
    string str = "Hello World";
    cout << str << endl;
    return 0;
}
```

### 布尔类型 bool

作用：布尔数据类型代表真或假的值

bool类型只有两个值：

- true
- false

### 数据的输入

作用：用于从键盘获取数据

关键字：`cin`

## 运算符

### 算术运算符

#### 加减乘除

作用：用于处理四则运算

#### 取模运算

求余数

#### 递增递减

递增：++

递减：--

### 赋值运算

赋值 = 

加等于 +=

减等于 -=

....

### 比较运算

相等 ==

不等于 != 

小于 < 

大于 >

小于等于 <=

大于等于 >=

### 逻辑运算符

作用：用于根据表达式的值返回真值或假值

! 非

&& 与

|| 或

## 程序流程结构

### 选择结构

if (条件)

​	执行语句

if (条件)

​	执行语句

else if (条件1)

​	执行语句1

else 

​	执行语句2



### 三目运算符

作用：通过三目运算符实现简单的判断

语法：`表达式1 ? 表达式2 : 表达式3 `

### switch

switch(值)
{

​	case 值1:

​			语句1

​	case 值2:

​			语句2

​	default:

​			语句

}

### 循环结构

#### while语句

while(条件)

{

​	语句

}



#### do...while语句

do

{

​	语句

}while(条件)

#### for循环

for (初始状态; 判断条件; 最终范围)

{

​	语句	

}

### 跳转语句

break:直接跳出当前循环

continue：跳过当前判断条件，执行下一次循环

goto：无条件跳转语句





## 数组

所谓数组，就是一个集合，里面存放了相同类型的数据元素

### 一维数组

int arr[]

### 二维数组

int arr\[][]

##  函数

作用：将一段经常使用的代码封装起来，减少重复代码

### 函数的定义

函数的定义一般5个步骤：

1. 返回值类型
2. 函数名
3. 参数表列
4. 函数体语句
5. return 表达式

 ## 指针

### 指针的基本概念

作用：可以通过指针间接访问内存

- 内存编号是从0开始记录的，一般用十六进制数字表示
- 可以利用指针变量保存地址

### 指针变量的定义和使用

指针变量定义语法：`数据类型 *变量名`

示例：

```c++
#include <iostream>
using namespace std;

int main()
{
    int a = 10;
    int * p = &a;
    cout << "a's mem address = " << p << endl;
    cout << "a's value = " << *p << endl;
    return 0;
}
```

### 空指针和野指针

空指针：指针变量指向内存编号为0的空间

用途：初始化指针变量

注意：空指针指向的内存是不可以访问的



野指针：指针变量指向非法的内存空间

## 结构体

结构体属于用户自定义的数据类型，允许用户存储不同的数据类型。

### 结构体的定义和使用

语法：`struct 结构体名{结构体成员列表}`

通过结构体创建变量的方式有三种：

- struct结构体名 变量名
- struct结构体名 变量名 = {成员1值， 成员2值}
- 定义结构体时顺便创建变量

