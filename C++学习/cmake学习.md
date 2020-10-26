## 前言

本文算是对[官方文档](https://cmake.org/cmake/help/v3.18/guide/tutorial/index.html)的翻译内容吧，不算多么深的技术。也将其中的代码发布到[Github](https://github.com/TUGOhost/cmakeDemo)。跟官网一样，也是一步步的来。

## 开始（Step 1）

大部分基础项目都是从源代码文件编译成可执行文件。对于一个简单的项目来说，`CMakeLists.txt`只需要三行就可以解决。

创建一个`CMakeLists.txt`文件，在`Step1` 的文件夹中：

```cmake
# 设置cmake最低的版本号
cmake_minimum_required(VERSION 3.10)

# 设置项目名
project(Tutorial)

# 添加可执行的文件
add_executable(Tutorial tutorial.cc)
```

**注意**在这个例子中使用了小写的命令。大写、小写或者大小写都用的命令在`CMake`中都是被支持的。

`tutorial.cc`的源代码是一个能够计算一个数的平方根。

### 添加版本号和头文件

第一个功能是我们给可执行文件和项目添加版本号。尽管我们可以在源代码中这样操作，但是使用`CMake`可以提供更大的灵活性。

首先，修改`CMakeLists.txt`文件以使用`project()`命令设置项目名称和版本号：

```cmake
# 设置项目名和版本号
project(Tutorial VERSION 1.0) 
```

之后，配置头文件将版本号传给源代码：

```cmake
configure_file(tutorial.h.in tutorial.h)
```

由于已配置的文件将被写入二叉树，因此我们必须将该目录添加到路径列表中以搜索包含文件。将代码添加到`CMakeLists.txt`文件的末尾：

```cmake
target_include_directories(Tutorial PUBLIC
             "${PROJECT_BINARY_DIR}"
             )
```

在源目录中创建`tutorial.h.in`：

```c
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

当`CMake`配置头文件的时候，`@Tutorial_VERSION_MAJOR@`和`@Tutorial_VERSION_MINOR@`都会被替换掉。

下一步，修改`tutorial.cc`，包含头文件`tutorial.h`。

最后，通过更新`tutorial.cc`来打印版本号：

```cc
  if (argc < 2) {
    // 打印版本号
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
```

### 指定C++标准

```cmake
# 指定C++标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

### 编译并且测试

通过运行`cmake`程序或者`cmake-gui`来配置项目，然后使用配所选的构建工具进行构建。

```
mkdir Step1_build && cd Step1_build
cmake ../Step1
cmake --build
```

```shell
./Tutorial 25
the sqrt of 25 is 5
```

## 添加库文件（Step 2）

现在我们向我们的项目中添加一个库文件。这个库文件将会包含我们自己实现的求一个数的平方根。然后可执行文件使用此库而不是编译器提供的标准·平方根函数。

作为教程，我们将这个库文件放在名为`MathFunctions`的子目录中。这个子目录包含着``mysqrt.h`的头文件和`mysqrt.cc`的源代码文件。这个源代码文件中包含一个叫`mysqrt`的函数提供了与编译器的`sqrt`函数类似的功能。

将下面这行添加到`MathFuntions`目录下的`CMakeLists.txt`中：

```cmake
add_library(MathFunctions mysqrt.cc)
```

为了能够调用库文件，我们将`Step2`文件夹中的`CMakeLists.txt`文件中添加一个`add_subdirectory()`以便构建该库。我们将新库添加到可执行文件并将`MathFunctions`作为包含文件以便可以找到`mysqrt.h`头文件。

```cmake
# 设置cmake最低的版本号
cmake_minimum_required(VERSION 3.10)

# 设置项目名和版本号
project(Tutorial VERSION 1.0) 

# 指定C++标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

configure_file(tutorial.h.in tutorial.h)

# 添加可执行的文件
add_executable(Tutorial tutorial.cc)


# 添加MathFunctions库
add_subdirectory(MathFunctions)

# 添加可执行程序
add_executable(Tutorial tutorial.cc)

target_link_libraries(Tutorial PUBLIC MathFunctions)

# 将二叉树添加到包含文件的搜索路径以便让我们能够找到tutorial.h
target_include_directories(Tutorial PUBLIC
             "${PROJECT_BINARY_DIR}"
             "${PROJECT_SOURCE_DIR}/MathFunctions"
             )
```

 对于大型的项目来说，`CMakeLists.txt`可以设置`MathFunctions`库设为可选。

```
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# 配置头文件以传递某些CMake设置到源代码
configure_file(tutorial.h.in tutorial.h)
```

此设置存储在缓存中，因此用户无需在每次构建目录上运行CMake时都设置该值。

下一个更改是使编译和链接`MathFunctions`库成为条件：

```cmake
if(USE_MYMATH)
    add_subdirectory(MathFunctions)
    list(APPEND EXTRA_LIBS MathFunctions)
    list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
endif()


# 添加可执行程序
add_executable(Tutorial tutorial.cc)

target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# 将二叉树添加到包含文件的搜索路径以便让我们能够找到tutorial.h
target_include_directories(Tutorial PUBLIC
             "${PROJECT_BINARY_DIR}"
             ${EXTRA_INCLUDES}
             )
```

注意使用变量`EXTRA_LIBS`来收集所有可选库，以供以后链接到可执行文件中。

变量`EXTRA_INCLUDES`用于可选的头文件。当处理许多可选组件时，这是一种经典方法。

对源代码的相应更改非常简单。

首先，如果需要，在`tutorial.cc`中包含`MathFunctions.h`头文件

```c++
#ifdef USE_MYMATH
#include "MathFunctions.h"
#endif
```

然后相同的文件中使`USE_MYMATH`控制使用那个平方根函数

```c++
#ifdef USE_MYMATH
  const double outputValue = mysqrt(inputValue);
#else
  const double outputValue = sqrt(inputValue);
#endif
```

之后在`tutorial.h.in`中添加：

```c
#cmakedefine USE_MYMATH
```

## 给库文件添加使用规则（Step 3）

使用规则可以使库或者可执行文件的链接以及include得到更好的控制，同时还可以控制CMake内部目标的传递属性。使用规则的主要命令是：

- target_compile_definitions()
- target_compile_options()
- target_include_directories()
- target_link_libraries()

让我们添加库（Step 2）中的重构代码，以使用CMake的使用规则方法。我们首先申明，链接到`MathFunctions`的任何人都需要包括当前源目录，而`MathFunctions`本身不需要。因此，这可以成为接口使用规则。

记住`INTERFACE`表示消费者需要而生产者不需要的东西。将下面代码添加到`/MathFunctions/CMakeLists.txt`的末尾：

```cmake
target_include_directories(MathFunctions
    INTERFACE${CMAKE_CURRENT_SOURCE_DIR}
    )
```

现在，我们已经指定了`MathFunctions`的使用规则，我们可以安全的从顶级`CMakeLists.txt`中删除对`EXTRA_INCLUDES`变量的使用，这里:

```cmake
if(USE_MYMATH)
    add_subdirectory(MathFunctions)
    list(APPEND EXTRA_LIBS MathFunctions)    
endif()
```

还有这里：

```cmake
target_include_directories(Tutorial PUBLIC
    "${PROJECT_BINARY_DIR}"
    )
```

完成此操作后，运行cmake可执行文件或者cmake-gui来配置项目，然后使用您选择的构建工具或者使用cmake --build进行构建。

## 安装并测试（Step 4）

现在我们可以往我们的项目中添加安装谷子额和测试支持了。

### 安装规则

安装规则相当简单：对于`MathFunctions`我们要安装库和头文件，对于应用程序，我们要安装可执行文件和配置的头文件。

所以在`MathFunctions/CMakeLists.txt`末尾添加：

```cmake
install(TARGETS MathFunctions DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```

并在顶级目录下的`CMakeLists.txt`中添加：

```cmake
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )
```



