# 前言

需要将特定的smali代码插入到dex文件中起到特殊作用，但是对于smali语法一知半解，这次来总结一下，并介绍如何使用工具来讲smali代码插入dex文件中。

# Smali语法

可以使用IDEA或者Android Studio的`Java2Smali`插件来查看smali代码。

源码：

```java
package cn.soulapp;

import android.os.Bundle;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;

import com.example.souldemo.R;

public class MainAcitivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}
```

smali代码：

其中`#`后面为注释

```smali
# 类名 .class
.class public Lcn/soulapp/MainAcitivity;
# 该类的父类
.super Landroidx/appcompat/app/AppCompatActivity;
# 文件名
.source "MainAcitivity.java"


# direct methods
.method public constructor <init>()V
	# 表示需要寄存器数量，这里表示需要1个寄存器
    .registers 1

	# 表示逻辑代码的开始处
    .prologue
    # 表示Java源文件名的行数
    .line 10
    # invoke-direct 用于调用非 static 直接方法（也就是说，本质上不可覆盖的实例方法，即 private 实例方法或构造函数）。
    invoke-direct {p0}, Landroidx/appcompat/app/AppCompatActivity;-><init>()V
	# 表示该方法无返回值
    return-void
# 表示方法执行结束
.end method
# 后面的可以参考给出的Dalvik 字节码链接中的信息，很多都是从这里获得的。

# virtual methods
.method protected onCreate(Landroid/os/Bundle;)V
    .registers 3
    .param p1, "savedInstanceState"    # Landroid/os/Bundle;
        .annotation build Landroidx/annotation/Nullable;
        .end annotation
    .end param

    .prologue
    .line 13
    invoke-super {p0, p1}, Landroidx/appcompat/app/AppCompatActivity;->onCreate(Landroid/os/Bundle;)V

    .line 14
    const v0, 0x7f0a001c

    invoke-virtual {p0, v0}, Lcn/soulapp/MainAcitivity;->setContentView(I)V

    .line 15
    return-void
.end method
```

## Java类型对应的类型描述符

