# String
String表示字符串，Java中所有字符串的字面值都是String类的实例，例如"ABC"。字符串是常量，在定义后不能被改变，字符串缓冲区支持可变的字符串。因为String对象是不可变的，所以可以共享他们。例如：
> String str = "abc";

相当于
> char data[] = {'a','b','c'};
> String str = new String(data);

这里还有一些字符串的例子
```java
System.out.println("abc");
String cde = "cde";
System.ou.println("abc" + cde);
String c = "abc".substring(2,3);
String d = cde.substring(1,2);
```
String类提供了检查字符序列中单个字符的方法，比如有比较字符串，搜索字符串，提取子字符串，创建一个字符串的副本，字符串的大小写转换等。实例映射是基于`Character`类中指定的Unicode标准的。**Java语言提供了对字符串链接运算符的特别支持(+)**，该符号也可用于将其他类型转换成字符串。字符串的链接实际上是通过`StringBuffer`或`StringBuilder`的`append()`方法来实现的，字符串的转换通过`toString`方法来实现，该方法由Object类定义，并可被Java中的所有类继承。除非另有说明，传递一个空参数在这个类构造函数或方法会导致`NullPointerException`异常被抛出。**String表示一个字符串通过UTF-16(unicode)格式**，补充字符通过代理对（参见Character类的Unicode Character Representations获取更多的信息）表示。索引值参考字符编码单元，所以补充字符在String中占两个位置。
## 定义
```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence{}
```
从该类的声明中我们可以看出String是final类型的，表示该类不能被继承，同时该类实现了三个接口：`java.io.Serializable`、`Comparable<String>`、`CharSequence`
## 属性
```java
/** The value is used for character storage. */
    private final char value[];
```
这个是一个字符数组，并且是final类型，他用于存储字符串内容，从final这个关键字中我们可已看出，String的内容一旦被初始化了是不能被更改的。虽然有这样的例子：
> String s = "a";
> s = "b";

但是，这并不是对s的修改，而是重新指向了新的字符串，从这里我们也能知道，**String其实就是用char[]实现的**。
```java
/** Cache the hash code for the string */
    private int hash; // Default to 0
```
缓存字符串的hash Code，默认值为0
```java
/** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -6849794470754667710L;
/**
     * Class String is special cased within the Serialization Stream Protocol.
     *
     * A String instance is written into an ObjectOutputStream according to
     * <a href="{@docRoot}/../platform/serialization/spec/output.html">
     * Object Serialization Specification, Section 6.2, "Stream Elements"</a>
     */
    private static final ObjectStreamField[] serialPersistentFields =
        new ObjectStreamField[0];
```
因为`String`实现了`Serializable`接口，所以支持序列化和反序列化支持。Java的序列化机制是通过在运行时判断类的`serialVersion`来验证版本一致性的。在进行反序列化时，JVM会把传来的字节流中的`serialVersionUID`与本地相应实体类（类）的`serialVersionUID`进行比较，如果相同就认为是一致的，可以进行反序列化，否则就会出现序列化版本不一致的异常（`InvalidCastException`）。
## 构造方法
String类作为一个java.lang包中比较常用的类，自然有很多重载的构造方法，在这里介绍几种典型的构造方法。
### 1.使用字符数组、字符串构造一个String
我们知道，其实String就是使用字符数组（char[]）实现的。所以我们可以使用一个字符数组来创建一个String，那么这里值得注意的是，**当我们使用字符数组创建String的时候，会用到`Arrays.copyOf`方法和`Arrays.copyOfRange`方法。这两个方法是将原有的字符数组中的内容逐一的复制到String中的字符数组中**。同样，我们也可以用一个String类型的对象来初始化一个String。这里将直接`源String`中的`value`和`hash`两个属性直接赋值给`目标String`。因为String一旦定义之后是不可以改变的，所以也就是不用担心改变`源String`的值会影响到`目标String`的值。
> 当然，在使用字符数组来创建一个新的String对象的时候，不仅可以使用整个字符数组，也可以使用字符数组的一部分，只要多传入两个参数`int offset`和`int count`就可以了。

