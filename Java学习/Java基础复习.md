# Java基础
## 数据类型
### 包装类型
八个基本类型：
- Boolean/boolean/1
- Byte/byte/8
- Character/char/16
- Short/short/16
- Integer/int/32
- Float/float/32
- Long/long/64
- Double/double/64
基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成。
> Integer x = 2;  // 装箱
> int y = x;  // 拆箱
### 缓冲池
new Integer(123)与Integer.valueOf(123)的区别在于：
- new Integer(123)每次都会新建一个对象
- Integer.valueOf(123)会使用缓存池中的对象，多次调用会取得同一个对象的引用。
```java
Integer x = new Integer(123);
Integer y = new Integer(123);
System.out.println(x == y);  // false
Integer z = Integer.valueOf(123);
Integer k = Integer.valueOf(123);
System.out.println(z == k);  // true
```
valueOf()方法的实现比较简单，就是先判断是否在缓存池中，如果在的话就直接返回缓存池的内容。
```java
public static Integer valueOf(int i){
  if (i >= IntegerCache.low && i <= IntegerCache.high){
    return IntegerCache.cache[i + (-IntegerCache.low)];
  }
  return new Integer(i);
}
```
IntegerCache源码：
```java
private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```
##  String
### 概览
String被声明为final，因此它不可被继承。
```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
```
内部使用char数组存储数据，该数组被声明为final，这意味着value数组初始化之后就不能再引用其他数组。并且String内部没有改变value数组的方法，因此可以保证String不可变。
### 不可变的好处
1. 可以缓存hash值
2. String Pool的需要
如果一个String对象已经被创建过了，那么就会从String Pool中取得引用。只有String是不可变的，才可能使用String Pool。
3. 安全性
String经常作为参数，String不可变性可以保证参数不可变。
4. 线程安全
String不可变性天生具备线程安全，可以在多个线程中安全地使用。
### String，StringBuffer and StringBuilder
1. 可变性
- String不可变
- StringBuffer和StringBuilder可变
2. 线程安全
- String不可变，因此是线程安全的
- StringBuilder不是线程安全的
- StringBuffer是线程安全的，内部使用synchronized进行同步
### String.intern()
使用String.intern()可以保证相同用内容的字符串变量引用同一的内存对象。
下面示例中，s1和s2采用new String()的方式新建了两个不同对象，而s3是通过s1.intern()方法取得一个对象引用。intern()首先把s1引用的对象放到String Pool（字符串常量池）中，然后返回这个对象引用。因此s3和s1引用的是同一个字符串常量池的对象。
```java
String s1 = new String("aaa");
String s2 = new String("aaa");
System.out.println(s1 == s2); // false
String s3 = s1.intern();
System.out.println(s1.intern() == s3); // true
```
如果是采用"bbb"这种使用双引号的形式创建字符串实例，会自动地将新建的对象放入String Pool中。
```java
String s4 = "bbb";
String s5 = "bbb";
System.out.println(s4 == s5); // true
```
在Java 7之前，字符串常量池被放在运行时常量池中，它属于永久代。而在Java 7，字符串常量池被移到Native Method中。这是因为永久代的空间有限，在大量使用字符串的场景下会导致OutOfMemoryError错误。
## 运算
### 参数传递
### 隐式类型转换
### switch
## 继承
### 访问权限
Java中有三个访问权限修饰符：private、protected以及public，如果不加访问修饰符，表示包级可见。
可以对类或类中的成员（字段以及方法）加上访问修饰符。
- 类可见表示其他类可以用这个类创建实例对象。
- 成员可见表示其他类可以用这个类的实例对象访问到该对象；

