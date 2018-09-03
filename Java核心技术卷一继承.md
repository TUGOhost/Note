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