### 2.使用字符数组构造一个String
在Java中，String实例中保存有一个`char[]`字符数组，`char[]`字符数组是以unicode码来存储的，String和char以内存形式，byte是网络传输或者存储的序列化形式。所以在很多传输和存储的过程中需要将byte[]数组和String进行相互转化。所以，String提供了一系列重载的构造方法来将一个字符数组转化成String，提到byte[]和String之间的相互转换就不得不关注编码问题。`String(byte[] bytes,Charset charset)`**是通过Charset来解码指定的byte数组，将其解码成unicode的char[]数组，构造成新的String**。
> 这里的bytes字节流是使用charset进行编码的，想要将他转换成unicode的char[]数组，而又保证不出现乱码，那就要指定其解码方法。

同样使用字节数组来构造String也有很多种形式，按照是否指定解码方式分的话可以分为两种：
> String(byte bytes[]) String(byte bytes[], int offset, int length)
> String(byte bytes[], Charset charset)
> String(byte bytes[], String charsetName)
> String(byte bytes[], int offset, int length, Charset charset)
> String(byte bytes[], int offset, int length, String charsetName)
如果我们在使用byte[]构造Stringd的时候，使用的是下面这四种构造方法（带有`charsetName`或者`charset`参数）的一种的话，那么就会使用`StringCoding.decode`方法进行解码，使用的解码的字符集就是我们指定的`charsetName`或者`charset`。我们在使用byte[]构造String的时候，如果没有指明解码使用的字符集的话，那么`StringCoding`的`decode`方法首先调用系统的默认编码格式，如果没有指定编码格式则默认使用**ISO-8859-1**编码格式进行编码操作。
主要体现代码如下：
```java
static char[] decode(String charsetName, byte[] ba, int off, int len)
        throws UnsupportedEncodingException
    {
        StringDecoder sd = deref(decoder);
        String csn = (charsetName == null) ? "ISO-8859-1" : charsetName;
        if ((sd == null) || !(csn.equals(sd.requestedCharsetName())
                              || csn.equals(sd.charsetName()))) {
            sd = null;
            try {
                Charset cs = lookupCharset(csn);
                if (cs != null)
                    sd = new StringDecoder(cs, csn);
            } catch (IllegalCharsetNameException x) {}
            if (sd == null)
                throw new UnsupportedEncodingException(csn);
            set(decoder, sd);
        }
        return sd.decode(ba, off, len);
    }
```
### 3.使用StringBuffer和StringBuilder构造一个String
作为String的两个“兄弟”，StringBuffer和StringBuilder也可以当作构造String的参数。
```java
public String(StringBuffer buffer) {
        synchronized(buffer) {
            this.value = Arrays.copyOf(buffer.getValue(), buffer.length());
        }
    }

public String(StringBuilder builder) {
        this.value = Arrays.copyOf(builder.getValue(), builder.length());
    }
```
当然，这两个构造方法是很少用到的，至少我从来没有使用过，因为当我们有了StringBuffer或者StringBuilder对象之后可以直接使用他们的toString方法来得到String。关于效率问题，Java的官方文档有提到过说使用StringBuilder的toString方法发会更快一些，原因是StringBuffer的`toString`方法是synchronized的，在牺牲了效率的情况下保证了线程安全。
```java
// StringBuffer
@Override
public synchronized String toString() {
    if (toStringCache == null) {
        toStringCache = Arrays.copyOfRange(value, 0, count);
    }
    return new String(toStringCache, true);
}
// StringBuilder
@Override
public String toString() {
    // Create a copy, don't share the array
    return new String(value, 0, count);
}
```
### 4.一个特数的保护类型的构造方法
String除了提供了很多共有的供程序员使用的构造方法以外，还提供了一个保护类的构造方法，我们看下他是怎么样的：
```java
String(char[] value, boolean share) {
    // assert share : "unshared not supported";
    this.value = value;
}
```
从代码中我们可以看出，该方法和`String(char[] value)`有两点区别，第一个，该方法多了一个参数：`booleanshare`，其实这个参数在方法体中根本没被使用，也给了注释，目前不支持使用false，只使用了true。那么可以断定，**加入这个share的只是为了区分于String(char[] value)方法**，不加这个参数就没办法定义这个函数，只有参数不能进行重载。那么，第二个区别就是具体的方法实现不同。我们前面提到过，`String(char[] value)`方法在创建String的时候会用到`Arrays`的`copyOf`方法将value中的内容逐一复制到String当中，而这个`String(char[] value,boolean share)`方法则是直接将value的引用赋值给String的value。那么也就是说，这个方法构造出来的String和参数穿过来的`char[] value`共享同一个数组。那么，为什么Java会提供这样一个方法呢？首先，我们分析一下使用该构造函数的好处：
> 首先，**性能好**，这个很简单，一个是直接给数组赋值（相当于将String的value的指针指向char[] 数组），一个是逐一拷贝。当然是直接赋值快了。其次，共享内部数组**节约内存**