protected用于修饰成员，表示在继承体系中成员对于子类可见，但是这个访问修饰符对于类没有意义。
设计良好的模块会隐藏所有的实现细节，把它的API与它的实现清晰地隔离开来。模块之间只能通过他们的API进行通信，一个模块不需要知道其他模块的内部工作情况，这个概念被成为信息隐藏或封装。因此访问权限应当尽可能地使每个类或者成员不被外界访问。
如果子类地方法重写了父类地方法，那么子类中该方法地访问级别不允许低于夫类的访问级别。这是为了确保可以使用父类实例的地方都可以使用子类实例，也就是确保满足里氏替换原则。
字段绝不能是公有的，因为这么做的话就失去了对这个字段修改行为的控制，客户端可以对其随意修改。例如下面的例子，AccessExample拥有id共有字段，如果在某个时刻，我们想要使用int去存储id字段，那么就需要去修改所有的客户端代码。
```java
public class AccessExample{
    public String id;
}
```
可以使用公有的getter和setter方法来替换公有字段，这样的话就可以控制对字段的修改行为。
```java
public class AccessExample{
    private int id;
    public String getId(){
        return id + "";
    }
    public void setId(String id){
        this.id = Integer.valueOf(id);
    }
}
```
但是也有例外，如果是包级私有的类或者私有的嵌套类，那么直接暴露成员不会有特别大的影响。
```java
public class AccessWithInnerClassExample{
    private class InnerClass{
        int x;
    }
    private AccessWithInnerClassExample(){
        innerClass = new InnerClass();
    }
    public int getValue(){
        return innerClass.x;
    }
}
```
### 抽象类与接口类
#### 1.抽象类
抽象类和抽象方法都使用abstrac关键字进行声明。抽象类一般会包含抽象方法，抽象方法一定位于抽象类中。
抽象类和普通类最大的区别是，抽象列不能被实例化，需要继承抽象类才能实例化其子类。
```java
public abstract classs AbstractClassExample{
    protected int x;
    private int y;
    public abstract void func1();
    public void func2(){
        System.out.println("func2");
    }
}
```
```java
public class AbstractExtendClassExample extends AbstractClassExample{
    @Override
    public void func1(){
        System.out.println("funct1");
    }
}
```
#### 2.接口
接口是抽象类的延伸，在Java 8之前，他可以看成是一个完全抽象的类，也就是说它不能有任何的方法实现。
从Java 8开始，接口也可以拥有默认的方法实现，这是因为不支持默认方法的接口的维护成本太高了。在Java 8之前，如果一个接口想要添加新的方法，那么要修改所有实现该接口的类。
接口的成员（字段 + 方法）默认都是public的，并且不允许定义为private或者protected。
接口的字段默认都是static和final的。
```java
public interface InterfaceExample{
    void func1();
    default void func2(){
        System.out.println("func2");
    }
    int x = 123;
}
```
```java
public class InterfaceImplementExample implements InterfaceExample {
    @Overrid
    public void func1(){
        System.out.println("func1");
    }
}
```
#### 3.比较
- 从设计层面上看，抽象类提供了一种IS-A关系，那么就必须满足里氏替换原则，即子类对象必须能够替换掉所有父类对象。而接口更像是一种LIKE-A关系，它只是提供一种方法实现契约，并不要求接口和实现接口的类具有IS-A关系。
- 从使用上看，一个类可以实现多个接口，但是不能继承多个抽象类。
- 接口的字段只能是static和final类型的，而抽象类的字段没有这种限制。
- 接口的成员只能是public的，而抽象类的成员可以有多种访问权限。
#### 4.使用选择
使用接口：
- 需要让不想关的类都实现一个方法，例如不相关的类都可以实现Compareable接口中的compareTo()方法；
- 需要使用多重继承。
使用抽象类：
- 需要在几个相关的类中共享代码。
- 需要能控制继承来的成员的访问权限，而不是都为public
- 需要继承非静态和非常量字段。
在很多情况下，接口优先于抽象类，因为接口没有抽象类严格的类层次结构要求，可以灵活地为一个类添加行为。并且从Java 8开始，接口也可以有默认地方法实现，使得修改接口的成本也变的很低。
### super
- 访问父类的构造函数：可以使用super()函数访问父类的构造函数，从而委托父类完成一些初始化的工作。
- 访问父类的成员：如果子类重写了父类的中某个方法的实现，可以通过使用super关键字来引用父类的方法实现。
### 重写和重载
- 重写（override）存在于继承体系中，指子类实现了一个父类在方法声明上完全相同的一个方法。子类的返回值类型要等于或者小于父类的返回值；
- 重载（overload）存在于同一个类中，指一个方法已经存在的方法名称上相同，但是参数类型、个数、顺序至少有一个不同。应该注意的是，返回值不同，其他都是相同不算是重载。
## Object通用方法
### 概览
### equals()
### hashCode()
### toString()
### clone()
## 关键字
### final
### static

