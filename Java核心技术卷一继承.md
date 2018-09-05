# Java核心技术卷一继承

### super关键字

```java
public Manager(String name, double salary, int year, int month, int day){
	super(name, salary, year, month, day);
    bonus = 0;
}
```

这里的关键字super具有不同的含义。语句`super(n, s, year, month, day);`是“调用超类Employee中含有n、s、year、month和day参数的构造器”的简写形式。

由于Manager类的构造器不能访问Employee类的私有域，所以必须利用Employee类的构造器对这部分私有域进行初始化，我们可以通过super实现对超类构造器的调用。使用super调用构造器的语句必须是子类构造器的第一条语句。

注意区分`this`和`super`关键字。

关键字`this`的两个用途：

1. 引用yin'shi隐式参数
2. 调用该类其他的构造器

关键字`super`的两个用途：

1. 调用超类的方法
2. 调用超类的构造器

`调用构造器的语句只能作为另一个构造器的第一天语句`

### 理解方法调用

1. 编译器查看对象的声明类型和方法名
2. 接下来编译器将查看调用方法时提供的参数类型
3. 至此，编译器已获得需要调用的方法名字和参数类型

### 强制类型转换

在进行类型转换之前，先查看一下是否能够成功地转换。这个过程简单地使用instanceof操作符就可以实现。

### Object：所有类的超类

Object类是Java中所有类的始祖，在Java中每个类都是由他扩展而来的。

Object类中的equals方法用于监测一个对象是否等于另外一个对象。在Object类中，这个方法判断两个对象是否具有相同的引用。如果两个对象具有相同的引用，他们一定是相等的。

### 对象包装器与自动装箱

这些对象包装器类拥有很明显的名字：Integer、Long、Float、Double、Short、Byte、Character、Void和Boolean（前6个类派生于公共的超类Number）

### 反射

反射库（reflection library）提供了一个非常丰富且精心设计的工具集，以便编写能够动态操纵Java代码的程序。

能够分析类能力的程序成为反射（reflective）

反射机制可以用来：

- 在运行时分析类的能力
- 在运行时查看对象，例如，编写一个toString方法供所有类使用
- 实现通用的数组操作码
- 利用Method对象，这个对象很想C++中的函数指针

Object类中的getClass()方法将会返回一个Class类型的实例。

最常用的Class方法是getName。这个方法将返回类的名字。

Class类实际上是一个泛型类。

newInstance()方法可以用来动态地创建一个类的实例。

#### 捕获异常

当程序运行过程中发生错误时，就会“抛出异常”。抛出异常比终止程序要灵活得多，这是因为可以提供一个“捕获”异常的处理器（handler）对异常情况进行处理。

如果没有提供处理器， 程序就会终止，并在控制台上打印出一条信息，其中给出了异常的类型。可能在前面已经看到过一些异常报告，例如，偶然使用了null引用或者数组越界等。

异常有两种类型：未检查异常和已检查异常。对于已检查异常，编译器将会检查是否提供了处理器。然而，有很多常见的异常，例如，访问null引用，都属于未检查异常。编译器不会查看是否为这些错误提供了处理器。毕竟，应该精心地编写代码来避免这些错误的发生，而不要将精力花在编写异常处理器上。

#### 利用反射分析类的能力

在java.long.reflect包中有三个类Field、Method和Constructor分别用于描述类的域、方法和构造器。

Class类中地getFields、getMethods和getConstructors方法将分别返回类提供的public域、方法和构造器数组，其中包括超类的公有成员。Class类的getDeclareFields、getDeclareMethods和getDeclaredConstructors方法将分别返回类中声明的全部域、方法和构造器，其中包括私有和受保护成员，但不包括超类的成员。

### 继承的设计技巧

1. 将公共操作和域放在超类
2. 不要使用受保护的域
3. 使用继承实现“is-a”关系
4. 除非所有继承的方法都有意义，否则不要使用继承
5. 在覆盖方法时，不要改变预期的行为
6. 使用多态，而非类型信息
7. 不要过多地使用反射