但是，该方法之所以设置为protected，是因为一旦该方法设置为共有，在外面可以访问的话，那就破坏了不可变性。
> 所以，从安全性角度考虑，他也是**安全**的。对于调用他的方法来说，由于无论是原字符串还是新字符串，其value数组本身就是String对象的私有属性，从外部是无法访问的，因此对两个字符串来说是很安全的。

## 其他方法
```java
length() 返回字符串长度

isEmpty() 返回字符串是否为空

charAt(int index) 返回字符串中第（index+1）个字符

char[] toCharArray() 转化成字符数组

trim() 去掉两端空格

toUpperCase() 转化为大写

toLowerCase() 转化为小写

String concat(String str) //拼接字符串

String replace(char oldChar, char newChar) //将字符串中的oldChar字符换成newChar字符

//以上两个方法都使用了String(char[] value, boolean share)；

boolean matches(String regex) //判断字符串是否匹配给定的regex正则表达式

boolean contains(CharSequence s) //判断字符串是否包含字符序列s

String[] split(String regex, int limit) 按照字符regex将字符串分成limit份。

String[] split(String regex)
```
### getBytes
在创建String的时候，可以使用byte[]数组，将一个字节数组转换成字符串，同样，我们可以将一个字符串转换成字节数组，那么String提供了很多重载的getBytes方法。但是，值得注意的是，在使用这些方法的时候一定要注意编码问题。比如：
```java
String s = "你好，世界！"; 
byte[] bytes = s.getBytes();
```
这段代码在不同的平台上运行得到结果是不一样的。**由于我们没有指定编码方式，所以在该方法对字符串进行编码的时候就会使用系统的默认编码方式，比如在中文操作系统中可能会使用GBK或者GB2312进行编码，在英文操作系统中有可能使用iso-8859-1进行编码**。这样写出来的代码就和机器环境有很强的关联性了，所以，为了避免不必要的麻烦，我们要**指定编码方式**。如使用以下方式：
```java
String s = "你好，世界！"; 
byte[] bytes = s.getBytes("utf-8");
```
### 比较方法
```java
boolean equals(Object anObject)；
boolean contentEquals(StringBuffer sb)；
boolean contentEquals(CharSequence cs)；
boolean equalsIgnoreCase(String anotherString)；
int compareTo(String anotherString)；
int compareToIgnoreCase(String str)；
boolean regionMatches(int toffset, String other, int ooffset,int len)  //局部匹配
boolean regionMatches(boolean ignoreCase, int toffset,String other, int ooffset, int len)   //局部匹配
```
字符串有一系列方法用于比较两个字符串的关系。 前四个返回boolean的方法很容易理解，前三个比较就是比较String和要比较的目标对象的字符数组的内容，一样就返回true,不一样就返回false，核心代码如下：
```java
 int n = value.length;
 while (n-- != 0) {
     if (v1[i] != v2[i])
         return false;
     i++;
 }
```
v1 v2分别代表String的字符数组和目标对象的字符数组。 第四个和前三个唯一的区别就是他会将两个字符数组的内容都使用toUpperCase方法转换成大写再进行比较，以此来忽略大小写进行比较。相同则返回true，不想同则返回false
**在这里，看到这几个比较的方法代码，有很多编程的技巧我们应该学习**。我们看equals方法：
```java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                     return false;
                 i++;
             }
             return true;
        }
    }
    return false;
}
```
该方法首先判断`this == anObject ?`，也就是说判断要比较的对象和当前对象是不是同一个对象，如果是直接返回true，如不是再继续比较，然后在判断`anObject`是不是`String`类型的，如果不是，直接返回false,如果是再继续比较，到了能终于比较字符数组的时候，他还是先比较了两个数组的长度，不一样直接返回false，一样再逐一比较值。 虽然代码写的内容比较多，但是可以很大程度上提高比较的效率。值得学习～～！！！