## 反射
![](/image/Java反射机制.png)

## 反射是什么

反射的作用用一句简单的话来讲就是可以对代码进行操作的代码，这个特性经常在被用于创建JavaBean中，通常造轮子的人会用到这个特性，而应用程序员用到这个特性的场景则较少。

能够分析类能力的程序就叫做反射，简单来说就是可以对代码进行操作的代码。反射机制的功能极为强大，可以用来：

- 在运行时分析类的能力
- 在运行时查看对象
- 实现通用的数组操作代码
- 利用Method对象来实现方法

 ## 从获取Class类开始

在程序运行期间，Java运行时系统始终为所有的对象维护一个被称为运行时的类型标识。这个信息跟踪着每个对象所属的类。这个类的获取方式有以下三种：

1. 使用Object类中的`getClass()`方法来`返回一个Class类的实例`：

   ```java
   User user;
   Class userClass  = user.getClass();
   ```

2. 我们可以使用Class类的`getName()`方法来`获取包含包名在内的类名`。同样的，在已知这个名字的情况下，我们可以使用静态方法`forName()获得类名对应的Class对象`：

```java
Random generator = new Random();
Class randomClass = generator.getClass();
//className = "java.util.Random"
String className = randomClass.getName();
//第二种方式获取
Class newRandomClass = Class.forName(className);
```

3. 获得Class类对象的第三种方法很简单，如果T是任意的Java类型（或者void关键字），T.class将代表匹配的类对象。例如：

```java
Class randomClass = Random.class;
Class intClass = int.class;
Class doubleClass = Double[].class;
```

如果我们想要创建一个类的实例，可以使用`newInstance()`方法来动态创建：

```java
String s = "java.util.Random";
Object m = Class.forName(s).newInstance();
```

## 构造函数的反射

获得构造函数的方法

```java
    //根据指定参数获得public构造器
    Constructor getConstructor(Class[] params);
    //获得public的所有构造器
    Constructor[] getConstructors();
    //根据指定参数获得public和非public的构造器
    Constructor getDeclaredConstructor(Class[] params);
    //获得public的所有构造器 
    Constructor[] getDeclaredConstructors();
```

看这些方法如何使用，先来个Student类供我们反射使用

```java
public class Student {

    private static String TAG = Student.class.getSimpleName();
    public int age;
    private String name;

    public Student() {
        age = 20;
        name = "小明";
    }

    public Student(int age, String name) {
        Log.e(TAG, "Student: " + "age " + age + " name " + name);
    }

    public void StudentA() {
        Log.e(TAG, "StudentA: ");
    }

    public void StudentA(int age) {
        Log.e(TAG, "StudentA: " + "age " + age);
    }

    public void StudentA(int age, String name) {
        Log.e(TAG, "StudentA: " + "age " + age + " name " + name);
    }
}
```

## 利用反射分析类的能力

在java.lang.reflect包(反射库)中有三各类`Field`,`Method`和`Constructor`分别用于描述类的域，方法和构造器。这三个类都有一个叫做`getName()`的方法，用于返回项目的名称。Filed类有一个`getType()`方法，用于返回描述域所属类型的Class对象。Method和Constructor类有能够报告参数类型的方法，Method类还有一个可以报告返回类型的方法。