| Java 类型 | 类型描述符 |
| --------- | ---------- |
| char      | C          |
| byte      | B          |
| short     | S          |
| int       | I          |
| long      | J          |
| float     | F          |
| double    | D          |
| boolean   | Z          |
| void      | V          |
| 对象      | L          |
| 数组      | [          |

基本类型的表示很简单，int 用 `I` 表示即可。对象的表示，顶级父类 Object 的表示方法 `Ljava/lang/Object;`，再比如 String 类型，就用 `Ljava/lang/String` 表示。

对于数组，DalviK 有特殊的表示方法 `[` 后面跟上数组元素的类型。`int[]` 的表示方式就是 `[I`, `String[]` 的表示方法是 `[Ljava/lang/String;`。二维数组用 `[[` 表示，`[[Ljava/lang/String` 就是指 `String[][]`，以此类推。

# org.jf.dexlib2工具

github地址：https://github.com/JesusFreke/smali/tree/master/dexlib2/

下载地址：https://bitbucket.org/JesusFreke/smali/downloads/

dexlib2工具可以修改dex文件内容，你可以插入想要的代码或者重写一个dex文件。当然一些反编译apk的工具也引用了该工具，比如apktool。

## 步骤

### 1.创建DexBackedDexFile

我们可以通过 **DexFileFactory** 类来创建一个 **DexBackedDexFile**实例

```kotlin
val dexBackedDexFile = DexFileFactory.loadDexFile(File("classes.dex"), Opcodes.getDefault())
```

一个 **DexBackedDexFile** 实例代表一个dex文件，通过 **DexBackedDexFile** 我们可以访问到所有Class

```kotlin
//所有classes
    val classes = dexBackedDexFile.classes
    println(classes.size)
```

### 2.ClassDef

**ClassDef** 代表一个类,我们可以通过 **DexBackedDexFile** 实例来获取一个类，也可以自行创建一个类.

```kotlin
//从DexBackedDexFile获取ClassDef
val zzz = dexBackedDexFile.classes.find {
        //type 代表class name
        it.type == "Lcom.xxx.yyy.zzz;"
    }
    //父类
    println(zzz?.superclass)
复制代码
val type = "Lcom.xxx.yyy.zzz;"
    val defClass = ImmutableClassDef(
        //类名
        type,
        AccessFlags.PUBLIC.value,
        //父类
        "Ljava/lang/Object;",
        //继承的接口
        null,
        //源文件
        null,
        //注解
        null,
        //静态成员变量
        listOf(
            ImmutableField(
                //所属类
                type,
                //成员名称
                "field1",
                //成员类型
                "I",
                AccessFlags.PRIVATE.value,
                //初始化值
                ImmutableIntEncodedValue(666),
                null,
                null
            )
        ),
        //成员变量
        null,
        //直系方法（自定义）
        null,
        //非直系（继承重载等）
        null
    )
```

### 3.修改某类的方法返回值

```kotlin
fun main() {

    val dexBackedDexFile = DexFileFactory.loadDexFile(File("classes.dex"), Opcodes.getDefault())

    val type = javaClassoType("com.uzmap.pkg.EntranceActivity")

    //定义一个DexRewriter
    val reWriter = DexRewriter(object : RewriterModule() {
        //修改方法
        override fun getMethodRewriter(rewriters: Rewriters): Rewriter<Method> = Rewriter {
            //判断类名和方法名
            if (it.definingClass == type && it.name == "isFromNativeSDK") {
                //返回修改后的方法
                return@Rewriter MethodWrapper(it).apply {
                    //修改方法的实现
                    methodImplementation = ImmutableMethodImplementation(
                        //寄存器个数
                        1, listOf(
                            ImmutableInstruction11n(
                                //指令
                                Opcode.CONST_4,
                                //寄存器
                                0,
                                //值
                                1),
                            //return p0
                            ImmutableInstruction11x(
                                Opcode.RETURN,
                                0
                            )
                        ), null, null
                    )
                }.build()
            }
            it
        }
    })

    val newDexFile = reWriter.dexFileRewriter.rewrite(dexBackedDexFile)
    DexFileFactory.writeDexFile("new.dex", newDexFile)
}

fun javaClassToType(clz: String): String = "L${clz.replace(".", "/")};"
```

上面我们修改了 **isFromNativeSDK** 方法使其返回true，这里有关于寄存器的知识:

**当一个方法被调用的时候，方法的参数被置于最后N个寄存器中。如果一个方法有2个参数，5个寄存器（v0-v4），那么参数将置于最后2个寄存器——v3和v4。 非静态方法中的第一个参数总是调用该方法的对象。**

这里指定了一个寄存器，就是第一个参数this——代表EntranceActivity实例。

修改后的smali代码:

```kotlin
# virtual methods
.method protected final isFromNativeSDK()Z
    .locals 0

    const/4 p0, 0x1

    return p0
.end method
```

​		寄存器都是32位的，能够支持任何类型。64位类型（Long和Double型）用2个寄存器表示。 有两种方式指定一个方法中有多少寄存器是可用的。.registers指令指定了方法中寄存器的总数。.locals指令表明了方法中非参寄存器的数量。

**MethodWrapper** 类的定义:

```kotlin
import org.jf.dexlib2.iface.Method
import org.jf.dexlib2.immutable.ImmutableMethod

class MethodWrapper(method: Method) {

    var definingClass = method.definingClass
    var name = method.name
    var parameters = method.parameters
    var returnType = method.returnType
    var accessFlags = method.accessFlags
    var annotations = method.annotations
    var hiddenApiRestrictions = method.hiddenApiRestrictions
    var methodImplementation = method.implementation

    fun build(): ImmutableMethod = ImmutableMethod(
        definingClass,
        name,
        parameters,
        returnType,
        accessFlags,
        annotations,
        hiddenApiRestrictions,
        methodImplementation
    )
}
```

# 总结

smali就相当于汇编一样，是比较底层的语言。研究它无非是对于安卓逆向或者事安卓加固有些许帮助，并可以通过修改dex文件来获得一些特殊功能。另外，研究它也可以增加自己反编译的知识，毕竟翻译这个还是很需要时间的，一些巨人已经将自己的成果发出来供大家使用，我们只是站在巨人的肩膀上。天下没有绝对的安全。

# 参考

- [Dalvik 字节码](https://source.android.google.cn/devices/tech/dalvik/dalvik-bytecode)
- [Dalvik 可执行文件格式](https://source.android.google.cn/devices/tech/dalvik/dex-format)
- [Dalvik 可执行指令格式](https://source.android.google.cn/devices/tech/dalvik/instruction-formats)
- [怎么使用org.jf.dexlib2去写或者重写dex文件](https://stackoverflow.com/questions/34916976/how-to-use-org-jf-dexlib2-write-or-rewrite-dex-file)