contentEquals有两个重载，**StringBuffer需要考虑线程安全问题**，再加锁之后调用`contentEquals((CharSequence) sb)`方法。`contentEquals((CharSequence) sb)`则分两种情况，一种是`cs instanceof AbstractStringBuilder`，另外一种是参数是String类型。具体比较方式几乎和equals方法类似，**先做“宏观”比较，在做“微观”比较**。
下面这个是equalsIgnoreCase代码的实现：
```java
public boolean equalsIgnoreCase(String anotherString) {
    return (this == anotherString) ? true
            : (anotherString != null)
            && (anotherString.value.length == value.length)
            && regionMatches(true, 0, anotherString, 0, value.length);
}
```
看到这段代码，眼前为之一亮。使用一个三目运算符和&&操作代替了多个if语句。
### hashCode
hashCode的实现其实就是使用数学公式：
> s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]

s[i]是string的第i个字符，n是String的长度。那为什么这里用31，而不是其它数呢? 计算机的乘法涉及到移位计算。当一个数乘以2时，就直接拿该数左移一位即可！选择31原因是因为31是一个素数！
所谓素数：
质数又称素数。指在一个大于1的自然数中，除了1和此整数自身外，没法被其他自然数整除的数。
在存储数据计算hash地址的时候，我们希望尽量减少有同样的hash地址，所谓“冲突”。如果使用相同hash地址的数据过多，那么这些数据所组成的hash链就更长，从而降低了查询效率！所以在选择系数的时候要选择尽量长的系数并且让乘法尽量不要溢出的系数，因为如果计算出来的hash地址越大，所谓的“冲突”就越少，查找起来效率也会提高。
31可以 由i*31== (i<<5)-1来表示，现在很多虚拟机里面都有做相关优化，使用31的原因可能是为了更好的分配hash地址，并且31只占用5bits！
在java乘法中如果数字相乘过大会导致溢出的问题，从而导致数据的丢失.
而31则是素数（质数）而且不是很长的数字，最终它被选择为相乘的系数的原因不过与此！
> 在Java中，整型数是32位的，也就是说最多有2^32= 4294967296个整数，将任意一个字符串，经过hashCode计算之后，得到的整数应该在这4294967296数之中。那么，最多有 4294967297个不同的字符串作hashCode之后，肯定有两个结果是一样的, hashCode可以保证相同的字符串的hash值肯定相同，但是，hash值相同并不一定是value值就相同。

### substring
```java
public String substring(int beginIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    int subLen = value.length - beginIndex;
    if (subLen < 0) {
        throw new StringIndexOutOfBoundsException(subLen);
    }
    return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
}
```
前面我们介绍过，java 8 中的substring方法使用String(value, beginIndex, subLen)方法创建一个新的String并返回，这个方法会将原来的char[]中的值逐一复制到新的String中，两个数组并不是共享的，虽然这样做损失一些性能，但是有效地避免了内存泄露。
### replaceFirst、replaceAll、replace区别
```java
String replaceFirst(String regex, String replacement)
String replaceAll(String regex, String replacement)
String replace(CharSequence target, CharSequence replacement)
```
> 1)replace的参数是char和CharSequence,即可以支持字符的替换,也支持字符串的替换 2)replaceAll和replaceFirst的参数是regex,即基于规则表达式的替换,比如,可以通过replaceAll(“\d”, “*”)把一个字符串所有的数字字符都换成星号; **相同点是都是全部替换**,即把源字符串中的某一字符或字符串全部换成指定的字符或字符串, **如果只想替换第一次出现的,可以使用 replaceFirst()**,这个方法也是基于规则表达式的替换,但与replaceAll()不同的是,只替换第一次出现的字符串;**另外,如果replaceAll()和replaceFirst()所用的参数据不是基于规则表达式的,则与replace()替换字符串的效果是一样的,即这两者也支持字符串的操作;**