这三个类还有一个叫做`getModifiers()`的方法，它将返回一个整型数值，用不同的位开关描述public和static这样的修饰符使用情况。另外，还可以利用java.lang.reflect包中的Modifier类的静态方法分析`getModifiers()`返回的整型数值。例如，可以使用Modifier类中的`isPublic()`，`isPrivate()`或`isFinal()`判断方法或构造器是否是public,private或final。我们需要做的全部工作就是调用Modifier类的相应方法，并对返回的整数数值进行分析，另外，还可以利用`Modifier.toString()`方法将修饰符打印出来。

Class类中的`getFields()`，`getMethods()`和`getConstructors()`方法将分别返回类提供的public域、方法和构造器数组，其中包括超类的公有成员。Class类的`getDeclareFieds()`，`getDeclareMethods()`和`getDeclareConstructors()`方法将分别返回类中声明的全部域、方法和构造器，其中包括私有和受保护成员，但不包括超类的成员。

下面我们来编写一个程序可以做到输入类名，然后打印出这个类的全部信息的作用：

```java
package com.reflect.test;

import com.sun.org.apache.xpath.internal.operations.Mod;

import java.io.File;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.lang.reflect.Modifier;
import java.util.Scanner;

public class ReflectionTest {

    public static void main(String[] args) {
        String name;
        if(args.length > 0){
            name = args[0];
        }else{
            Scanner in = new Scanner(System.in);
            System.out.println("请输入类名:");
            name = in.next();
        }

        try{
            Class c1 = Class.forName(name);
            Class superclass = c1.getSuperclass();
            String modifiers = Modifier.toString(c1.getModifiers());
            if(modifiers.length() > 0){
                System.out.println(modifiers + " ");
            }
            System.out.println("class"+name);
            if(superclass != null && superclass != Object.class){
                System.out.println("extends"+superclass.getName());
            }
            System.out.println("\n{\n");
            printConstructors(c1);
            System.out.println();
            printMethods(c1);
            System.out.println();
            printFields(c1);
            System.out.println("}");
        }catch (ClassNotFoundException e){
            e.printStackTrace();
        }
        System.exit(0);
    }

    private static void printFields(Class c1) {
        Field[] fields = c1.getDeclaredFields();

        for(Field field : fields){
            Class type = field.getType();
            String name = field.getName();
            System.out.println("   ");
            String modifiers = Modifier.toString(field.getModifiers());
            if(modifiers.length() > 0){
                System.out.println(modifiers + " ");
            }
            System.out.println(type.getName() + " " + name + ";");
        }
    }

    private static void printMethods(Class c1) {
        Method[] methods = c1.getDeclaredMethods();

        for(Method method : methods){
            Class returnType = method.getReturnType();
            String name = method.getName();
            System.out.println("  ");
            String modifiers = Modifier.toString(method.getModifiers());
            if(modifiers.length() > 0){
                System.out.println(modifiers + " ");
            }
            System.out.println(returnType.getName()+" "+name+"(");
            Class[] paramTypes = method.getParameterTypes();
            for(int j = 0; j < paramTypes.length; j++){
                if(j > 0){
                    System.out.println(",");
                }
                System.out.println(paramTypes[j].getName());
            }
            System.out.println(");");
        }
    }

    private static void printConstructors(Class c1) {
        Constructor[] constructors = c1.getDeclaredConstructors();
        for(Constructor constructor : constructors){
            String name = constructor.getName();
            System.out.println("  ");
            String modifiers = Modifier.toString(constructor.getModifiers());
            if(modifiers.length() > 0){
                System.out.println(modifiers + " ");
            }
            System.out.println(name + "(");
            Class[] paramTypes = constructor.getParameterTypes();
            for(int j = 0; j < paramTypes.length; j++){
                if(j > 0){
                    System.out.println(",");
                }
                System.out.println(paramTypes[j].getName());

            }
            System.out.println(");");
        }
    }
}

```

输入`java.long.Double`

回显：