### copyValueOf 和 valueOf
String的底层是由char[]实现的：通过一个char[]类型的value属性！早期的String构造器的实现呢，不会拷贝数组的，直接将参数的char[]数组作为String的value属性。然后`test[0] = 'A';`将导致字符串的变化。为了避免这个问题，提供了`copyValueOf`方法，每次都拷贝成新的字符数组来构造新的String对象。但是现在的String对象，在构造器中就通过拷贝新数组实现了，所以这两个方面在本质上已经没区别了。
> valueOf()有很多种形式的重载，包括：
```java
public static String valueOf(boolean b) {
      return b ? "true" : "false";
  }

  public static String valueOf(char c) {
       char data[] = {c};
       return new String(data, true);
  }
  public static String valueOf(int i) {
      return Integer.toString(i);
  }

  public static String valueOf(long l) {
     return Long.toString(l);
  }

 public static String valueOf(float f) {
     return Float.toString(f);
 }

 public static String valueOf(double d) {
    return Double.toString(d);
}
```
可以看到这些方法可以将六种基本数据类型的变量转换成String类型。
### intern()方法
```java
public native String intern();
```
该方法返回一个字符串对象的内部化引用。 众所周知：String类维护一个初始为空的字符串的对象池，当intern方法被调用时，如果对象池中已经包含这一个相等的字符串对象则返回对象池中的实例，否则添加字符串到对象池并返回该字符串的引用。
### String对“+”的重载
我们知道，Java是不支持重载运算符，String的“+”是java中唯一的一个重载运算符，那么java使如何实现这个加号的呢？我们先看一段代码：
```java
public static void main(String[] args) {
    String string="hollis";
    String string2 = string + "chuang";
}
```
然后我们将这段代码**反编译**：
```java
public static void main(String args[]){
   String string = "hollis";
   String string2 = (new StringBuilder(String.valueOf(string))).append("chuang").toString();
}
```
看了反编译之后的代码我们发现，其实String对“+”的支持其实就是使用了StringBuilder以及他的append、toString两个方法。
### String.valueOf和Integer.toString的区别
接下来我们看以下这段代码，我们有三种方式将一个int类型的变量变成呢过String类型，那么他们有什么区别？
```java
1.int i = 5;
2.String i1 = "" + i;
3.String i2 = String.valueOf(i);
4.String i3 = Integer.toString(i);
```
> 1、第三行和第四行没有任何区别，因为String.valueOf(i)也是调用Integer.toString(i)来实现的。 2、第二行代码其实是String i1 = (new StringBuilder()).append(i).toString();，首先创建一个StringBuilder对象，然后再调用append方法，再调用toString方法。

## 参考链接
- https://docs.oracle.com/javase/8/docs/api/java/lang/String.html
- https://www.hollischuang.com/archives/99

# StringBuffer & StringBuilder
## 简介
`StringBuffer`与`StringBuilder`是两个常用的操作字符串的类。大家都知道，`StringBuilder`是线程不安全的，而`StringBuffer`是线程安全的。前者是JDK1.5加入的，后者在JDK1.0就有了。

## 继承关系
```java
public final class StringBuffer
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence

public final class StringBuilder
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence    
```
可以看到，两个类的继承关系是一模一样的。`Serializable`是可以序列化的标志。`CharSequence`接口包含了`charAt()`、`length()`、`subSequence()`、`toString()`这几个方法，`String`类也实现了这个接口。这里的重点是抽象类`AbstractStringBuilder`，这个类封装了`StringBuilder`和`StringBuffer`大部分操作的实现。
## AbstractStringBuilder
### 变量及构造方法
```java
char[] value;
int count;
AbstractStringBuilder() {
}
AbstractStringBuilder(int capacity) {
    value = new char[capacity];
}
```
`AbstractStringBuilder`内部用一个`char[]`数组保存字符串，可以在构造的时候指定初始容量方法。
### 扩容
```java
public void ensureCapacity(int minimumCapacity) {
    if (minimumCapacity > 0)
        ensureCapacityInternal(minimumCapacity);
}
 private void ensureCapacityInternal(int minimumCapacity) {
    // overflow-conscious code
    if (minimumCapacity - value.length > 0)
        expandCapacity(minimumCapacity);
}
void expandCapacity(int minimumCapacity) {
    int newCapacity = value.length * 2 + 2;
    if (newCapacity - minimumCapacity < 0)
        newCapacity = minimumCapacity;
    if (newCapacity < 0) {
        if (minimumCapacity < 0) // overflow
            throw new OutOfMemoryError();
        newCapacity = Integer.MAX_VALUE;
    }
    value = Arrays.copyOf(value, newCapacity);
}
```
扩容的方法最终是由`expandCapacity()`实现的，在这个方法中首先把容量扩大为**原来的容量加2**，如果此时仍小于指定的容量，那么就把新的容量设为`minimumCapacity`。然后判断是否溢出，如果溢出了，把容量设为`Integer.MAX_VALUE`。最后把`value`值进行拷贝，**这显然是耗时操作**。
### append()方法
```java
public AbstractStringBuilder append(String str) {
        if (str == null)
            return appendNull();
        int len = str.length();
        ensureCapacityInternal(count + len);
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }
```
`append()`是最常用的方法，它有很多形式的重载。上面是其中一种，用于追加字符串。如果`str`是`null`,则会调用`appendNull()`方法。这个方法其实是追加了`'n'`、`'u'`、`'l'`、`'l'`这几个字符。如果不是`null`，则首先扩容，然后调用`String`的`getChars()`方法将`str`追加到`value`末尾。最后返回对象本身，所以`append()`可以连续调用。
## StringBuilder
`AbstractStringBuilder`已经实现了大部分需要的方法，`StringBuilder`和`StringBuffer`只需要调用即可。下面来看看`StringBuilder`的实现。
### 构造器
```java
public StringBuilder() {
    super(16);
}
public StringBuilder(int capacity) {
    super(capacity);
}
public StringBuilder(String str) {
    super(str.length() + 16);
    append(str);
}
public StringBuilder(CharSequence seq) {
    this(seq.length() + 16);
    append(seq);
}
```
可以看出，`StringBuilder`**默认的容量大小为16**。当然也可以指定初始容量，或者以一个已有的字符序列给`StringBuilder`对象赋初始值。
### append()方法
```java
public StringBuilder append(String str) {
    super.append(str);
    return this;
}
public StringBuilder append(CharSequence s) {
    super.append(s);
    return this;
}
```
`append()`的重载方法很多，这里随便列举了两个。显然，这里是直接调用的父类`AbstractStringBuilder`中的方法。
### toString()
```java
public String toString() {
    // Create a copy, don't share the array
    return new String(value, 0, count);
}
```
`toString()`方法返回了一个新的`String`对象，与原来的对象不共享内存。其实`AbstractStringBuilder`中的`subString()`方法也是如此。
## StringBuffer
`StiringBuffer`跟`StringBuilder`类似，只不过为了实现同步，很多方法使用`Synchronized`修饰，如下面的方法：
```java
public synchronized int length() {
        return count;
}
public synchronized StringBuffer append(String str) {
    toStringCache = null;
    super.append(str);
    return this;
}
public synchronized void setLength(int newLength) {
    toStringCache = null;
    super.setLength(newLength);
}
```
可以看到，方法前面确实加了`Synchronized`。
另外，在上面的`append()`以及`setLength()`方法里面还有个变量`toStringCache`。这个变量是用于最近一次`toString()`方法的缓存，任何时候只要`StringBuffer`被修改了这个变量会被赋值为`null`。`StringBuffer`的`toString`如下：
```java
public synchronized String toString() {
    if (toStringCache == null) {
        toStringCache = Arrays.copyOfRange(value, 0, count);
    }
    return new String(toStringCache, true);
}
```
在这个方法中，如果`toStringCache`为`null`则先缓存。最终返回的`String`对象有点不同，这个构造方法还有个参数`true`。找到`String`的源码看一下：
```java
String(char[] value, boolean share) {
    // assert share : "unshared not supported";
    this.value = value;
}
```
原来这个构造方法构造出来的`String`对象并没有实际复制字符串，只是把`value`指向了构造参数，这是为了节省复制元素的时间。不过这个构造器是具有包访问权限，一般情况下是不能调用的。
## 总结
- `StringBuilder`和`StringBuffer`都是可变字符串，前者线程不安全，后者线程安全。
- `StringBuilder`和`StringBuffer`的大部分方法均调用父类`AbstractStringBuilder`的实现。其扩容机制首先是把容量变为原来容量的2倍加2。最大容量是`Integer.MAX_VALUE`，也就是`0x7fffffff`。
- `StringBuilder`和`StringBuffer`的默认容量都是16，最好预先估计好字符串的大小避免扩容带来的时间消耗。