```java
请输入类名:
java.lang.Double
public final 
Disconnected from the target VM, address: '127.0.0.1:51190', transport: 'socket'
classjava.lang.Double
extendsjava.lang.Number

{

  
public 
java.lang.Double(
double
);
  
public 
java.lang.Double(
java.lang.String
);

  
public 
boolean equals(
java.lang.Object
);
  
public static 
java.lang.String toString(
double
);
  
public 
java.lang.String toString(
);
  
public 
int hashCode(
);
  
public static 
int hashCode(
double
);
  
public static 
double min(
double
,
double
);
  
public static 
double max(
double
,
double
);
  
public static native 
long doubleToRawLongBits(
double
);
  
public static 
long doubleToLongBits(
double
);
  
public static native 
double longBitsToDouble(
long
);
  
public volatile 
int compareTo(
java.lang.Object
);
  
public 
int compareTo(
java.lang.Double
);
  
public 
byte byteValue(
);
  
public 
short shortValue(
);
  
public 
int intValue(
);
  
public 
long longValue(
);
  
public 
float floatValue(
);
  
public 
double doubleValue(
);
  
public static 
java.lang.Double valueOf(
java.lang.String
);
  
public static 
java.lang.Double valueOf(
double
);
  
public static 
java.lang.String toHexString(
double
);
  
public static 
int compare(
double
,
double
);
  
public static 
boolean isNaN(
double
);
  
public 
boolean isNaN(
);
  
public static 
boolean isFinite(
double
);
  
public static 
boolean isInfinite(
double
);
  
public 
boolean isInfinite(
);
  
public static 
double sum(
double
,
double
);
  
public static 
double parseDouble(
java.lang.String
);

   
public static final 
double POSITIVE_INFINITY;
   
public static final 
double NEGATIVE_INFINITY;
   
public static final 
double NaN;
   
public static final 
double MAX_VALUE;
   
public static final 
double MIN_NORMAL;
   
public static final 
double MIN_VALUE;
   
public static final 
int MAX_EXPONENT;
   
public static final 
int MIN_EXPONENT;
   
public static final 
int SIZE;
   
public static final 
int BYTES;
   
public static final 
java.lang.Class TYPE;
   
private final 
double value;
   
private static final 
long serialVersionUID;
}

Process finished with exit code 0

```

![](/image/125.png)

`不积跬步无以至千里`

## 思维导图

![](/image/17.jpg)

扩展阅读：

https://blog.csdn.net/zhangqiluGrubby/article/details/60874013

https://www.daidingkang.cc/2017/07/18/java-reflection-annotations/

https://www.javazhiyin.com/17014.html

https://www.javazhiyin.com/17008.html
## 异常
## 泛型
## 注解
### 概念及作用

1. 概念

- 注解即元数据,就是源代码的元数据
- 注解在代码中添加信息提供了一种形式化的方法,可以在后续中更方便的 使用这些数据
- Annotation是一种应用于类、方法、参数、变量、构造器及包声明中的特殊修饰符。它是一种由JSR-175标准选择用来描述元数据的一种工具。

2. 作用

- 生成文档
- 跟踪代码依赖性，实现替代配置文件功能,减少配置。如Spring中的一些注解
- 在编译时进行格式检查，如@Override等
- 每当你创建描述符性质的类或者接口时,一旦其中包含重复性的工作，就可以考虑使用注解来简化与自动化该过程。

### 什么是java注解？

在java语法中，使用`@`符号作为开头，并在@后面紧跟注解名。被运用于类，接口，方法和字段之上，例如：

```java

@Override
void myMethod() { 
......
}
```

这其中@Override就是注解。这个注解的作用也就是告诉编译器，myMethod()方法覆写了父类中的myMethod()方法。

### java中内置的注解

java中有三个内置的注解：

- **@Override:**表示当前的方法定义将覆盖超类中的方法，如果出现错误，编译器就会报错。

- **@Deprecated:**如果使用此注解，编译器会出现警告信息。

- **@SuppressWarnings:**忽略编译器的警告信息。

本文不在阐述三种内置注解的使用情节和方法，感兴趣的请看[这里](http://www.jianshu.com/p/89c07ce0c99c)

### 元注解

> 自定义注解的时候用到的，也就是自定义**注解的注解**；（这句话我自己说的，不知道对不对）

元注解的作用就是负责注解其他注解。`Java5.0`定义了4个标准的meta-annotation类型，它们被用来提供对其它 annotation类型作说明。

#### `Java5.0`定义的4个元注解：

1. @Target
2. @Retention
3. @Documented
4. @Inherited

> java8加了两个新注解，后续我会讲到。

这些类型和它们所支持的类在java.lang.annotation包中可以找到。

#### @Target

> @Target说明了Annotation所修饰的对象范围：Annotation可被用于  packages、types（类、接口、枚举、Annotation类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数）。在Annotation类型的声明中使用了target可更加明晰其修饰的目标。

**作用：用于描述注解的使用范围（即：被描述的注解可以用在什么地方）**

**取值(ElementType)有：**

| 类型           | 用途                                      |
| -------------- | ----------------------------------------- |
| CONSTRUCTOR    | 用于描述构造器                            |
| FIELD          | 用于描述域                                |
| LOCAL_VARIABLE | 用于描述局部变量                          |
| METHOD         | 用于描述方法                              |
| PACKAGE        | 用于描述包                                |
| PARAMETER      | 用于描述参数                              |
| TYPE           | 用于描述类、接口(包括注解类型) 或enum声明 |

比如说这个注解表示只能在方法中使用：

```java

@Target({ElementType.METHOD})
public @interface MyCustomAnnotation {

}

//使用
public class MyClass {
   @MyCustomAnnotation
   public void myMethod()
   {
    ......
   }
}
```

#### @Retention

> @Retention定义了该Annotation被保留的时间长短：某些Annotation仅出现在源代码中，而被编译器丢弃；而另一些却被编译在class文件中；编译在class文件中的Annotation可能会被虚拟机忽略，而另一些在class被装载时将被读取（请注意并不影响class的执行，因为Annotation与class在使用上是被分离的）。使用这个meta-Annotation可以对  Annotation的“生命周期”限制。

**作用：表示需要在什么级别保存该注释信息，用于描述注解的生命周期（即：被描述的注解在什么范围内有效）**

**取值（RetentionPoicy）有：**

| 类型    | 用途                             | 说明                             |
| ------- | -------------------------------- | -------------------------------- |
| SOURCE  | 在源文件中有效（即源文件保留）   | 仅出现在源代码中，而被编译器丢弃 |
| CLASS   | 在class文件中有效（即class保留） | 被编译在class文件中              |
| RUNTIME | 在运行时有效（即运行时保留）     | 编译在class文件中                |

使用示例：

```java

/***
 * 字段注解接口
 */
@Target(value = {ElementType.FIELD})//注解可以被添加在属性上
@Retention(value = RetentionPolicy.RUNTIME)//注解保存在JVM运行时刻,能够在运行时刻通过反射API来获取到注解的信息
public @interface Column {
    String name();//注解的name属性
}
```

#### @Documented

> @Documented用于描述其它类型的annotation应该被作为被标注的程序成员的公共API，因此可以被例如javadoc此类的工具文档化。Documented是一个标记注解，没有成员。

**作用：将注解包含在javadoc中**

示例：

```java

java.lang.annotation.Documented
@Documented
public @interface MyCustomAnnotation { //Annotation body}
```

#### @Inherited

- 是一个标记注解

- 阐述了某个被标注的类型是被继承的

- 使用了@Inherited修饰的annotation类型被用于一个class,则这个annotation将被用于该class的子类
  @Inherited annotation类型是被标注过的class的子类所继承。类并不从实现的接口继承annotation,方法不从它所重载的方法继承annotation

- 当@Inherited  annotation类型标注的annotation的Retention是RetentionPolicy.RUNTIME，则反射API增强了这种继承性。如果我们使用java.lang.reflect去查询一个@Inherited   annotation类型的annotation时，反射代码检查将展开工作：检查class和其父类，直到发现指定的annotation类型被发现，或者到达类继承结构的顶层。

**作用：允许子类继承父类中的注解**

示例，这里的MyParentClass 使用的注解标注了@Inherited，所以子类可以继承这个注解信息：

```java

java.lang.annotation.Inherited
@Inherited
public @interface MyCustomAnnotation {
}

@MyCustomAnnotation
public class MyParentClass { 
  ... 
}

public class MyChildClass extends MyParentClass { 
   ... 
}
```

### 自定义注解

#### 格式

```java

public @interface 注解名{
  定义体
}
```

#### 注解参数的可支持数据类型:

- 所有基本数据类型(int,float,double,boolean,byte,char,long,short)
- String 类型
- Class类型
- enum类型
- Annotation类型
- 以上所有类型的数组

#### 规则

- 修饰符只能是public 或默认(default)
- 参数成员只能用基本类型byte,short,int,long,float,double,boolean八种基本类型和String,Enum,Class,annotations及这些类型的数组
- 如果只有一个参数成员,最好将名称设为”value”
- 注解元素必须有确定的值,可以在注解中定义默认值,也可以使用注解时指定,非基本类型的值不可为null,常使用空字符串或0作默认值
- 在表现一个元素存在或缺失的状态时,定义一下特殊值来表示,如空字符串或负值

#### 示例:

```java

/**
 * test注解
 * @author ddk
 *
 */ 
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface TestAnnotation {
    /**
     * id
     * @return
     */
    public int id() default -1;
    /**
     * name
     * @return
     */
    public String name() default "";
}
```

### 注解处理器类库

> java.lang.reflect.AnnotatedElement

Java使用Annotation接口来代表程序元素前面的注解，该接口是所有Annotation类型的父接口。除此之外，Java在java.lang.reflect  包下新增了AnnotatedElement接口，该接口代表程序中可以接受注解的程序元素，该接口主要有如下几个实现类：

- 　Class：类定义
- 　Constructor：构造器定义
- 　Field：累的成员变量定义
- 　Method：类的方法定义
- 　Package：类的包定义

java.lang.reflect 包下主要包含一些实现反射功能的工具类，实际上，java.lang.reflect  包所有提供的反射API扩充了读取运行时Annotation信息的能力。当一个Annotation类型被定义为运行时的Annotation后，该注解才能是运行时可见，当class文件被装载时被保存在class文件中的Annotation才会被虚拟机读取。

AnnotatedElement 接口是所有程序元素（Class、Method和Constructor）的父接口，所以程序通过反射获取了某个类的AnnotatedElement对象之后，程序就可以调用该对象的如下四个个方法来访问Annotation信息：

- 方法1： T getAnnotation(Class annotationClass): 返回改程序元素上存在的、指定类型的注解，如果该类型注解不存在，则返回null。
- 方法2：Annotation[] getAnnotations():返回该程序元素上存在的所有注解。
- 方法3：boolean is AnnotationPresent(Class<?extends Annotation> annotationClass):判断该程序元素上是否包含指定类型的注解，存在则返回true，否则返回false.
- 方法4：Annotation[]  getDeclaredAnnotations()：返回直接存在于此元素上的所有注释。与此接口中的其他方法不同，该方法将忽略继承的注释。（如果没有注释直接存在于此元素上，则返回长度为零的一个数组。）该方法的调用者可以随意修改返回的数组；这不会对其他调用者返回的数组产生任何影响。

#### 注解处理器示例:

```java

/***********注解声明***************/

/**
 * 水果名称注解
 * @author peida
 *
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitName {
    String value() default "";
}

/**
 * 水果颜色注解
 * @author peida
 *
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitColor {
    /**
     * 颜色枚举
     * @author peida
     *
     */
    public enum Color{ BULE,RED,GREEN};
    
    /**
     * 颜色属性
     * @return
     */
    Color fruitColor() default Color.GREEN;

}

/**
 * 水果供应者注解
 * @author peida
 *
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitProvider {
    /**
     * 供应商编号
     * @return
     */
    public int id() default -1;
    
    /**
     * 供应商名称
     * @return
     */
    public String name() default "";
    
    /**
     * 供应商地址
     * @return
     */
    public String address() default "";
}

/***********注解使用***************/

public class Apple {
    
    @FruitName("Apple")
    private String appleName;
    
    @FruitColor(fruitColor=Color.RED)
    private String appleColor;
    
    @FruitProvider(id=1,name="陕西红富士集团",address="陕西省西安市延安路89号红富士大厦")
    private String appleProvider;
    
    public void setAppleColor(String appleColor) {
        this.appleColor = appleColor;
    }
    public String getAppleColor() {
        return appleColor;
    }
    
    public void setAppleName(String appleName) {
        this.appleName = appleName;
    }
    public String getAppleName() {
        return appleName;
    }
    
    public void setAppleProvider(String appleProvider) {
        this.appleProvider = appleProvider;
    }
    public String getAppleProvider() {
        return appleProvider;
    }
    
    public void displayName(){
        System.out.println("水果的名字是：苹果");
    }
}

/***********注解处理器***************/
//其实是用的反射


public class FruitInfoUtil {
    public static void getFruitInfo(Class<?> clazz){
        
        String strFruitName=" 水果名称：";
        String strFruitColor=" 水果颜色：";
        String strFruitProvicer="供应商信息：";
        
        Field[] fields = clazz.getDeclaredFields();
        
        for(Field field :fields){
            if(field.isAnnotationPresent(FruitName.class)){
                FruitName fruitName = (FruitName) field.getAnnotation(FruitName.class);
                strFruitName=strFruitName+fruitName.value();
                System.out.println(strFruitName);
            }
            else if(field.isAnnotationPresent(FruitColor.class)){
                FruitColor fruitColor= (FruitColor) field.getAnnotation(FruitColor.class);
                strFruitColor=strFruitColor+fruitColor.fruitColor().toString();
                System.out.println(strFruitColor);
            }
            else if(field.isAnnotationPresent(FruitProvider.class)){
                FruitProvider fruitProvider= (FruitProvider) field.getAnnotation(FruitProvider.class);
                strFruitProvicer=" 供应商编号："+fruitProvider.id()+" 供应商名称："+fruitProvider.name()+" 供应商地址："+fruitProvider.address();
                System.out.println(strFruitProvicer);
            }
        }
    }
}

/***********输出结果***************/
public class FruitRun {

    /**
     * @param args
     */
    public static void main(String[] args) {
        
        FruitInfoUtil.getFruitInfo(Apple.class);
        
    }

}

====================================
 水果名称：Apple
 水果颜色：RED
 供应商编号：1 供应商名称：陕西红富士集团 供应商地址：陕西省西安市延安路89号红富士大厦
```

### Java 8 中注解新特性

- @Repeatable 元注解,表示被修饰的注解可以用在同一个声明式或者类型加上多个相同的注解（包含不同的属性值）

- @Native 元注解,本地方法 

- java8 中Annotation 可以被用在任何使用 Type 的地方

```java

 //初始化对象时
String myString = new @NotNull String();
//对象类型转化时
myString = (@NonNull String) str;
//使用 implements 表达式时
class MyList<T> implements @ReadOnly List<@ReadOnly T>{
...
}
//使用 throws 表达式时
public void validateValues() throws @Critical ValidationFailedException{
...
}
```

### 思维导图

![](/image/18.jpg)

### 扩展阅读

http://www.cnblogs.com/ITtangtang/p/3974531.html

www.jianshu.com/p/948549b92e0a

https://www.jianshu.com/p/89c07ce0c99c
## 特性
### Java各个版本的新特性
### Java与C++的区别
### JRE or JDK